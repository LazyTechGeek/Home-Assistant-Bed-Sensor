Home Assistant Bed Sensor

```
substitutions:

  name: bed-occupancy
  friendly_name: Bed Occupancy

  api_encryption_key: "YOUR_API_ENCRYPTION_KEY"
  ota_password: "YOUR_OTA_PASSWORD"
  ap_ssid: "YOUR_AP_SSID"
  ap_password: "YOUR_AP_PASSWORD"
  bed_sensor_pin: "YOUR_BED_SENSOR_PIN"

esphome:
  name: ${name}
  friendly_name: ${friendly_name}

esp8266:
  board: d1_mini

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: ${api_encryption_key}

ota:
  - platform: esphome

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: ${ap_ssid}
    password: ${ap_password}

captive_portal:

number:
  - platform: template
    name: "Occupied Delay"
    id: occupied_delay
    min_value: 0
    max_value: 30
    step: 1
    initial_value: 1
    optimistic: true
    restore_value: true
    unit_of_measurement: "s"

  - platform: template
    name: "Vacant Delay"
    id: vacant_delay
    min_value: 0
    max_value: 30
    step: 1
    initial_value: 3
    optimistic: true
    restore_value: true
    unit_of_measurement: "s"

binary_sensor:
  - platform: gpio
    pin:
      number: ${bed_sensor_pin}
      inverted: true
      mode:
        input: true
        pullup: true
    name: ${friendly_name}
    id: bed_occupancy
    device_class: occupancy

    filters:
      - delayed_on: !lambda |-
          return id(occupied_delay).state * 1000;

      - delayed_off: !lambda |-
          return id(vacant_delay).state * 1000;

switch:
  - platform: template
    name: "Enabled"
    id: bed_sensor_enabled
    optimistic: true                    # Assumes the switch changed state successfully
    restore_mode: RESTORE_DEFAULT_ON    # Remembers last state; defaults to OFF


  - platform: template
    name: "Nagging Mode"
    id: bed_nagging_mode
    optimistic: true                    # Assumes the switch changed state successfully
    restore_mode: RESTORE_DEFAULT_OFF   # Remembers last state; defaults to OFF
```
