# HA Heizungs-Automationen (HC1 / Vorlauf / Offset / Watchdog)

Sammlung von Home-Assistant Automationen zur robusten Heizungsregelung mit Fokus auf:
- **wetterbasiertem Raum-Offset**
- **Vorlauf-Sollwert-Schreiben mit 37°C Cap**
- **sanfter Vorlauf-Nachregelung über Raumfehler**
- **Watchdog gegen 0/ungültige Sollwerte**
- **Autotune für Gain**
- **Heizmodus-Guard (immer `heat`)**
- **Pumpe aus bei Heizpause**

Die Automationen sind im **neuen HA-Schema** geschrieben (`triggers / conditions / actions` + `trigger:`/`action:`), um Probleme im Trace/Editor zu vermeiden.

---

## Inhalt / Automationen

| ID | Zweck |
|---|---|
| `hc1_autotune_gain` | Passt Gain abhängig vom absoluten Raumfehler an |
| `hc1_offset_writer_weather` | Schreibt wetterbasierten Offset (Clamp, Limit, 0.5er Schritte, Hysterese) |
| `flow_setpoint_writer_37cap` | Schreibt Vorlauf-Soll mit Clamp (30–37°C), Rate-Limit, 0.5°C Schritte |
| `flow_setpoint_nudger_v1` | Sanfte Nachregelung des Vorlaufs über Raumfehler (mit Schutz vor 0/unknown) |
| `watchdog_flow_setpoint` | Setzt Vorlauf-Soll wieder auf sicheren Wert wenn ungültig |
| `store_last_valid_flow_setpoint` | Speichert letzten gültigen Vorlauf-Soll (Variante A: nur gültig 20–70) |
| `pump_off_when_no_heating` | Setzt Pumpenleistung auf 0% wenn Heizen aus |
| `hc1_heatmode_guard` | Erzwingt HVAC-Modus `heat` |

---

## Features

- **Hysterese** gegen Flattern
- **Änderungsbegrenzung (per-change-limit)** gegen Sprünge
- **Clamp** auf sichere Min/Max-Werte
- **0.5er Rundung** (Offset/Vorlauf) für stabile Setpoints
- **Schutz vor `unknown/unavailable/0`** (wichtig für Trace + stabile Regelung)
- **Logbook-Logging** für schnelle Diagnose

---

## Voraussetzungen

### Home Assistant
- YAML Automationen (z. B. `automations.yaml` oder Packages)
- Deine Entities müssen zu den `entity_id`s passen

### Benötigte Helper
- `input_number.hc1_offset_gain`
- `input_number.last_valid_flow_setpoint`  
  ⚠️ Der Wertebereich muss zu deiner Konfig passen (z. B. 20–70°C).  
  Die Automation `store_last_valid_flow_setpoint` speichert nur Werte innerhalb dieses Bereichs.

---

## Installation

### Option A: Als Package (empfohlen)
1. Datei nach `config/packages/heating_automations.yaml` kopieren
2. Packages aktivieren (falls noch nicht geschehen):

```yaml
homeassistant:
  packages: !include_dir_named packages


	3.	Home Assistant neu starten

Option B: Direkt in automations.yaml
	•	Automations-Blöcke einfügen
	•	HA Automationen neu laden oder HA neu starten

⸻

Konfiguration / Anpassung

Entities anpassen

Passe diese Entities an deine Installation an (Beispiele):
	•	Vorlauf Soll: number.boiler_selected_flow_temperature
	•	Empfehlung Sensor: sensor.vorlauf_soll_37c_cap
	•	Heizen aktiv: binary_sensor.boiler_heating_active
	•	Kühlung aktiv (Filter): binary_sensor.hc1_cooling_filtered
	•	Heizfreigabe: binary_sensor.hc1_heizfreigabe
	•	Raumtemperatur: sensor.thermostat_hc1_current_room_temperature
	•	Raum-Soll: number.thermostat_hc1_selected_room_temperature

Tuning Parameter (wichtig)
	•	min_safe / max_safe (Vorlauf-Grenzen)
	•	hysteresis (gegen Flattern)
	•	per_change_limit (max. Änderung pro Lauf)
	•	step_up / step_down (Nudger Schritte)

Tipp: In den ersten Tagen das Logbook beobachten und hysteresis/per_change_limit lieber konservativ wählen.

⸻

Troubleshooting

Trace/Editor Fehler („t.includes“, „Error in describing trigger…“)
	•	Stelle sicher, dass kein altes Schema (platform: / service:) mehr in den Automationen steckt.
	•	Nutze konsequent triggers: mit trigger: und actions: mit action:.

Vorlauf wird zu niedrig (z. B. 0–2°C)
	•	Die Writer/Nudger in diesem Repo blocken Writes unter min_safe bzw. setzen sp_cur mindestens auf sp_min.
	•	Wenn du trotzdem 0 siehst: prüfen, ob ein anderes Script/Automation das Setpoint-Entity schreibt.

Pumpe bleibt auf 0%

pump_off_when_no_heating setzt nur auf 0%.
Wenn du Restore willst, brauchst du eine zusätzliche Automation („Pumpe an bei Heizbeginn“).

⸻

Sicherheitshinweis

Diese Automationen schreiben aktiv Heizungs-Setpoints. Nutze sinnvolle Min/Max-Grenzen und teste Änderungen vorsichtig (besonders im Winterbetrieb).
