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


## Light Automation
```
alias: Bed Sensor Automation - Lights
description: ''
triggers:
  - trigger: state
    entity_id:
      - binary_sensor.bedroom_bed_sensor_occupancy
    from:
      - 'off'
    id: '1'
    to:
      - 'on'
  - trigger: state
    entity_id:
      - binary_sensor.bedroom_bed_sensor_occupancy
    from:
      - 'on'
    id: '2'
    to:
      - 'off'
conditions:
  - condition: switch.is_on
    target:
      entity_id: switch.bedroom_bed_sensor_armed
    options:
      behavior: any
      for: '00:00:00'
actions:
  - choose:
      - conditions:
          - condition: time
            after: '20:00:00'
            before: '07:00:00'
            weekday:
              - mon
              - tue
              - wed
              - thu
              - fri
              - sat
              - sun
          - condition: trigger
            id:
              - '1'
        sequence:
          - action: light.turn_off
            metadata: {}
            target:
              entity_id: light.sonoff_100062a5bc
            data: {}
          - action: switch.turn_off
            metadata: {}
            target:
              entity_id:
                - switch.sonoff_hall
                - switch.sonoff_10006bce68
            data: {}
      - conditions:
          - condition: time
            after: '20:00:00'
            before: '07:00:00'
            weekday:
              - mon
              - tue
              - wed
              - thu
              - fri
              - sat
              - sun
          - condition: trigger
            id:
              - '2'
        sequence:
          - action: light.turn_on
            metadata: {}
            target:
              entity_id: light.sonoff_100062a5bc
            data: {}
          - action: switch.turn_on
            metadata: {}
            target:
              entity_id:
                - switch.sonoff_hall
                - switch.sonoff_10006bce68
            data: {}
mode: single
```
