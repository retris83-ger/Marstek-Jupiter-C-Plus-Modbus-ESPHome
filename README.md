# Marstek Jupiter C+ 🔋✨
### Intelligente ESPHome & Home Assistant Integration via Modbus RS485

Dieses Projekt bietet eine hochoptimierte, **100 % lokale** Anbindung für das **Marstek Jupiter C+** Speichersystem. Während Standard-Apps oft träge sind oder Cloud-Zwang bedeuten, ermöglicht dieses Projekt eine blitzschnelle Auswertung und smarte Zusatzfunktionen direkt in Home Assistant.

---

## 🌟 Die "+" Features (Warum diese Integration?)

Im Gegensatz zu einfachen Auslese-Skripten bietet der **Marstek Jupiter C+** echte Smart-Features:

* **Intelligente Nacht- & Standby-Logik:** Erkennt automatisch, wenn keine Sonne scheint oder keine Last anliegt. Durch einen **15W-Schwellenwert** werden Messungenauigkeiten im Leerlauf ignoriert. Das Dashboard zeigt nachts sauber "Standby" an.
* **Dynamische Restlaufzeit:** Basierend auf der gewählten Akkugröße und dem Hausverbrauch berechnet der ESP32 sekundengenau, wie lange der Speicher noch durchhält (Anzeige in `h` und `min`).
* **Präzises 4-String Monitoring:** Jeder der 4 PV-Eingänge wird separat überwacht.
* **Stabilisierte Solar-Messung:** Die Solarproduktion wird für maximale Stabilität über die Summe der Einzelstrings (PV1-PV4) berechnet und integriert, was deutlich zuverlässiger ist als die internen Register des Speichers.

---

## ⚡ Home Assistant Energy Dashboard
Dieses Projekt ist darauf optimiert, deinen Speicher perfekt im offiziellen **Energie-Dashboard** von Home Assistant abzubilden. Für maximale Stabilität nutzen wir hierfür die summierten Sensoren:

* **Solarproduktion:** `sensor.marstek_jupiter_c_solar_gesamtleistung`
* **Batteriespeicher (Eingehend):** `sensor.marstek_jupiter_c_batterie_energie_geladen`
* **Batteriespeicher (Ausgehend):** `sensor.marstek_jupiter_c_batterie_energie_entladen`

---

## 🛠 Benötigte Komponenten & Hardware

Für den Aufbau werden folgende Komponenten benötigt:

