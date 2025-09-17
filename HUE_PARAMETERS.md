# Ataraxia Lighting – Hue Defaults (v1.15)

**Purpose:** Apply Hue light defaults and reporting via Zigbee2MQTT.  
**Reference device:** Not used in this version.  
**MQTT base topic:** Configurable (default `zigbee2mqtt`).

---

## Inputs

| Name | Default | What it does |
|---|---|---|
| **MQTT Base Topic** | `zigbee2mqtt` | Root MQTT topic for Z2M (e.g. `zigbee2mqtt`). |
| **state_action** | `false` | Also publish action events from Z2M. |
| **color_sync** | `true` | Keep `xy` and `color_temp` modes in sync. |
| **transition** | `0.4` | Default transition for level/color/on/off commands. |
| **optimistic** | `true` | Assume command success and update state immediately. |
| **Delay between MQTT publishes** | `0.5` seconds | Pause after each publish to avoid flooding. |
| **Configure Switch State Reporting** | `true` | Configure reporting on endpoint 11 for on/off, level, color temperature. |
| **Power-on behavior** | `previous` | Behavior after power restore: `previous`, `"on"`, or `"off"`. |
| **Target devices** | `[]` | One or more Hue bulbs to configure. |

---

## Hidden / Internal Defaults (actual parameter names)

### Zigbee attribute reporting (if **Configure Switch State Reporting** = ON, Endpoint 11)

| Cluster.Attribute | EP | Min | Max | Change |
|---|---:|---:|---:|---:|
| `genOnOff.onOff` | 11 | 0 | 10800 | 0 |
| `genLevelCtrl.currentLevel` | 11 | 0 | 10800 | 1 |
| `lightingColorCtrl.colorTemperature` | 11 | 0 | 10800 | 1 |

### Zigbee2MQTT device options (published to `…/bridge/request/device/options`)
| Option | Value (from inputs) |
|---|---|
| `transition` | user input (default `0.4`) |
| `color_sync` | user input (default `true`) |
| `state_action` | user input (default `false`) |
| `optimistic` | user input (default `true`) |
| `qos` | `0` |

### Light defaults (pushed via `/set`, one key per publish)
| Key | Value |
|---|---|
| `power_on_behavior` | user input (default `previous`) |
| `color_temp_startup` | `370` (mireds) |
| `color_options.execute_if_off` | `true` *(always pushed)* |
| `level_config.execute_if_off` | `true` *(always pushed)* |

---

## Hard-coded Device Defaults

Hue doesn’t expose many vendor-specific options. Hard-coded here are:  
- **Light defaults**: `color_temp_startup=370`, `color_options.execute_if_off=true`, `level_config.execute_if_off=true`  
- **Device options**: `qos=0` (others are user-configurable)

---

## Adaptive lighting parameters

- ### `level_control`
  Governs brightness and ramping via `currentLevel`.  
  **Why it matters:** Adaptive lighting constantly adjusts brightness; reporting ensures HA stays accurate.

- ### `power_on_behavior`
  Default: **previous**. Controls state after power restore.  
  **Why it matters:** Ensures lights restore to their last adaptive state after a power loss.

- ### `color_temp_startup`
  Default: **370 mireds**. Startup color temperature.  
  **Why it matters:** Ensures adaptive color temperature is predictable from power-on.

- ### `color_options.execute_if_off`
  Default: **true**. Allows color commands while off.  
  **Why it matters:** Adaptive color updates are stored while off, so they’re correct on next power-on.

- ### `level_config.execute_if_off`
  Default: **true**. Allows brightness updates while off.  
  **Why it matters:** Ensures adaptive brightness changes are retained while the light is off.

- ### Device options that affect adaptive lighting
  - **`transition`** (default **0.4s**): Smooth fade time for adaptive updates.  
  - **`color_sync`** (default **true**): Keeps color modes aligned for adaptive color temperature control.  
  - **`optimistic`** (default **true**): HA assumes success; adaptive logic runs smoothly without waiting.  
  - **`state_action`** (default **false**): Action events usually unnecessary; off by default to reduce MQTT noise.
