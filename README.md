# ⚡ Boat Battery & Bilge Monitoring System (ESPHome + ESP32)

**A smart marine monitoring node for real-time battery, bilge, and temperature data**
Built on **ESPHome**, this system continuously tracks **voltage, current, temperature, and bilge water level**, while a **stainless-steel waterproof latching push button** safely controls main system power.

---

## 🧩 Overview

This project provides a reliable, remotely monitored **boat power management and bilge alert system**.
It integrates precision current and voltage sensors, waterproof temperature sensing, and Wi-Fi telemetry — all powered by a protected 5 V supply controlled by a rugged stainless-steel **mechanical ON/OFF button**.

When the button is latched **ON**, 5 V flows to the ESP32 and the system boots automatically.
When toggled **OFF**, it completely disconnects power, ensuring **zero quiescent draw** during storage.

---

## ⚙️ Hardware Components

| Component              | Model / Type                                          | Function                                                   |
| :--------------------- | :---------------------------------------------------- | :--------------------------------------------------------- |
| **Microcontroller**    | ESP32 Development Board                               | Core processor for data collection and Wi-Fi communication |
| **Voltage Sensor**     | INA226 Voltage/Current Module                         | Measures battery voltage via I²C                           |
| **Current Sensor**     | YHDC HSTS016L Hall Split-Core                         | Monitors current draw without breaking the circuit         |
| **Temperature Sensor** | DS18B20 Waterproof Probe                              | Measures temperature (bilge or ambient)                    |
| **Bilge Sensor**       | Rule-A-Matic Plus Float Switch                        | Detects high-water level in bilge                          |
| **Power Converter**    | PlusRoc Waterproof 12 V/24 V → 5 V DC-DC Step-Down    | Provides regulated 5 V for ESP32                           |
| **Power Switch**       | 12 mm Stainless-Steel Latching Waterproof Push Button | Physically switches 5 V power to the ESP32 ON/OFF          |

---

## 🔌 Power & Wiring Overview

### ⚡ Power Distribution

1. **Boat Battery (12 V / 24 V)**
   Supplies power to the **PlusRoc DC-DC Step-Down Module**.

2. **PlusRoc DC-DC Converter**

   * Input: 12 – 24 V DC
   * Output: 5 V @ up to 3 A
   * Feeds the ESP32 and all sensors.

3. **Stainless-Steel Push Button Switch**

   * Wired **inline with the +5 V output** of the DC-DC converter.
   * When pressed **ON** (latched), 5 V passes to the ESP32 VIN pin.
   * When pressed **OFF**, the circuit opens and fully powers down the system.

### 🔧 Wiring Summary

| Function                    | ESP32 Pin                    | Description                            |
| :-------------------------- | :--------------------------- | :------------------------------------- |
| INA226 #1 (Battery A)       | SDA = GPIO 21, SCL = GPIO 22 | I²C Bus A                              |
| INA226 #2 (Battery B)       | SDA = GPIO 18, SCL = GPIO 4  | I²C Bus B                              |
| DS18B20 Temperature         | GPIO 19                      | 1-Wire temperature probe               |
| Bilge Float Switch          | GPIO 14                      | Active-low digital input with pull-up  |
| Current Sensor (YHDC)       | GPIO 33                      | Analog voltage proportional to current |
| GND                         | GND                          | Common ground for all sensors          |
| **Power Button (Latching)** | **Inline with +5 V**         | **Hardware power switch for ESP32**    |

**Power Path:**
`Boat Battery (12/24 V)` → `DC-DC 5 V Converter` → `Latching Switch` → `ESP32 VIN`

**Ground Path:**
All device grounds must be common: converter GND → ESP32 GND → sensor GNDs.

---

## 🪛 Installation Notes

* Mount the **stainless-steel button** in a visible, easily accessible location.
* Add a small hole in the case cover so you can visibly see the ESP32 power LED.
* Rated at **2 A @ 12 V/24 V DC**, sufficient for the ESP32 and small sensor loads.
* For marine safety, use **heat-shrink butt connectors** or **waterproof crimp terminals**.
* Optionally add a **fuse (500 mA–1 A)** between converter and switch for circuit protection.

---

## 💻 Firmware

The project’s ESPHome configuration (`mercury-engine-battery.yaml`) defines:

* Dual INA226 sensors for battery banks
* ADC current measurement (Hall effect)
* DS18B20 and ESP internal temperature
* Bilge float-switch alerting via webhook
* Self-healing Wi-Fi with automatic reboot
* Periodic HTTP POST telemetry to Nabu Casa or other endpoints

---

## 🌐 Integration with Home Assistant

When powered ON, the ESP32 connects automatically to Wi-Fi and exposes all sensors via ESPHome or HTTP JSON webhooks.

Example webhook payload:

```json
{
  "voltage": "12.63",
  "current": "1.84",
  "voltage2": "12.61",
  "temp": "70.8",
  "bilgealarm": "OFF",
  "wifi": "DockNetwork",
  "wifirssi": "-61",
  "esptemp": "95.4"
}
```

---

## 🧠 Features

✅ Dual-channel voltage and current monitoring
✅ Waterproof temperature probe
✅ Bilge high-water alarm
✅ Complete power isolation via stainless-steel latching button
✅ Automatic recovery from Wi-Fi faults
✅ OTA updates through ESPHome Dashboard
✅ Seamless Home Assistant integration

---

## 🧭 Safety & Best Practices

* Always disconnect boat power before wiring.
* Ensure the switch’s **12 V/24 V rating** matches your converter output.
* Use tinned marine wire and waterproof connectors.
* Add a small inline fuse to protect against short circuits.

---

## 📦 Repository Structure

```
Boat-Battery-Monitor/
│
├── mercury-engine-battery.yaml      # ESPHome configuration file
├── README.md                        # Documentation and wiring overview
└── /images/                         # Wiring diagrams and photos (optional)
```

---

## 📸 Future Enhancements

* Add solar-charging telemetry (CN3791 / TP4056)
* Add relay control for automatic bilge pump test
* Expand for dual-battery isolation management
* Optional OLED display for local voltage readout

---

## 🧭 Author

**Created by:** Paul Goldstein
**Organization:** ThrillFishing
**Website:** [ThrillFishing.com](https://ThrillFishing.com)
**License:** MIT

---
