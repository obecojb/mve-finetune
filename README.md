# Marstek Battery Control — Home Assistant Blueprints

These blueprints are a companion and fine-tuning extension for the excellent
[Marstek Venus Energy Manager](https://github.com/ffunes/Marstek-Venus-Energy-Manager) by ffunes.

The Energy Manager handles all the core control logic for Marstek battery storage —
these blueprints add a permission layer on top:
They decide based on **battery temperature**, **current electricity price** and
**grid consumption vs. generation** whether the batteries are available to the
Energy Manager for charging or discharging at all.
The actual control remains entirely with the Energy Manager.

> **In short:** The Energy Manager decides *how* to charge —
> these blueprints decide *whether* charging is allowed.

---

Home Assistant blueprints for controlling battery storage based on Tibber electricity price and PV surplus.

Developed for Marstek Venus (MTV01/02/03), works with any battery storage
that controls charging/discharging via HA switches.

---

## Blueprints

### 1. Battery Charge Lock (Electricity Price + PV Surplus)

`akku_ladesperre_tibber_pv.yaml`

Automatically switches charging/discharging based on electricity price and PV surplus.

**Logic (priority order):**

| Condition | Discharge | Charge |
|---|---|---|
| Temperature alarm active | — (no action) | — (no action) |
| Grid draw < feed-in threshold (PV surplus) | OFF | ON |
| Electricity price < threshold (cheap) | OFF | ON |
| Electricity price ≥ threshold (expensive) | ON | OFF |

**Configurable parameters:**
- Electricity price sensor + threshold (default: 25 ct/kWh)
- Grid power sensor + feed-in/draw thresholds (default: ±100 W)
- Delay for grid power trigger (default: 2 minutes)
- Discharge switches (multiple supported)
- Charge switches (multiple supported)
- Temperature alarm boolean (safety interlock)

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/obecojb/mve-finetune/main/akku_ladesperre_tibber_pv.yaml)

---

### 2. Battery Temperature Alarm

`batterie_temperatur_alarm.yaml`

Disables charging and discharging when a temperature threshold is exceeded. Sends a push notification.
Also checks on HA start whether temperatures are already elevated.

**Companion blueprint:** use together with "Battery Temperature All-Clear".

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/obecojb/mve-finetune/main/batterie_temperatur_alarm.yaml)

---

### 3. Battery Temperature All-Clear

`batterie_temperatur_entwarnung.yaml`

Re-enables charging and discharging when temperature has dropped back below threshold after an alarm. Sends a push notification.

**Companion blueprint:** use together with "Battery Temperature Alarm".

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/obecojb/mve-finetune/main/batterie_temperatur_entwarnung.yaml)

---

## Recommended Setup

Install all three blueprints and configure them with the same entities:

```
Temperature Alarm Blueprint
  → sets input_boolean.battery_temperature_alarm = ON
  → turns all charge/discharge switches OFF

Temperature All-Clear Blueprint
  → waits for input_boolean.battery_temperature_alarm = ON
  → resets it to OFF
  → turns all charge/discharge switches ON

Charge Lock Blueprint
  → checks input_boolean.battery_temperature_alarm
  → aborts if ON (temperature alarm takes priority)
```

## Requirements

- Home Assistant ≥ 2023.4
- Electricity price sensor in **ct/kWh** (e.g. Tibber integration)
- Grid power sensor in **watts** (negative value = feed-in)
- `input_boolean` for temperature alarm status (create manually)
- Notify service for push notifications
