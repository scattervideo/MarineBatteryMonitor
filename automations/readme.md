
```markdown
# 🚤 BoatIoT Webhook Data Automation (Home Assistant)

This automation processes incoming **JSON telemetry data** from the **ESP32 Boat Battery & Bilge Monitoring System** and updates a set of **Home Assistant helper sensors**.  
It ensures that all live readings—voltage, current, temperature, Wi-Fi, and bilge status—stay synchronized between your ESPHome device and your Home Assistant dashboard.

---

## 🧠 Purpose

When the ESP32 system posts telemetry via an HTTP webhook, this automation:
- Validates the incoming **security token**
- Extracts each reading (voltage, current, temp, Wi-Fi, etc.)
- Updates **input_number**, **input_text**, and **input_datetime** helpers
- Records the timestamp of the last successful update
- Provides secure and reliable real-time boat system monitoring

---

## 🧩 How It Works

1. **Trigger**  
   Activated whenever a webhook request is received at:
```

https://YOUR_HASS_URL/api/webhook/YOUR_WEBHOOK_ID

````
The ESPHome device sends data to this endpoint approximately every 2 minutes.

2. **Security Validation**  
The automation checks:
```jinja2
trigger.json.token == "YOUR SECURITY TOKEN"
````

to ensure that only authorized devices update your data.

3. **Data Mapping**
   Each field in the JSON payload updates a matching helper in Home Assistant:

   * `voltage`, `voltage2`, `current`
   * `temp`, `esptemp`
   * `wifi`, `bilgealarm`
   * Timestamp of last update

---

## 🧰 Required Helper Entities

Before enabling the automation, create these helpers in Home Assistant
(**Settings → Devices & Services → Helpers → + Create Helper**).

### 🧮 Input Numbers

| Entity ID                             | Name              | Unit | Description                                 |
| :------------------------------------ | :---------------- | :--- | :------------------------------------------ |
| `input_number.thrillfishing_voltage`  | Battery Voltage   | V    | Primary battery voltage                     |
| `input_number.thrillfishing_voltage2` | Battery Voltage 2 | V    | Secondary or auxiliary bank voltage         |
| `input_number.thrillfishing_current`  | Battery Current   | A    | Current flow from Hall sensor               |
| `input_number.thrillfishing_temp`     | Boat Temperature  | °F   | Ambient or bilge temperature                |
| `input_number.thrillfishing_esp_temp` | ESP Temperature   | °F   | ESP32 internal temperature                  |
| `input_number.boat_uptime`            | Boat Uptime       | sec  | Optional uptime value (can remain disabled) |

### 🧾 Input Texts

| Entity ID                | Name            | Description                              |
| :----------------------- | :-------------- | :--------------------------------------- |
| `input_text.bilge_alarm` | Bilge Alarm     | “ON” / “OFF” state of bilge float switch |
| `input_text.boat_wifi`   | Boat Wi-Fi SSID | Connected network name                   |

### ⏱️ Input Datetime

| Entity ID                      | Name                | Description                                      |
| :----------------------------- | :------------------ | :----------------------------------------------- |
| `input_datetime.boat_last_run` | Last BoatIoT Update | Records the timestamp of the last webhook update |

---

## ⚙️ Automation YAML

```yaml
alias: BoatIOT Update Data From Webhook
description: "Updates all BoatIoT helper sensors when webhook is received"
trigger:
  - platform: webhook
    webhook_id: YOUR WEBHOOK ID
    allowed_methods: [POST, PUT]
    local_only: false

condition:
  - condition: template
    alias: Check for token
    value_template: >
      {{ trigger.json.token | string | trim == "YOUR SECURITY TOKEN" | string | trim }}

action:
  # Boat temperature
  - alias: Thrillfishing Temp
    if:
      - condition: template
        value_template: "{{ trigger.json.temp is defined }}"
    then:
      - service: input_number.set_value
        data:
          value: "{{ trigger.json.temp }}"
        target:
          entity_id: input_number.thrillfishing_temp

  # Voltage readings
  - alias: Thrillfishing Voltage
    if:
      - condition: template
        value_template: "{{ trigger.json.voltage is defined }}"
    then:
      - service: input_number.set_value
        data:
          value: "{{ trigger.json.voltage }}"
        target:
          entity_id: input_number.thrillfishing_voltage

  - alias: Thrillfishing Voltage2
    if:
      - condition: template
        value_template: "{{ trigger.json.voltage2 is defined }}"
    then:
      - service: input_number.set_value
        data:
          value: "{{ trigger.json.voltage2 }}"
        target:
          entity_id: input_number.thrillfishing_voltage2

  # Current & internal temperature
  - alias: Thrillfishing Current
    if:
      - condition: template
        value_template: "{{ trigger.json.current is defined }}"
    then:
      - service: input_number.set_value
        data:
          value: "{{ trigger.json.current }}"
        target:
          entity_id: input_number.thrillfishing_current

  - alias: Thrillfishing ESP Temp
    if:
      - condition: template
        value_template: "{{ trigger.json.esptemp is defined }}"
    then:
      - service: input_number.set_value
        data:
          value: "{{ trigger.json.esptemp }}"
        target:
          entity_id: input_number.thrillfishing_esp_temp

  # Bilge alarm
  - alias: Thrillfishing Bilge Alarm
    - service: input_text.set_value
      data:
        value: "{{ trigger.json.bilgealarm }}"
      target:
        entity_id: input_text.bilge_alarm

  # Wi-Fi info
  - alias: Boat WiFi SSID
    - service: input_text.set_value
      data:
        value: "{{ trigger.json.get('wifi', 'No WiFi data') }}"
      target:
        entity_id: input_text.boat_wifi

  # Timestamp of last update
  - alias: Boat Last Update
    - service: input_datetime.set_datetime
      data:
        datetime: "{{ now() }}"
      target:
        entity_id: input_datetime.boat_last_run

mode: single
```

---

## 🔐 Security Notes

* Replace both `YOUR WEBHOOK ID` and `YOUR SECURITY TOKEN` with your actual secrets.
* Never expose your webhook publicly. Use **Nabu Casa**, **Cloudflare Tunnel**, or **HTTPS reverse proxy**.
* Consider adding **IP filtering** or **rate limits** if hosting externally.

---

## 🧾 Example JSON Payload

```json
{
  "voltage": "12.64",
  "current": "2.35",
  "voltage2": "12.61",
  "temp": "72.3",
  "wifi": "DockNetwork",
  "bilgealarm": "OFF",
  "esptemp": "97.5",
  "token": "YOUR SECURITY TOKEN"
}
```

---

## 📊 Dashboard Example

You can visualize the helpers in a **Lovelace Entities Card** or **Mushroom Entities Card**:

```yaml
type: entities
title: BoatIoT Telemetry
entities:
  - entity: input_number.thrillfishing_voltage
    name: Battery Voltage
  - entity: input_number.thrillfishing_voltage2
    name: Battery Voltage 2
  - entity: input_number.thrillfishing_current
    name: Battery Current
  - entity: input_number.thrillfishing_temp
    name: Boat Temperature
  - entity: input_text.bilge_alarm
    name: Bilge Status
  - entity: input_text.boat_wifi
    name: Connected Wi-Fi
  - entity: input_datetime.boat_last_run
    name: Last Update
```

---

## 🧭 Author

**Created by:** Paul Goldstein
**Organization:** ThrillFishing Technologies
**Website:** [ThrillFishing.com](https://ThrillFishing.com)
**License:** MIT

```
