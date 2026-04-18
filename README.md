# Marstek Jupiter C+ 🔋✨
### Intelligente ESPHome & Home Assistant Integration via Modbus RS485

Dieses Projekt bietet eine hochoptimierte, **100 % lokale** Anbindung für das **Marstek Jupiter C+** Speichersystem. Während die Standard-Lösungen oft träge sind, ermöglicht dieses Projekt eine blitzschnelle Auswertung direkt in Home Assistant – ganz ohne Cloud-Zwang.

---

## 🌟 Die "+" Features (Warum diese Integration?)

Im Gegensatz zu einfachen Modbus-Scannern bietet der **Marstek Jupiter C+** echte Smart-Features:

* **Intelligente Nacht- & Standby-Logik:** Das System erkennt automatisch, wenn keine Sonne scheint oder keine Last anliegt. Durch einen **15W-Schwellenwert** werden Messungenauigkeiten im Leerlauf ignoriert. Das Dashboard zeigt nachts sauber "Standby" an.
* **Dynamische Restlaufzeit:** Basierend auf der gewählten Akkugröße und deinem aktuellen Hausverbrauch berechnet der ESP32 sekundengenau, wie lange dein Speicher noch durchhält (in h und min).
* **Automatisches Lade-Tracking:** Erkennt sofort den Wechsel zwischen Laden und Entladen.
* **Präzise Energie-Statistiken:** Alle Erträge werden auf 2 Nachkommastellen genau geliefert, optimiert für das Home Assistant Energy Dashboard.

---

## 🛠 Hardware & Pin-Belegung

### Benötigte Komponenten
* **ESP32 DevKitC**
* **RS485-zu-TTL Adapter** ([Empfehlung: Module mit automatischer Flusssteuerung / Auto-Direction](https://www.amazon.de/dp/B099DRKBGQ?ref=ppx_yo2ov_dt_b_fed_asin_title)).
* **SP17 5 Polig Anschlussstecker Luftfahrtstecker** (https://www.amazon.de/dp/B0D92PT7W9?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1)

### Verkabelung (ESP32 zu Adapter)
Die Kommunikation nutzt die Hardware-UART des ESP32:

| ESP32 Pin | RS485 Adapter | Beschreibung |
| :--- | :--- | :--- |
| **5V / 3.3V** | VCC | Stromversorgung |
| **GND** | GND | Masse |
| **GPIO 17** | **DI** (Data Input) | TX-Leitung (Daten senden) |
| **GPIO 16** | **RO** (Receiver Output) | RX-Leitung (Daten empfangen) |

**Verbindung zum Marstek Jupiter C+:**
Verbinde die Klemmen **A+** und **B-** des Adapters mit dem RS485-Port deines Speichers (Pin 1 & 2).

---

## 📊 Ausgewertete Werte (Sensoren)

| Kategorie | Datenpunkte |
| :--- | :--- |
| **PV-Anlage** | Spannung (V), Strom (A) & Leistung (W) für **alle 4 Strings** einzeln |
| **Batterie** | SOC (%), Spannung (V), Strom (A), Kapazität (Wh) |
| **Statistik** | Erzeugung heute (kWh), Monatserzeugung (kWh), Netzeinspeisung (kWh) |
| **System** | 2x Temperaturfühler (°C), WLAN-Signalstärke, Uptime |
| **Logik** | Batterie-Status (Laden / Standby / Restzeit) |

---

## 🧠 Technische Insights (Die "Deep Dives")

### Warum 16-Bit (U_WORD) statt 32-Bit?
Obwohl Handbücher oft 32-Bit für Erträge vorschlagen, liefert der **Marstek Jupiter C+** diese Daten in stabilen 16-Bit Registern. Versuche, 32-Bit (DWORD) zu lesen, führen zu korrupten Daten und Millionen-Werten. Diese Konfiguration nutzt daher das korrekte `U_WORD` Format.

### Stabile Bus-Kommunikation
Um den "Stau" auf der Modbus-Leitung zu verhindern (`command queue empty`), nutzt dieses Projekt **entzerrte Intervalle**:
* **Fast (5s):** Kritische Watt-Werte für Live-Anzeigen.
* **Slow (32s):** Spannungen und Basis-Daten.
* **V-Slow (305s):** Temperaturen und Langzeit-Statistiken.

---

## ⚙️ Installation & Sicherheit

### ⚠️ Wichtiger Hinweis zu Secrets
Aus Sicherheitsgründen nutzt diese Konfiguration eine **`secrets.yaml`** Datei. Dies verhindert, dass deine WLAN-Zugangsdaten (SSID & Passwort) im Hauptcode stehen und versehentlich (z.B. beim Teilen deiner Config) veröffentlicht werden.

**Vorgehensweise:**
1.  Erstelle im selben Ordner wie deine YAML eine Datei namens `secrets.yaml`.
2.  Füge dort deine Daten ein:
    ```yaml
    wifi_ssid: "DEINE_WLAN_SSID"
    wifi_password: "DEIN_WLAN_PASSWORT"
    ```
3.  Die Hauptdatei (`marstek-jupiter-c-modbus.yaml`) greift dann automatisch via `!secret` auf diese Werte zu.

### Schritte:
1.  Lade die `marstek-jupiter-c-modbus.yaml` und deine `secrets.yaml` in deinen ESPHome-Ordner.
2.  Flashe den ESP32.
3.  In Home Assistant: Wähle über das Dropdown deine **Akkugröße** (z.B. 5.12 kWh) und stelle deine **Reserve** ein – erst dann wird die Restzeit präzise berechnet.

---

**Hinweis:** Dieses Projekt ist von der Community für die Community. Nutze es, um deine Energiewende ein Stück smarter zu machen! 🚀🔋
