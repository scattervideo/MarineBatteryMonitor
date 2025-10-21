
````markdown
# ⚡ Boat Battery & Bilge Monitoring System (ESPHome + ESP32)

**A smart marine monitoring node for real-time battery, bilge, and temperature data**  
Built on **ESPHome**, this system continuously tracks **voltage, current, temperature, and bilge water level**, sending secure telemetry to **Home Assistant**.

> 📰 **Learn more:**  
> See the full build guide on [ThrillFishing.com – “Never Get Stranded Again: Building the ThrillFishing Battery Monitor”](https://thrillfishing.com/never-get-stranded-again-building-the-thrillfishing-battery-monitor/)

---

## ⚙️ Quick Start Setup Guide

### 1. 🧰 Hardware Assembly
1. Print the 3D case (see [3D Printing Instructions](#-3d-printing-instructions)).  
2. Mount the ESP32, INA226 modules, Hall effect current sensor, DS18B20 probe, and DC-DC converter inside the enclosure.  
3. Wire the **12 mm stainless-steel latching push button** inline with the 5V output from the converter.  
4. Add **inline fuses** on both the 12/24V input and 5V output for protection.  
5. Connect all grounds together (converter → ESP32 → sensors).

### 2. 🔌 Flash ESPHome Firmware
1. Install ESPHome via Home Assistant or [esphome.io](https://esphome.io/).  
2. Flash the included `mercury-engine-battery.yaml` to your ESP32 board.  
3. Configure your Wi-Fi credentials and security token within the YAML file:
   ```yaml
   wifi:
     ssid: "YourNetwork"
     password: "YourPassword"
   globals:
     - id: token
       type: std::string
       initial_value: '"YOUR_SECURITY_TOKEN"'
````

4. After flashing, confirm connection in the ESPHome dashboard.

### 3. 🌐 Create a Webhook in Home Assistant

1. In Home Assistant, go to **Automations & Scenes → + Create Automation → From Scratch**.
2. Add a **Webhook trigger** and copy the webhook ID (e.g., `boat_iot_webhook`).
3. Replace that ID in the ESPHome YAML under your webhook POST URL.
4. Paste the provided [automation YAML](#️-automation-yaml) below into your automation editor.

### 4. 📈 Create Helper Entities

Add the required `input_number`, `input_text`, and `input_datetime` helpers
(see [Required Helper Entities](#-required-helper-entities)).

### 5. 🧠 Build Your Dashboard

Add a simple entities or ApexCharts card (see [Dashboard Example](#-dashboard-example)) to visualize data such as:

* Battery voltage and current
* Boat/bilge temperature
* Wi-Fi strength
* Last update time

---

## 📚 Table of Contents

* [🌐 System Communication Overview](#-system-communication-overview)

  * [🔄 Communication Paths](#-communication-paths)
  * [🚨 Important Home Assistant Automation Alerts](#-important-home-assistant-automation-alerts)
* [🧩 Overview](#-overview)
* [⚙️ Hardware Components](#️-hardware-components)
* [🔌 Power & Wiring Overview](#-power--wiring-overview)
* [🪛 Installation Notes](#-installation-notes)
* [💻 Firmware](#-firmware)
* [🏠 Home Assistant Webhook Automation](#-home-assistant-webhook-automation)

  * [🧰 Required Helper Entities](#-required-helper-entities)
  * [⚙️ Automation YAML](#️-automation-yaml)
* [🔐 Security Notes](#-security-notes)
* [🧾 Example JSON Payload](#-example-json-payload)
* [📊 Dashboard Example](#-dashboard-example)
* [🧱 3D Printing Instructions](#-3d-printing-instructions)
* [🔌 Components and Wiring Notes](#-components-and-wiring-notes)
* [📸 Future Enhancements](#-future-enhancements)
* [🧭 Author](#-author)

---

## 🌐 System Communication Overview

The system communicates over **Wi-Fi** to a remote **Home Assistant** instance via a **Nabu Casa webhook**, sending JSON data for key parameters.

* The Wi-Fi ESPHome code includes **redundant SSID failover capabilities**.

  * This allows seamless switching between **marina Wi-Fi** and a **mobile hotspot** when away.
* The webhook is protected by a **user-generated security token** for secure communication.

### 🔄 Communication Paths

There are two main data transmission paths:

1. **Interval Path** – Communicates every **2 minutes** for regular status updates.
2. **Alarm Path** – Communicates **immediately** for urgent events (e.g., bilge alarm or fault status).

### 🚨 Important Home Assistant Automation Alerts

The following automations provide critical safety and diagnostic alerts:

* ⚠️ **High Water Alert** – Bilge float switch triggered
* 🕒 **No Communication for 30 Minutes** – Possible Wi-Fi loss or device failure
* 🌡️ **High Temperature Alert** – Abnormal temperature rise detected
* 🔋 **Low Battery Voltage Alert** – Voltage drop below safe operating range

---

## 🧩 Overview

A reliable, remotely monitored **boat power management and bilge alert system** integrating:

* Dual voltage sensing
* Hall effect current monitoring
* Waterproof temperature probe
* Wi-Fi telemetry
* Fail-safe hardware ON/OFF control via a latching button

---

## ⚙️ Hardware Components

| Component              | Model / Type                         | Function                           |
| :--------------------- | :----------------------------------- | :--------------------------------- |
| **Microcontroller**    | ESP32 Development Board              | Core logic and Wi-Fi               |
| **Voltage Sensor**     | INA226 Module                        | Measures DC voltage/current        |
| **Current Sensor**     | YHDC HSTS016L                        | Split-core Hall effect sensor      |
| **Temperature Sensor** | DS18B20 Waterproof Probe             | Measures ambient/bilge temperature |
| **Bilge Sensor**       | Rule-A-Matic Plus Float Switch       | Detects high-water levels          |
| **Power Converter**    | PlusRoc 12V/24V → 5V Step-Down       | Powers ESP32 safely                |
| **Power Switch**       | 12 mm Stainless Latching Push Button | Turns 5V to ESP32 ON/OFF           |
| **Inline Fuses**       | 5A Marine Fuse Holders               | Protects voltage and power lines   |

---

## 🔌 Power & Wiring Overview

**Power Path:**
`Boat Battery (12/24 V)` → `DC-DC 5V Converter` → `Latching Switch` → `ESP32 VIN`

**Ground Path:**
All grounds must be common: converter GND → ESP32 GND → sensors.

| Function        | ESP32 Pin                  | Description       |
| :-------------- | :------------------------- | :---------------- |
| INA226 (Batt A) | SDA = GPIO21, SCL = GPIO22 | Voltage sense     |
| INA226 (Batt B) | SDA = GPIO18, SCL = GPIO4  | Secondary         |
| DS18B20         | GPIO19                     | Temperature probe |
| Bilge Switch    | GPIO14                     | Moisture float    |
| Current Sensor  | GPIO33                     | Analog input      |

---

## 🪛 Installation Notes

* Mount the **stainless-steel latching switch** near the helm or console.
* Add a **hole in the case cover** for ESP32 power LED visibility.
* Use **marine-grade wiring**, **tinned copper**, and **heat-shrink crimp connectors**.
* Include **inline fuses** on input and output power lines.

---

## 💻 Firmware

ESPHome configuration (`mercury-engine-battery.yaml`):

* INA226 sensors for voltage & current
* DS18B20 & ESP32 temperature
* Bilge float-switch detection
* Redundant Wi-Fi networks
* Automatic reboot & JSON telemetry POST to Home Assistant

---

## 🏠 Home Assistant Webhook Automation

### 🧰 Required Helper Entities

| Type           | Entity                                | Description       |
| :------------- | :------------------------------------ | :---------------- |
| Input Number   | `input_number.thrillfishing_voltage`  | Battery voltage   |
| Input Number   | `input_number.thrillfishing_voltage2` | Secondary voltage |
| Input Number   | `input_number.thrillfishing_current`  | Current draw      |
| Input Number   | `input_number.thrillfishing_temp`     | Boat temperature  |
| Input Number   | `input_number.thrillfishing_esp_temp` | ESP temperature   |
| Input Text     | `input_text.bilge_alarm`              | Bilge ON/OFF      |
| Input Text     | `input_text.boat_wifi`                | Wi-Fi SSID        |
| Input Datetime | `input_datetime.boat_last_run`        | Last update time  |

---

## ⚙️ Automation YAML

```yaml
alias: BoatIOT Update Data From Webhook
trigger:
  - platform: webhook
    webhook_id: YOUR_WEBHOOK_ID
    allowed_methods: [POST, PUT]
    local_only: false
condition:
  - condition: template
    value_template: >
      {{ trigger.json.token | string | trim == "YOUR_SECURITY_TOKEN" | string | trim }}
action:
  - service: input_number.set_value
    target: { entity_id: input_number.thrillfishing_voltage }
    data: { value: "{{ trigger.json.voltage }}" }
  - service: input_number.set_value
    target: { entity_id: input_number.thrillfishing_current }
    data: { value: "{{ trigger.json.current }}" }
  - service: input_number.set_value
    target: { entity_id: input_number.thrillfishing_temp }
    data: { value: "{{ trigger.json.temp }}" }
  - service: input_text.set_value
    target: { entity_id: input_text.bilge_alarm }
    data: { value: "{{ trigger.json.bilgealarm }}" }
  - service: input_datetime.set_datetime
    target: { entity_id: input_datetime.boat_last_run }
    data: { datetime: "{{ now() }}" }
mode: single
```

---

## 🔐 Security Notes

* Replace placeholders for `YOUR_WEBHOOK_ID` and `YOUR_SECURITY_TOKEN`.
* Use **Nabu Casa** or **HTTPS reverse proxy** for webhook protection.
* Optionally limit access by IP address or rate limiting.

---

## 🧾 Example JSON Payload

```json
{
  "voltage": "12.63",
  "current": "1.84",
  "voltage2": "12.61",
  "temp": "70.8",
  "bilgealarm": "OFF",
  "wifi": "DockNetwork",
  "esptemp": "95.4",
  "token": "YOUR_SECURITY_TOKEN"
}
```

---

## 📊 Dashboard Example

```yaml
type: entities
title: BoatIoT Telemetry
entities:
  - entity: input_number.thrillfishing_voltage
  - entity: input_number.thrillfishing_current
  - entity: input_number.thrillfishing_temp
  - entity: input_text.bilge_alarm
  - entity: input_text.boat_wifi
  - entity: input_datetime.boat_last_run
```

---

## 🧱 3D Printing Instructions

1. Import into slicer (*PrusaSlicer*, *Cura*, *Bambu Studio*).
2. **Separate / Break Apart Parts** to print individually.
3. Lay flat for best adhesion.
4. Supports: only where needed.
5. **Recommended:**

   * Layer height: `0.2 mm`
   * Infill: `20–30%`
   * Material: **PETG** or **ASA**

---

## 🔌 Components and Wiring Notes

* Always add **inline fuses**:

  * Between **battery positive → DC-DC converter input**
  * Between **converter 5V → ESP32 VIN**
* Common ground is required for all devices.
* Use **marine-grade connectors** and **tinned copper wire**.

---

## 📸 Future Enhancements

* 🔆 Solar-charging telemetry (CN3791 / TP4056)
* 📡 **LoRa or cellular data communications** for remote coverage where Wi-Fi isn’t available
* 💡 OLED/E-Ink display for onboard diagnostics
* 🌊 Smart bilge pump automation
* 🧭 GPS or motion-based theft detection

---

## 🧭 Author

**Created by:** Paul Goldstein
**Organization:** ThrillFishing Technologies
**Website:** [ThrillFishing.com](https://ThrillFishing.com)
**License:** MIT

```
