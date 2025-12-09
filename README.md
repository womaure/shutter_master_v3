# Shutter Master V3 – Automatische Rollladensteuerung für Home Assistant

> **Entwickler**: Wolfgang Maurer  
> **Getestet mit**: Home Assistant OS, Core 2025.12.2, Supervisor 2025.12.3, Operating System 16.3  
> **Zweck**: Automatische Steuerung von Rollläden basierend auf Sonnenstand, Temperatur, Wind, Regen und Nachtkühlung – mit Hysterese und Positionsprofilen.

---

## 📌 Übersicht

Diese Automation steuert **vier Rollläden** (`cover.rollladen_1`, `cover.rollladen_3`, `cover.rollladen_4`, `cover.rollladen_5`) basierend auf:

- **Sonnenhöhe** (Azimut/Elevation) → 4 Bereiche: `<10°`, `10–35°`, `35–50°`, `>50°`
- **Solarstrahlung** → Schwellen: `250`, `500`, `750`, `900` W/m²
- **Innentemperatur** → Schwellen: `24°C`, `26°C`, `28°C`
- **Außentemperatur** → für Nachtkühlung (wenn draußen mindestens `4°C` kühler als innen)
- **Windgeschwindigkeit** → ab `40 km/h` → alle Rollläden hoch
- **Regenintensität** → wenn >0 → alle Rollläden hoch
- **Hysterese** → mindestens `3 Minuten` zwischen Ausführungen
- **Positionsprofile** → je Rollladen und Sonnenhöhe unterschiedliche Öffnungsgrade

---

## 🧩 Verwendete Sensoren & Entitäten

Diese Automation nutzt **ausschließlich die Sensoren und Entitäten aus der Home Assistant-Installation von Wolfgang Maurer**:

```yaml
# Rollläden
cover.rollladen_1
cover.rollladen_3
cover.rollladen_4
cover.rollladen_5

# Sensoren
sensor.remshalden_buoch_solar_irradiance         # Solarstrahlung (W/m²)
sensor.aqara_temp_hum_wohnzimmer_temperature     # Innentemperatur
sensor.remshalden_buoch_outside_temperature      # Außentemperatur
sensor.remshalden_buoch_wind_gust                # Windgeschwindigkeit (km/h)
sensor.remshalden_buoch_rain_intensity           # Regenintensität (mm/h)

# Steuerung
input_boolean.rollladenautomationen              # Globale Freigabe (on/off)
input_number.last_shutter_run                    # Letzte Ausführung (Unix-Timestamp)
```

> ⚠️ **Wichtig**: Der `input_number.last_shutter_run` muss manuell in `configuration.yaml` oder über die UI erstellt werden.

---

## ⚙️ Funktionsweise

### 1. **Trigger**
Die Automation wird ausgelöst, wenn sich einer der folgenden Sensoren ändert:
- Solarstrahlung
- Innentemperatur
- Außentemperatur
- Windgeschwindigkeit
- Regenintensität
- Sonnenhöhe (`sun.sun.elevation`)

### 2. **Bedingung**
Nur aktiv, wenn `input_boolean.rollladenautomationen` auf `on` steht.

### 3. **Aktionen**

#### a) **Hysterese (3 Minuten)**
- Verhindert ständiges Auf/Abfahren.
- Speichert den Zeitpunkt der letzten Ausführung in `input_number.last_shutter_run`.

#### b) **Wind-/Regenschutz**
- Wenn Wind >40 km/h oder Regen >0 → alle Rollläden auf `100%` (geschlossen).

#### c) **Nachtkühlung**
- Nur nachts (`sun_elev < 0`) und wenn draußen mindestens `4°C` kühler als innen → alle Rollläden auf `25%` öffnen.

#### d) **Sonnenschutz**
- Bestimmt den „Modus“ (`off`, `very_low`, `low`, `mid`, `high`) basierend auf Solarstrahlung und Innentemperatur.
- Bestimmt den „Sonnenhöhenbereich“ (`very_low`, `low`, `mid`, `high`) basierend auf `sun.sun.elevation`.
- Weist jedem Rollladen eine Position zu, basierend auf dem aktuellen Sonnenhöhenbereich.

#### e) **Positionsprofile**
- Jeder Rollladen hat ein eigenes Profil je Sonnenhöhe:

| Rollladen | <10° | 10–35° | 35–50° | >50° |
|----------|------|--------|--------|------|
| Rolladen 1 | 80% | 80% | 80% | 80% |
| Rolladen 3 | 80% | 80% | 60% | 40% |
| Rolladen 4 | 80% | 80% | 60% | 40% |
| Rolladen 5 | 90% | 80% | 80% | 40% |

#### f) **Hysterese-Zeit speichern**
- Speichert den aktuellen Unix-Timestamp in `input_number.last_shutter_run`.

---

## 📄 Installation

1. **Erstelle den `input_number`** (wenn noch nicht vorhanden):

   ```yaml
   # configuration.yaml
   input_number:
     last_shutter_run:
       name: "Letzte Shutter-Ausführung (Unix-Timestamp)"
       min: 0
       max: 9999999999
       step: 1
       mode: slider
   ```

2. **Gehe zu „Automations > +“ → „Edit in YAML“**

3. **Füge den gesamten Code aus dieser Datei ein** (ohne `automation:` am Anfang).

4. **Speichere und starte Home Assistant neu** (oder lade Automations neu).

5. **Teste mit „Developer Tools > YAML“ → „Validate“**.

---

## ✅ Getestet mit

- **Home Assistant OS**
- **Core 2025.12.2**
- **Supervisor 2025.12.3**
- **Operating System 16.3**
- **Frontend 20251203.2**

---

## 🛑 Keine Erfindungen

- Alle Sensoren und Entitäten entsprechen **exakt denen in der Installation vom Entwickler**.
- Keine Annahmen, keine Standardwerte – **alles basierend auf den von ihm bereitgestellten Werten**.
- Keine zusätzlichen Funktionen oder Logik hinzugefügt.

---

## 📄 Lizenz

Keine Lizenz – frei verwendbar, aber mit Namensnennung. Keine kommerzielle Nutzung erlaubt.

---

## 📬 Kontakt

Wolfgang Maurer  
Remshalden, Germany  
*(Keine öffentliche Kontaktadresse – nur für interne Nutzung)*
