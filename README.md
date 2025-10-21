# ⚡ Boat Battery & Bilge Monitoring System (ESPHome + ESP32)

**Real-time battery health and bilge water monitoring system for marine applications**
Powered by **ESPHome**, this project continuously tracks **battery voltage, current, temperature, Wi-Fi signal, and bilge status**, reporting everything to **Home Assistant** via ESPHome API or HTTP webhooks.

---

## 🧩 Overview

This project is designed for boats that need **reliable 12 V/24 V battery monitoring** and **automatic bilge water detection**.
Built around an **ESP32**, the system measures:

| Sensor                        | Function                                                    |
| :---------------------------- | :---------------------------------------------------------- |
| **INA226**                    | Measures battery voltage via precision shunt resistor       |
| **YHDC HSTS016L**             | Measures current draw using a split-core Hall effect sensor |
| **DS18B20**                   | Measures temperature (inside bilge or ambient cabin)        |
| **Rule-A-Matic Float Switch** | Detects high-water levels in bilge area                     |
| **ESP32 Internal Temp**       | Monitors MCU temperature for diagnostics                    |

The system automatically:

* Sends JSON updates to Home Assistant via **Nabu Casa Webhooks**
* Restarts automatically after Wi-Fi failures or sensor dropouts
* Logs and transmits alarms when **bilge water is detected**

---

## ⚙️ Hardware Components

| Component              | Model                            | Function                                    |
| :--------------------- | :------------------------------- | :------------------------------------------ |
| **Microcontroller**    | ESP32 Development Board          | Core logic and Wi-Fi communication          |
| **Voltage Sensor**     | INA226 Voltage/Current Module    | Measures DC voltage and bus voltage         |
| **Current Sensor**     | YHDC Hall Split-Core HSTS016L    | Measures current draw without direct wiring |
| **Temperature Sensor** | DS18B20 Waterproof Probe         | Measures environmental or bilge temperature |
| **Bilge Sensor**       | Rule-A-Matic Plus Float Switch   | Triggers alarm on high water                |
| **Power Converter**    | PlusRoc DC-DC Step Down (12V→5V) | Powers ESP32 safely from boat battery       |

**Power Supply Example:**

* Input: 12 V or 24 V boat power
* Output: Regulated 5 V USB feed for ESP32

---

## 🔌 Wiring Diagram (Summary)

| Connection          | ESP32 Pin                  | Notes                                |
| :------------------ | :------------------------- | :----------------------------------- |
| INA226 (Voltage 1)  | SDA = GPIO21, SCL = GPIO22 | I²C Bus A                            |
| INA226 (Voltage 2)  | SDA = GPIO18, SCL = GPIO4  | I²C Bus B                            |
| DS18B20 Temperature | GPIO19                     | 1-Wire Bus                           |
| Bilge Float Switch  | GPIO14                     | Active-Low (with Pull-up)            |
| Current Sensor      | GPIO33                     | ADC input (voltage from Hall sensor) |

Power both sensors and ESP32 from the 5 V regulated output of the **PlusRoc DC-DC module**.
Add a **common ground** between all components.

---

## 💻 Firmware (ESPHome)

This project uses a single ESPHome YAML file named:

```
mercury-engine-battery.yaml
```

It includes:

* INA226 dual-bus voltage monitoring
* ADC current measurement (Hall effect)
* Wi-Fi signal and uptime tracking
* Dallas temperature and internal ESP temperature
* Automatic HTTP POST updates (every 2 min)
* Bilge alarm via Nabu Casa webhook
* Automatic reboot if sensors report invalid readings

⚠️ **Security Note:**
Replace the following placeholders before flashing:

* `INSERT YOUR WEBHOOK` → your Nabu Casa webhook URL
* `INSERT YOUR SECURITY TOKEN _ MAKE IT UP` → any unique token string

---

## 🌐 Home Assistant Integration

The device can be added in two ways:

1. **ESPHome Native API** – auto-discovered in Home Assistant
2. **HTTP Webhook** – sends structured JSON data for cloud or remote dashboards

**Example JSON Payload:**

```json
{
  "voltage": "12.64",
  "current": "2.47",
  "voltage2": "12.62",
  "temp": "71.3",
  "wifi": "DockNetwork",
  "wifirssi": "-62",
  "bilgealarm": "OFF",
  "esptemp": "97.4",
  "token": "YOURTOKEN123"
}
```

---

## 🧠 Features

✅ Dual-channel voltage sensing (INA226)
✅ Live current measurement with Hall effect isolation
✅ Temperature and humidity protection (waterproof probe)
✅ Bilge high-water detection with alarm webhook
✅ Self-healing Wi-Fi recovery and watchdog reboot
✅ OTA updates via ESPHome Dashboard
✅ Full compatibility with **Home Assistant**, **Node-RED**, and **MQTT bridges**

---

## 📦 Repository Structure

```
Boat-Battery-Monitor/
│
├── mercury-engine-battery.yaml      # ESPHome configuration
├── README.md                        # Project overview and documentation
└── /images/                         # Wiring diagrams and photos (optional)
```

---

## 📸 Future Enhancements

* Add solar charging telemetry (CN3791 or TP4056)
* Include GPS and motion detection for theft alerts
* Expand to multi-battery setups (start + house banks)
* Create dashboard Lovelace card for battery visualization
* Add Lora or cell data to replace WiFi

---

## 🧭 Author

**Created by:** Paul Goldstein
**Organization:** ThrillFishing Technologies
**Website:** [ThrillFishing.com](https://ThrillFishing.com)
**License:** MIT

---

Would you like me to generate a **formatted GitHub repository description** (title, topics, and short summary for the sidebar) or also include a **diagram image** (ESP32 pinout + wiring layout) in SVG/PNG for the `/images` folder?
