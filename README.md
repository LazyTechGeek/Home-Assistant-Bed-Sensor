Home Assistant Bed Sensor

```
substitutions:

  name: bed-sensor
  friendly_name: Bed Sensor

  api_encryption_key: "YOUR_API_ENCRYPTION_KEY"
  ota_password: "YOUR_OTA_PASSWORD"
  ap_ssid: "YOUR_AP_SSID"
  ap_password: "YOUR_AP_PASSWORD"
  bed_sensor_pin: "YOUR_BED_SENSOR_PIN"
  occupied_delay_initial: 1
  vacant_delay_initial: 3

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
    initial_value: ${occupied_delay_initial}
    optimistic: true
    restore_value: true
    unit_of_measurement: "s"

  - platform: template
    name: "Vacant Delay"
    id: vacant_delay
    min_value: 0
    max_value: 30
    step: 1
    initial_value: ${vacant_delay_initial}
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
    name: "Occupancy"
    id: occupancy
    device_class: occupancy

    filters:
      - delayed_on: !lambda |-
          return id(occupied_delay).state * 1000;

      - delayed_off: !lambda |-
          return id(vacant_delay).state * 1000;

switch:
  - platform: template
    name: "Armed"
    id: armed
    optimistic: true                    # Assumes the switch changed state successfully
    restore_mode: RESTORE_DEFAULT_ON    # Remembers last state; defaults to OFF


  - platform: template
    name: "Nagging Mode"
    id: mode
    optimistic: true                    # Assumes the switch changed state successfully
    restore_mode: RESTORE_DEFAULT_OFF   # Remembers last state; defaults to OFF
```


## Light Automation - Details added
```
alias: Bed Sensor - Armed & Nagging
description: ''
triggers:
  - trigger: mqtt
    options:
      topic: zigbee2mqtt/Button 02_Bed
      payload: double
    id: '1'
  - trigger: mqtt
    options:
      topic: zigbee2mqtt/Button 02_Bed
      payload: long
    id: '2'
conditions: []
actions:
  - choose:
      - conditions:
          - condition: trigger
            id:
              - '1'
        sequence:
          - choose:
              - conditions:
                  - condition: switch.is_on
                    target:
                      entity_id: switch.bedroom_bed_sensor_armed
                    options:
                      behavior: any
                      for: '00:00:00'
                sequence:
                  - action: switch.turn_off
                    metadata: {}
                    target:
                      entity_id: switch.bedroom_bed_sensor_armed
                    data: {}
                  - action: notify.notify
                    metadata: {}
                    data:
                      message: Disarming Bed Sensor
                  - action: notify.alexa_media_dave_s_echo_spot
                    metadata: {}
                    data:
                      message: Disarming Bed Sensor
                  - action: assist_satellite.announce
                    metadata: {}
                    target:
                      entity_id: >-
                        assist_satellite.home_assistant_voice_09c74b_assist_satellite
                    data:
                      message: Disarming Bed Sensor
                      preannounce: true
              - conditions:
                  - condition: switch.is_off
                    target:
                      entity_id: switch.bedroom_bed_sensor_armed
                    options:
                      behavior: any
                      for: '00:00:00'
                sequence:
                  - action: switch.turn_on
                    metadata: {}
                    target:
                      entity_id: switch.bedroom_bed_sensor_armed
                    data: {}
                  - action: notify.notify
                    metadata: {}
                    data:
                      message: Arming Bed Sensor
                  - action: notify.alexa_media_dave_s_echo_spot
                    metadata: {}
                    data:
                      message: Arming Bed Sensor
                  - action: assist_satellite.announce
                    metadata: {}
                    target:
                      entity_id: >-
                        assist_satellite.home_assistant_voice_09c74b_assist_satellite
                    data:
                      message: Arming Bed Sensor
                      preannounce: true
      - conditions:
          - condition: trigger
            id:
              - '2'
        sequence:
          - choose:
              - conditions:
                  - condition: switch.is_on
                    target:
                      entity_id: switch.bedroom_bed_occupancy_nagging_mode
                    options:
                      behavior: any
                      for: '00:00:00'
                sequence:
                  - action: switch.turn_off
                    metadata: {}
                    data: {}
                    target:
                      entity_id: switch.bedroom_bed_occupancy_nagging_mode
                  - action: notify.notify
                    metadata: {}
                    data:
                      message: Turning off Nagging Mode
                  - action: notify.alexa_media_dave_s_echo_spot
                    metadata: {}
                    data:
                      message: Turning off Nagging Mode
                  - action: assist_satellite.announce
                    metadata: {}
                    target:
                      entity_id: >-
                        assist_satellite.home_assistant_voice_09c74b_assist_satellite
                    data:
                      message: Turning off Nagging Mode
                      preannounce: true
              - conditions:
                  - condition: switch.is_off
                    target:
                      entity_id: switch.bedroom_bed_occupancy_nagging_mode
                    options:
                      behavior: any
                      for: '00:00:00'
                sequence:
                  - action: switch.turn_on
                    metadata: {}
                    data: {}
                    target:
                      entity_id: switch.bedroom_bed_occupancy_nagging_mode
                  - action: notify.notify
                    metadata: {}
                    data:
                      message: Turning on Nagging Mode
                  - action: notify.alexa_media_dave_s_echo_spot
                    metadata: {}
                    data:
                      message: Turning on Nagging Mode
                  - action: assist_satellite.announce
                    metadata: {}
                    target:
                      entity_id: >-
                        assist_satellite.home_assistant_voice_09c74b_assist_satellite
                    data:
                      message: Turning on Nagging Mode
                      preannounce: true
mode: single

```
