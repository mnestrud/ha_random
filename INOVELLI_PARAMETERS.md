# Ataraxia Lighting – Inovelli Blue Defaults (v1.13.1)

**Purpose:** Apply consistent Inovelli Blue defaults and reporting via Zigbee2MQTT.

---

## Inputs

| Name | Default | What it does |
|---|---|---|
| **MQTT Base Topic** | `zigbee2mqtt` | Root MQTT topic for Z2M (e.g. `zigbee2mqtt`). |
| **Configure Switch State Reporting** | `true` | Configures attribute reporting for on/off, level, and Inovelli default-levels. |
| **Configure Switch Options** | `true` | Sends all hard-coded device defaults via `bridge/request/device/configure`. |
| **Delay between MQTT publishes** | `0.5` seconds | Pause after each publish to avoid flooding the network/bridge. |
| **Target devices** | `[]` | Up to 20 devices to configure. |

---

### Zigbee attribute reporting (set if **Configure Switch State Reporting** = ON)

| Cluster.Attribute | EP | Min | Max | Change |
|---|---:|---:|---:|---:|
| `genOnOff.onOff` | 1 | 0 | 10800 | 0 |
| `genLevelCtrl.currentLevel` | 1 | 1 | 10800 | 1 |
| `manuSpecificInovelli.defaultLevelLocal` | 1 | 0 | 10800 | 1 |
| `manuSpecificInovelli.defaultLevelRemote` | 1 | 0 | 10800 | 1 |

### Zigbee2MQTT device options (published to `…/bridge/request/device/options`)
| Option | Value |
|---|---|
| `transition` | `0.4` |
| `state_action` | `false` |
| `optimistic` | `true` |
| `qos` | `0` |

### Light defaults (pushed via `/set`)
| Key | Value |
|---|---|
| `level_config.execute_if_off` | `true` |

---

## Hard-coded Device Defaults (sent via `…/bridge/request/device/configure`)

> These values are pushed to each device when **Configure Switch Options** is ON.

| Key | Value | Key | Value |
|---|---|---|---|
| `dimmingSpeedUpRemote` | 0 | `dimmingSpeedDownLocal` | 127 |
| `dimmingSpeedUpLocal` | 20 | `minimumLevel` | 1 |
| `rampRateOffToOnRemote` | 0 | `maximumLevel` | 254 |
| `rampRateOffToOnLocal` | 0 | `invertSwitch` | "No" |
| `dimmingSpeedDownRemote` | 127 | `autoTimerOff` | 0 |
| `stateAfterPowerRestored` | 255 | `loadLevelIndicatorTimeout` | "Stay On" |
| `smartBulbMode` | "Smart Bulb Mode" | `doubleTapUpToParam55` | "Disabled" |
| `doubleTapDownToParam56` | "Disabled" | `brightnessLevelForDoubleTapUp` | 254 |
| `brightnessLevelForDoubleTapDown` | 1 | `singleTapBehavior` | "Old Behavior" |
| `buttonDelay` | "300ms" | `ledColorWhenOn` | 170 |
| `ledColorWhenOff` | 170 | `ledIntensityWhenOn` | 33 |
| `ledIntensityWhenOff` | 1 | `defaultLevelLocal` | 251 |
| `defaultLevelRemote` | 251 | `localProtection` | "Disabled" |
| `bindingOffToOnSyncLevel` | "Disabled" | `firmwareUpdateInProgressIndicator` | "Enabled" |
| `fanControlMode` | "Disabled" | `lowLevelForFanControlMode` | 63 |
| `onOffLedMode` | "All" | `outputMode` | "Dimmer" |
| `activeEnergyReports` | 0 | `activePowerReports` | 0 |
| `auxSwitchUniqueScenes` | "Disabled" | `doubleTapClearNotifications` | "Disabled" |
| `fanLedLevelType` | 0 | `highLevelForFanControlMode` | 254 |
| `higherOutputInNonNeutral` | "Disabled (default)" | `ledBarScaling` | "Gen3 method (VZM-style)" |
| `ledColorForFanControlMode` | 255 | `mediumLevelForFanControlMode` | 128 |
| `periodicPowerAndEnergyReports` | 0 | `quickStartLevel` | 254 |
| `quickStartTime` | 0 | `relayClick` | "Enabled (Click Sound Off)" |
| `switchType` | "Single Pole" |  |  |

**Per-LED defaults (segments 1–7):**  
For each X in 1…7 →  
`defaultLedXColorWhenOff=255`, `defaultLedXColorWhenOn=255`, `defaultLedXIntensityWhenOff=101`, `defaultLedXIntensityWhenOn=101`.

---

## Adaptive lighting parameters

- ### `level_control`
  Manages brightness and ramping.  
  **Why it matters:** Adaptive lighting continuously adjusts brightness; accurate reporting ensures HA stays in sync.

- ### `minimumLevel`
  Default: **1**. Lowest allowed dim level.  
  **Why it matters:** Stops lights from flickering or failing to turn on at very low levels.

- ### `maximumLevel`
  Default: **254**. Highest allowed dim level.  
  **Why it matters:** Prevents adaptive lighting from setting levels brighter than desired.

- ### `smartBulbMode`
  Default: **Smart Bulb Mode**. Keeps the load powered for smart bulbs.  
  **Why it matters:** Ensures adaptive lighting works without cutting power to bulbs.

- ### `buttonDelay`
  Default: **300ms**. Tap delay for scene/multi-tap detection.  
  **Why it matters:** Influences responsiveness of adaptive lighting on manual input.

- ### `singleTapBehavior`
  Default: **Old Behavior**. Defines how a single tap behaves.  
  **Why it matters:** Impacts how adaptive lighting feels when toggling.

- ### `bindingOffToOnSyncLevel`
  Default: **Disabled**. Syncs brightness from bound devices when turning on.  
  **Why it matters:** Adaptive lighting prefers disabled, so levels follow the adaptive tick.

- ### `defaultLevelLocal` *(Adaptive Tick Target)*
  Default: **251**. On-level when toggled locally.  
  **Why it matters:** Adaptive tick updates this every cycle so the switch powers on at the adaptive brightness.

- ### `defaultLevelRemote` *(Adaptive Tick Target)*
  Default: **251**. On-level when toggled remotely.  
  **Why it matters:** Adaptive tick updates this every cycle so remote commands use adaptive brightness.

- ### `level_config.execute_if_off`
  Default: **true**. Allows brightness updates while off.  
  **Why it matters:** Keeps adaptive brightness in sync so lights turn on at the right level.