1.  **ESP32 DevKitC** (Oder vergleichbarer ESP32 Microcontroller).
2.  **RS485-zu-TTL Adapter**: [Amazon Link](https://www.amazon.de/dp/B099DRKBGQ?ref=ppx_yo2ov_dt_b_fed_asin_title) (Wichtig für die Kommunikation zwischen ESP und Speicher).
3.  **RS485-Verbindungsstecker**: [Amazon Link](https://www.amazon.de/dp/B0D92PT7W9?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1) (Dieser Stecker passt exakt in den Modbus-Port des Marstek Jupiter C+).

### Verkabelung (ESP32 zu Adapter)
| ESP32 Pin | RS485 Adapter | Beschreibung |
| :--- | :--- | :--- |
| **5V / 3.3V** | VCC | Stromversorgung |
| **GND** | GND | Masse |
| **GPIO 17** | **DI** (Data Input) | TX-Leitung (Daten senden) |
| **GPIO 16** | **RO** (Receiver Output) | RX-Leitung (Daten empfangen) |

**Verbindung zum Marstek:**
Verbinde **A+** und **B-** des Adapters über den oben verlinkten Stecker mit dem RS485-Port des Speichers.

---

## ⚙️ Installation & Sicherheit

### 1. Nutzung von Secrets
Diese Konfiguration nutzt eine **`secrets.yaml`**, um deine WLAN-Daten zu schützen. Erstelle im ESPHome-Verzeichnis eine Datei namens `secrets.yaml`:
```yaml
wifi_ssid: "DEINE_SSID"
wifi_password: "DEIN_PASSWORT"
```

### 2. Statische IP-Adresse (Wichtig!)
In der Datei `marstek-jupiter-c-modbus.yaml` ist in **Zeile 44** eine feste IP-Adresse (`192.168.201.93`) vordefiniert. 
* **Anpassen:** Ändere diese Adresse auf eine freie IP in deinem Netzwerk.
* **DHCP:** Falls du keine feste IP möchtest, lösche die Zeilen 43 bis 46 (`manual_ip` Block).

### 3. Schritte:
1. Lade die YAML-Dateien in deinen ESPHome-Ordner.
2. Flashe den ESP32.
3. Wähle in Home Assistant nach dem ersten Start unter Einstellungen/Geräte&Dienste/ESPHome:
    * **Akkugröße Auswahl** (`select.marstek_jupiter_c_akkugrosse_auswahl`)
    * **Entladestopp Reserve** (`number.marstek_jupiter_c_batterie_entladestopp_reserve`)
    * Batterie Entladestopp Reserve muss zwingend manuel eingetragen werden da dieser Wert nicht auslesbar ist !!! Dieser dient der Berechnung der Restenergiezeit.

---

## 🎨 Dashboard (Vorschlag)

Hier ist ein modernes Layout mit **Mushroom Cards** und **ApexCharts**.

<details>
<summary>👉 Dashboard-Code ausklappen</summary>

```yaml
type: sections
sections:
  - type: grid
    cards:
      - type: vertical-stack
        cards:
          - type: heading
            heading: ⚡ Marstek Jupiter C+
            heading_style: title
          - type: grid
            columns: 4
            square: false
            cards:
              - type: custom:mushroom-template-card
                layout: vertical
                entity: sensor.marstek_jupiter_c_solar_gesamtleistung
                primary: '{{ states(entity) | float(0) | round(0) }} W'
                secondary: >
                  {% set lade = states('sensor.marstek_jupiter_c_batterie_ladeleistung') | float(0) %} 
                  {% if lade > 0 %}↑ {{ lade | round(0) }}W{% else %}PV Ertrag{% endif %}
                icon: mdi:solar-power
                icon_color: orange
              - type: custom:mushroom-template-card
                entity: sensor.marstek_jupiter_c_batterie_soc
                primary: '{{ states(entity) | float(0) | round(0) }} %'
                secondary: '{{ states("sensor.marstek_jupiter_c_batterie_status_restzeit") }}'
                icon: |
                  {% set soc = states(entity) | int(0) %}
                  {% if soc > 90 %} mdi:battery
                  {% elif soc > 70 %} mdi:battery-70
                  {% elif soc > 40 %} mdi:battery-40
                  {% else %} mdi:battery-20
                  {% endif %}
                icon_color: >
                  {% set lade = states('sensor.marstek_jupiter_c_batterie_ladeleistung') | float(0) %}
                  {% if lade > 0 %} cyan {% elif states(entity) | int(0) > 20 %} green {% else %} red {% endif %}
                vertical: true
              - type: custom:mushroom-template-card
                layout: vertical
                entity: sensor.marstek_jupiter_c_netz_ausgangsleistung
                primary: '{{ states(entity) | float(0) | round(0) }} W'
                secondary: 'Haus Netz'
                icon: mdi:transmission-tower
                icon_color: blue
              - type: custom:mushroom-template-card
                layout: vertical
                entity: sensor.marstek_jupiter_c_solar_erzeugung_gesamt
                primary: '{{ states(entity) | float(0) | round(2) }} kWh'
                secondary: Tag
                icon: mdi:lightning-bolt
                icon_color: purple
```
</details>

---

**Hinweis:** Dieses Projekt ist von der Community für die Community. Nutze es, um deine Energiewende ein Stück smarter zu machen! 🚀🔋
