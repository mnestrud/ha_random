# Ataraxia Lighting Blueprints

A curated set of Home Assistant blueprints keeps Ataraxia's Zigbee lighting responsive, coordinated, and easy to maintain. The collection now follows a four-part structure: a master automation that drives adaptive updates, a per-area script that talks to MQTT, device-default scripts for Hue bulbs and Inovelli switches, and a flexible multi-tap automation. Two supplemental Markdown files document every parameter pushed to each platform.

## Repository map

| File | Type | What it does |
| --- | --- | --- |
| [`Ataraxia Lighting - Tick Blueprint.yaml`](./Ataraxia%20Lighting%20-%20Tick%20Blueprint.yaml) | Automation blueprint | Timer-driven "tick" automation that gathers Adaptive Lighting targets, evaluates area availability, and calls per-room scripts with staggered delays to avoid flooding Zigbee. |
| [`Ataraxia Lighting - Adaptive Push Script.yaml`](./Ataraxia%20Lighting%20-%20Adaptive%20Push%20Script.yaml) | Script blueprint | Per-area MQTT publisher that normalizes brightness/color, skips redundant publishes when HA state already matches, and sequences switch/group updates with optional delays. |
| [`Ataraxia Lighting - Switch Taps Automation.yaml`](./Ataraxia%20Lighting%20-%20Switch%20Taps%20Automation.yaml) | Automation blueprint | Full-featured Inovelli Blue multi-tap handler that wires single taps to Adaptive Lighting helpers, exposes optional actions per gesture, and can trigger LED effects using HA 2024.8+ action syntax. |
| [`Ataraxia Lighting - Inovelli Blue Defaults + Reporting Script Blueprint`](./Ataraxia%20Lighting%20-%20Inovelli%20Blue%20Defaults%20+%20Reporting%20Script%20Blueprint) | Script blueprint | Zigbee2MQTT-focused switch configurator that pushes bundled Inovelli parameters, configures attribute reporting, publishes device options, and enforces `execute_if_off` safeguards with controlled pacing. |
| [`Ataraxia Lighting - Hue Defaults.yaml`](./Ataraxia%20Lighting%20-%20Hue%20Defaults.yaml) | Script blueprint | Hue bulb configurator that publishes reporting, device options, and per-key light defaults (including execute-if-off flags) to Zigbee2MQTT with a throttle between MQTT calls. |
| [`INOVELLI_PARAMETERS.md`](./INOVELLI_PARAMETERS.md) | Documentation | Tabulates every hard-coded switch parameter, device option, and reporting setting applied by the Inovelli defaults script to aid auditing and customization. |
| [`HUE_PARAMETERS.md`](./HUE_PARAMETERS.md) | Documentation | Mirrors the Hue defaults blueprint inputs, reporting definitions, and always-on payloads so you can track what will be sent before executing the script. |

## How the pieces fit together

### Adaptive cadence and per-room fan-out
The master tick automation reads the global Adaptive Lighting switch, optional overrides, and a Zigbee2MQTT availability sensor before iterating through up to fifteen rooms. Each enabled room invokes the Adaptive Push script with the resolved target brightness, color temperature, and power state while staggering calls to keep MQTT traffic smooth. The script normalizes values, compares them to current Home Assistant entities when provided, and only publishes to switch or group topics when changes are required. Together they keep switches, light groups, and Adaptive Lighting synchronized without redundant chatter.

### Device defaults and reporting hygiene
Separate scripts maintain vendor-specific defaults. The Inovelli defaults blueprint can configure up to twenty switches in parallel, pushing reporting, device options, and a comprehensive parameter set through Zigbee2MQTT configure calls while spacing each publish. The Hue defaults blueprint performs an analogous role for Philips Hue lights, applying reporting on endpoint 11, device options, and startup behavior per light with a programmable delay. Running these scripts ensures hardware stays aligned with the expectations of the adaptive cadence.

### Manual control and scene layering
The multi-tap automation blueprint listens for tap events from a single Inovelli switch and wires them to Adaptive Lighting helpers, optional day/night scenes, per-gesture HA actions, and LED effects—all using the modern Home Assistant action syntax. Deploying it alongside the tick automation keeps manual interventions in step with the automated MQTT updates.

