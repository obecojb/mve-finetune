# Marstek Battery Control — Home Assistant Blueprints

Diese Blueprints sind eine Ergänzung und Weiterentwicklung des hervorragenden
[Marstek Venus Energy Managers](https://github.com/ffunes/Marstek-Venus-Energy-Manager) von ffunes.

Der Energy Manager übernimmt die gesamte Steuerungslogik der Marstek-Speicher —
diese Blueprints erweitern ihn um eine vorgelagerte Freigabe-Ebene:
Sie entscheiden anhand von **Batterietemperatur**, **aktuellem Strompreis** und
**Verbrauch bzw. Einspeisung**, ob die Batterien dem Energy Manager überhaupt
zum Laden oder Entladen zur Verfügung stehen.
Die eigentliche Steuerung bleibt vollständig beim Energy Manager.

> **Kurzgefasst:** Der Energy Manager entscheidet *wie* geladen wird —
> diese Blueprints entscheiden *ob* geladen werden darf.

---

Home Assistant Blueprints für die Steuerung von Batteriespeichern mit Tibber-Strompreis und PV-Überschuss.

Entwickelt für Marstek Venus (MTV01/02/03), funktioniert mit jedem Batteriespeicher
der Laden/Entladen per HA-Switch steuert.

---

## Blueprints

### 1. Akku Lade-Sperre (Strompreis + PV-Überschuss)

`akku_ladesperre_tibber_pv.yaml`

Schaltet Laden/Entladen automatisch anhand von Strompreis und PV-Überschuss.

**Logik (Priorität):**

| Bedingung | Entladen | Laden |
|---|---|---|
| Temperatur-Alarm aktiv | — | — |
| Netzbezug < Einspeisung-Schwelle (PV-Überschuss) | OFF | ON |
| Strompreis < Schwelle (günstig) | OFF | ON |
| Strompreis ≥ Schwelle (teuer) | ON | OFF |

**Konfigurierbare Parameter:**
- Strompreis-Sensor + Schwelle (Standard: 25 ct/kWh)
- Netzbezug-Sensor + Einspeisung-/Bezugs-Schwelle (Standard: ±100 W)
- Verzögerung für Netzbezug-Trigger (Standard: 2 Minuten)
- Entlade-Schalter (mehrere möglich)
- Lade-Schalter (mehrere möglich)
- Temperatur-Alarm Boolean (Schutzabschaltung)

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/obecojb/mve-finetune/main/blueprints/automation/marstek/akku_ladesperre_tibber_pv.yaml)

---

### 2. Batterie Temperatur-Alarm

`batterie_temperatur_alarm.yaml`

Deaktiviert Laden/Entladen bei Übertemperatur. Sendet Push-Benachrichtigung.
Prüft auch beim HA-Start ob Temperaturen bereits erhöht sind.

**Companion-Blueprint:** zusammen mit "Batterie Temperatur-Entwarnung" verwenden.

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/obecojb/mve-finetune/main/blueprints/automation/marstek/batterie_temperatur_alarm.yaml)

---

### 3. Batterie Temperatur-Entwarnung

`batterie_temperatur_entwarnung.yaml`

Reaktiviert Laden/Entladen wenn Temperatur nach einem Alarm wieder gesunken ist.
Sendet Push-Benachrichtigung.

**Companion-Blueprint:** zusammen mit "Batterie Temperatur-Alarm" verwenden.

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/obecojb/mve-finetune/main/blueprints/automation/marstek/batterie_temperatur_entwarnung.yaml)

---

## Empfohlene Kombination

Alle drei Blueprints zusammen installieren und mit denselben Entities konfigurieren:

```
Temperatur-Alarm Blueprint
  → setzt input_boolean.batterie_temperatur_alarm = ON
  → schaltet alle Charge/Discharge-Switches OFF

Temperatur-Entwarnung Blueprint
  → wartet auf input_boolean.batterie_temperatur_alarm = ON
  → setzt ihn zurück auf OFF
  → schaltet alle Charge/Discharge-Switches ON

Lade-Sperre Blueprint
  → prüft input_boolean.batterie_temperatur_alarm
  → bricht ab wenn ON (Temperatur-Alarm hat Vorrang)
```

## Voraussetzungen

- Home Assistant ≥ 2023.4
- Strompreis-Sensor in **ct/kWh** (z.B. Tibber-Integration)
- Netzbezug-Sensor in **Watt** (negativer Wert = Einspeisung)
- `input_boolean` für Temperatur-Alarm-Status (manuell anlegen)
- Notify-Service für Push-Benachrichtigungen
