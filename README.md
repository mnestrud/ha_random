# Ataraxia Lighting Blueprints

This repository collects Home Assistant blueprints for managing Ataraxia's adaptive lighting setup.  They cover three major tasks:

* Driving per-room lighting updates on a reliable cadence.
* Keeping Inovelli Blue switch defaults and Zigbee reporting aligned.
* Maintaining Hue bulb startup defaults for a cohesive experience.
* Exposing a highly configurable multi-tap automation for each Inovelli Blue switch.

All of the files are standard Home Assistant blueprints that can be imported with the **"Import Blueprint"** workflow in the UI or by copying the YAML into your `blueprints/` (automation/script) folders.  The sections below describe what each file does and how they work together.

## Repository layout

| File | Domain | Purpose |
| ---- | ------ | ------- |
| `AL_Tick_Automation_Blueprint.yaml` | Automation | Heartbeat automation that reads Adaptive Lighting values and fans them out to up to 15 areas via per-room scripts. |
| `AL_MQTT_script_blueprint.yaml` | Script | Synchronizes brightness and color temperature between Adaptive Lighting, Inovelli switches, and Zigbee2MQTT light groups while avoiding redundant MQTT traffic. |
| `Ataraxia Lighting - Inovelli Blue Defaults + Reporting Script Blueprint` | Script | Pushes a shared set of parameter defaults to multiple Inovelli Blue devices via HA config entities and optionally reconfigures Zigbee reporting through MQTT. |
| `Ataraxia Lighting - Hue Defaults` | Script | Applies a consistent set of Zigbee2MQTT device options and startup defaults to Hue bulbs. |
| `Ataraxia Inovelli Switch Automations` | Automation | Full-featured multi-tap automation for a single Inovelli Blue switch, including Adaptive Lighting integration, optional actions, and LED effects. |

## How the blueprints interact

### Adaptive Lighting heartbeat and per-room updates

1. **`AL_Tick_Automation_Blueprint.yaml`** listens for a Home Assistant timer or system start event to trigger a "tick".  On each tick it reads the global Adaptive Lighting switch and any per-area override switches, resolving a target brightness and color temperature for up to 15 areas.【F:AL_Tick_Automation_Blueprint.yaml†L1-L215】
2. For each enabled area whose controlling light is available, the automation calls the corresponding **`AL_MQTT_script_blueprint.yaml`** script with the resolved state, brightness, and color temperature.  It staggers calls with a configurable delay to avoid flooding Zigbee.【F:AL_Tick_Automation_Blueprint.yaml†L215-L387】【F:AL_MQTT_script_blueprint.yaml†L8-L132】
3. The script blueprint publishes MQTT payloads to the Inovelli switch `/set` topic and/or the Zigbee2MQTT light/group topic.  It first performs "already correct" checks using optional HA entities to skip redundant publishes, then issues the necessary MQTT commands and enforces an ordering delay when both switch and group targets are used.【F:AL_MQTT_script_blueprint.yaml†L16-L147】【F:AL_MQTT_script_blueprint.yaml†L147-L222】

Together, these two blueprints keep Inovelli switches, light groups, and Adaptive Lighting in sync without unnecessary traffic.

### Maintaining device defaults

* **`Ataraxia Lighting - Inovelli Blue Defaults + Reporting Script Blueprint`** reads the current configuration entities from a reference Inovelli Blue (or falls back to a hard-coded profile).  It can then push those values to up to 20 target devices by calling the appropriate `number.set_value` and `select.select_option` actions, and optionally resets/reapplies Zigbee reporting for key clusters via MQTT.【F:Ataraxia Lighting - Inovelli Blue Defaults + Reporting Script Blueprint†L1-L209】  Running this script before or after adding switches ensures the switches share a consistent configuration and reporting cadence.
* **`Ataraxia Lighting - Hue Defaults`** performs a similar role for Hue bulbs: for each selected device it publishes bridge-level option updates and device `/set` payloads, pausing between publishes to avoid congestion.  This keeps state-action behavior, transition defaults, and power-on behavior aligned across the Hue fleet.【F:Ataraxia Lighting - Hue Defaults†L1-L115】

These scripts complement the tick automation by keeping the underlying hardware configured with the expected defaults.

### Switch interactions and user control

* **`Ataraxia Inovelli Switch Automations`** is a per-device automation blueprint that reacts to Inovelli Blue multi-tap events.  It maps single taps to Adaptive Lighting helper/script interactions, offers dedicated day/night scenes on the config buttons, allows optional scene/script/light actions for single/double/triple taps, and drives LED effects through a helper script when configured.【F:Ataraxia Inovelli Switch Automations†L1-L209】【F:Ataraxia Inovelli Switch Automations†L209-L720】
* When used alongside the tick automation and MQTT script, single up/down taps can toggle the Adaptive Lighting helper and call the same per-room script used by the heartbeat, ensuring manual interventions stay in sync with the automated cadence.【F:Ataraxia Inovelli Switch Automations†L480-L720】

## Getting started

1. Import the desired blueprints into Home Assistant (Settings → Automations & Scenes → Blueprints → Import) using the raw URLs listed in each file's `source_url` field where provided, or copy the YAML into your `blueprints/automation` or `blueprints/script` directories.
2. Create helpers required by the blueprints (Adaptive Lighting dummy switch, per-area input_booleans, etc.) if they do not already exist.
3. Instantiate:
   * One automation from `AL_Tick_Automation_Blueprint.yaml`, wiring your heartbeat timer, Zigbee2MQTT bridge sensor, global Adaptive Lighting switch, and per-area entities/scripts.
   * One script per area from `AL_MQTT_script_blueprint.yaml` to target the matching Inovelli switch/light group topics.
   * Optional scripts from the defaults blueprints to initialize or periodically reapply device configurations.
   * Automations from `Ataraxia Inovelli Switch Automations` for each Inovelli Blue device to expose the multi-tap behaviors you prefer.
4. Test by manually triggering the timer finish event or a switch tap to confirm that the MQTT publishes and light behavior align with expectations.

## Notes

* All blueprints target Home Assistant 2024.8 or later, leveraging the new `action:` syntax for service calls where applicable.【F:Ataraxia Inovelli Switch Automations†L9-L15】
* MQTT topic examples in the scripts assume a Zigbee2MQTT deployment; adjust to match your setup.
* The defaults scripts run actions in parallel across selected devices but include pauses to remain friendly to Zigbee networks.

These blueprints are designed to be modular: you can use the multi-tap automation on its own, or deploy the full stack for a highly coordinated Adaptive Lighting experience.