### Reference tables
[`INOVELLI_PARAMETERS.md`](./INOVELLI_PARAMETERS.md) and [`HUE_PARAMETERS.md`](./HUE_PARAMETERS.md) provide printable tables for every publish the defaults scripts will attempt, including hidden execute-if-off safeguards and reporting intervals, so you can verify settings or document overrides before running them. Keep these nearby when customizing payloads or double-checking devices after upgrades.

## Step-by-step: Deploying the complete Ataraxia Lighting stack

1. **Confirm prerequisites.** Ensure Home Assistant 2024.8 or later, the Adaptive Lighting integration (with a helper switch exposing `brightness_pct` and `color_temp_kelvin`), MQTT enabled, and Zigbee2MQTT online with consistent base topics for switches and bulbs.
2. **Import the blueprints.** In Home Assistant navigate to *Settings → Automations & Scenes → Blueprints → Import*, and import each YAML blueprint from this repository using the URLs in their `source_url` metadata.
3. **Create required helpers.** Set up the heartbeat timer entity, global Adaptive Lighting helper switch, per-area boolean overrides if desired, and any input booleans or scripts referenced by the multi-tap automation (e.g., day/night scenes).
4. **Instantiate per-room Adaptive Push scripts.** For every room or light group, create a script from [`Ataraxia Lighting - Adaptive Push Script.yaml`](./Ataraxia%20Lighting%20-%20Adaptive%20Push%20Script.yaml), pointing it at the relevant Adaptive Lighting switch, Zigbee2MQTT topics, and optional HA entities for "already correct" checks. Test one script manually to verify MQTT publishes target the correct devices.
5. **Deploy the master tick automation.** Create an automation from [`Ataraxia Lighting - Tick Blueprint.yaml`](./Ataraxia%20Lighting%20-%20Tick%20Blueprint.yaml) that references your heartbeat timer, Zigbee2MQTT bridge availability sensor, global Adaptive Lighting switch, and the scripts instantiated in step 4. Configure per-area delays and overrides to suit your environment. Trigger the timer manually to confirm that each area updates in sequence.
6. **Configure manual control.** For each Inovelli switch, instantiate [`Ataraxia Lighting - Switch Taps Automation.yaml`](./Ataraxia%20Lighting%20-%20Switch%20Taps%20Automation.yaml), select the device, map single-tap actions to the Adaptive Lighting helper or per-room script, and optionally assign additional actions or LED effects for other gestures. Validate that taps trigger the expected MQTT updates without conflicting with the tick automation.
7. **Align Inovelli defaults.** Run the [`Ataraxia Lighting - Inovelli Blue Defaults + Reporting Script Blueprint`](./Ataraxia%20Lighting%20-%20Inovelli%20Blue%20Defaults%20+%20Reporting%20Script%20Blueprint) against batches of switches to push the curated parameter set, reporting, and execute-if-off safeguards. Refer to [`INOVELLI_PARAMETERS.md`](./INOVELLI_PARAMETERS.md) if you plan to tweak or verify any values. Allow enough time between runs for Zigbee2MQTT to process the publishes.
8. **Align Hue defaults.** Execute [`Ataraxia Lighting - Hue Defaults.yaml`](./Ataraxia%20Lighting%20-%20Hue%20Defaults.yaml) for your Hue bulbs to apply consistent reporting, device options, and startup behavior, using [`HUE_PARAMETERS.md`](./HUE_PARAMETERS.md) as a checklist. Schedule periodic re-runs after firmware updates or major changes.
9. **Smoke test the stack.** Finish the timer to observe room-by-room updates, toggle switches to ensure manual control stays synchronized, and review MQTT logs for unexpected traffic. Confirm the defaults scripts no longer log invalid values such as `minimumLevel=0` thanks to the enforced parameter set.

With these steps complete you have an end-to-end Ataraxia Lighting deployment: adaptive brightness and color ripple across rooms, manual taps feed the same logic, and vendor defaults stay locked to values that keep MQTT traffic lean and lights predictable.
