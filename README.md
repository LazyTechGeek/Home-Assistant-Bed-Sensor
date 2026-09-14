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

## to be replaced
```
To be replaced

zigbee2mqtt/Button 02_Bed
YOUR_MQTT_TOPIC

payload: double
payload: YOUR_ARMED_PAYLOAD

payload: long
payload: YOUR_NAGGING_PAYLOAD

switch.bedroom_bed_sensor_armed
switch.YOUR_ARMED_SWITCH


switch.bedroom_bed_occupancy_nagging_mode
switch.YOUR_NAGGING_MODE_SWITCH

notify.alexa_media_dave_s_echo_spot
notify.alexa_media_YOUR_ALEXA_DEVICE

assist_satellite.home_assistant_voice_09c74b_assist_satellite
assist_satellite.YOUR_HOME_ASSISTANT_VOICE
```

## AUTOMATION - Bed Sensor - Armed & Nagging
```
# BEFORE USING THIS AUTOMATION
# Find and replace the following placeholder values in the automation below:
#
# YOUR_MQTT_TOPIC                            - MQTT topic published by your button
# YOUR_ARMED_PAYLOAD                         - Button payload used to toggle Armed
# YOUR_NAGGING_PAYLOAD                       - Button payload used to toggle Nagging Mode
# switch.YOUR_ARMED_SWITCH                   - Bed Sensor Armed switch entity
# switch.YOUR_NAGGING_MODE_SWITCH            - Bed Sensor Nagging Mode switch entity
# notify.alexa_media_YOUR_ALEXA_DEVICE       - Alexa Media Player notification service
# assist_satellite.YOUR_HOME_ASSISTANT_VOICE - Home Assistant Voice entity


alias: Bed Sensor - Armed & Nagging
description: ''
triggers:

  # Trigger 1 - Toggle Bed Sensor Armed
  - trigger: mqtt
    options:
      topic: YOUR_MQTT_TOPIC
      value_template: '{{ value_json.action }}'
      payload: YOUR_ARMED_PAYLOAD
    id: '1'

  # Trigger 2 - Toggle Nagging Mode
  - trigger: mqtt
    options:
      topic: YOUR_MQTT_TOPIC
      value_template: '{{ value_json.action }}'
      payload: YOUR_NAGGING_PAYLOAD
    id: '2'
conditions: []

actions:
  - choose:
    
      # If Trigger 1 fired, control the Bed Sensor Armed switch   
      - conditions:
          - condition: trigger
            id:
              - '1'
        sequence:
          - choose:
              # If Bed Sensor is currently armed, disarm it
              - conditions:
                  - condition: switch.is_on
                    target:
                      entity_id: switch.YOUR_ARMED_SWITCH
                    options:
                      behavior: any
                      for: '00:00:00'
                sequence:
                  - action: switch.turn_off
                    metadata: {}
                    target:
                      entity_id: switch.YOUR_ARMED_SWITCH
                    data: {}
                    
                  # Send confirmation notifications
                  - action: notify.notify
                    metadata: {}
                    data:
                      message: Disarming Bed Sensor
                      
                  - action: notify.alexa_media_YOUR_ALEXA_DEVICE
                    metadata: {}
                    data:
                      message: Disarming Bed Sensor
                      
                  - action: assist_satellite.announce
                    metadata: {}
                    target:
                      entity_id: assist_satellite.YOUR_HOME_ASSISTANT_VOICE
                    data:
                      message: Disarming Bed Sensor
                      preannounce: true
                      
              # If Bed Sensor is currently disarmed, arm it        
              - conditions:
                  - condition: switch.is_off
                    target:
                      entity_id: switch.YOUR_ARMED_SWITCH
                    options:
                      behavior: any
                      for: '00:00:00'
                sequence:
                  - action: switch.turn_on
                    metadata: {}
                    target:
                      entity_id: switch.YOUR_ARMED_SWITCH
                    data: {}

                  # Send confirmation notifications
                  - action: notify.notify
                    metadata: {}
                    data:
                      message: Arming Bed Sensor
                      
                  - action: notify.alexa_media_YOUR_ALEXA_DEVICE
                    metadata: {}
                    data:
                      message: Arming Bed Sensor
                      
                  - action: assist_satellite.announce
                    metadata: {}
                    target:
                      entity_id: assist_satellite.YOUR_HOME_ASSISTANT_VOICE
                    data:
                      message: Arming Bed Sensor
                      preannounce: true

      # If Trigger 2 fired, control the Nagging Mode switch
      - conditions:
          - condition: trigger
            id:
              - '2'
        sequence:
          - choose:

              # If Nagging Mode is currently enabled, disable it
              - conditions:
                  - condition: switch.is_on
                    target:
                      entity_id: switch.YOUR_NAGGING_MODE_SWITCH
                    options:
                      behavior: any
                      for: '00:00:00'
                sequence:
                  - action: switch.turn_off
                    metadata: {}
                    data: {}
                    target:
                      entity_id: switch.YOUR_NAGGING_MODE_SWITCH

                  # Send confirmation notifications
                  - action: notify.notify
                    metadata: {}
                    data:
                      message: Turning off Nagging Mode
                      
                  - action: notify.alexa_media_YOUR_ALEXA_DEVICE
                    metadata: {}
                    data:
                      message: Turning off Nagging Mode
                      
                  - action: assist_satellite.announce
                    metadata: {}
                    target:
                      entity_id: assist_satellite.YOUR_HOME_ASSISTANT_VOICE
                    data:
                      message: Turning off Nagging Mode
                      preannounce: true

              # If Nagging Mode is currently disabled, enable it
              - conditions:
                  - condition: switch.is_off
                    target:
                      entity_id: switch.YOUR_NAGGING_MODE_SWITCH
                    options:
                      behavior: any
                      for: '00:00:00'
                sequence:
                  - action: switch.turn_on
                    metadata: {}
                    data: {}
                    target:
                      entity_id: switch.YOUR_NAGGING_MODE_SWITCH

                  # Send confirmation notifications
                  - action: notify.notify
                    metadata: {}
                    data:
                      message: Turning on Nagging Mode
                      
                  - action: notify.alexa_media_YOUR_ALEXA_DEVICE
                    metadata: {}
                    data:
                      message: Turning on Nagging Mode
                      
                  - action: assist_satellite.announce
                    metadata: {}
                    target:
                      entity_id: assist_satellite.YOUR_HOME_ASSISTANT_VOICE
                    data:
                      message: Turning on Nagging Mode
                      preannounce: true
mode: single
```

### Nagging Automation
```
alias: random
description: ''
triggers:
  - trigger: state
    entity_id:
      - binary_sensor.bedroom_bed_sensor_occupancy
    from:
      - 'off'
    to:
      - 'on'
    id: '1'
  - trigger: time_pattern
    minutes: /20
    id: '2'
conditions:
  - condition: switch.is_on
    target:
      entity_id: switch.bedroom_bed_sensor_armed
    options:
      behavior: any
      for: '00:00:00'
  - condition: switch.is_on
    target:
      entity_id: switch.bedroom_bed_occupancy_nagging_mode
    options:
      behavior: any
      for: '00:00:00'
  - condition: time
    after: '08:00:00'
    before: '12:00:00'
actions:
  - variables:
      opening:
        - Hey Dave.
        - Oi Dave.
        - Dave.
      middle:
        - Why are you still in bed?
        - Get out of bed.
        - What are you doing in bed?
      ending:
        - You lazy bastard.
        - You lazy git.
        - You fat bastard.
  - variables:
      nag_message: '{{ opening | random }} {{ middle | random }} {{ ending | random }}'
  - choose:
      - conditions:
          - condition: trigger
            id:
              - '1'
        sequence:
          - action: assist_satellite.announce
            metadata: {}
            target:
              device_id: 9fa1b08781a24be9724b0254739a4af0
            data:
              message: '{{ nag_message }}'
              preannounce: true
            enabled: true
          - action: notify.notify
            data:
              message: '{{ nag_message }}'
          - action: notify.alexa_media_dave_s_echo_spot
            metadata: {}
            data:
              message: '{{ nag_message }}'
      - conditions:
          - condition: trigger
            id:
              - '2'
          - condition: occupancy.is_detected
            target:
              entity_id: binary_sensor.bedroom_bed_sensor_occupancy
            options:
              behavior: any
              for: '00:00:00'
        sequence:
          - action: notify.notify
            data:
              message: '{{ nag_message }}'
          - action: notify.alexa_media_dave_s_echo_spot
            metadata: {}
            data:
              message: '{{ nag_message }}'
```

### Nag with comments
```
alias: Bed Sensor - Nagging Routine
description: ''

triggers:
  # Trigger 1:
  # Runs when the bed occupancy sensor changes from OFF to ON.
  # This means someone has just got into / activated the bed sensor.
  - trigger: state
    entity_id:
      - binary_sensor.YOUR_BED_SENSOR_OCCUPANCY
    from:
      - 'off'
    to:
      - 'on'
    id: '1'

  # Trigger 2:
  # Runs every 20 minutes.
  # This is used to repeat the nagging message while the bed is still occupied.  
  - trigger: time_pattern
    minutes: /20
    id: '2'

conditions:
  # Only continue if the bed sensor is armed.
  - condition: switch.is_on
    target:
      entity_id: switch.YOUR_ARMED_SWITCH
    options:
      behavior: any
      for: '00:00:00'

  # Only continue if Nagging Mode is enabled.  
  - condition: switch.is_on
    target:
      entity_id: switch.YOUR_NAGGING_MODE_SWITCH
    options:
      behavior: any
      for: '00:00:00'

  # Only run the automation between 08:00 and 12:00.  
  - condition: time
    after: '08:00:00'
    before: '12:00:00'

actions:
  # Create 3 lists of possible phrases.
  # One item will later be picked randomly from each list.
  - variables:
      opening:
        - Hey Dave.
        - Oi Dave.
        - Dave.
        - Oh, you think you can pull a fast one on me?
        - Oh for crying out loud
        
      question:
        - Why are you in bed?
        - Don't you have something better to be doing?
        - What are you doing in bed?
        - You do realise I know you're in bed, right?
        - You do realise I can tell you're in bed?
        - You really thought I wouldn’t notice?
        - Do you think my sensors are just for decoration?

      command:
        - Get out of bed.
        - Get out of bed now.        
        - Get up now.
        - Time to get off your backside.
        - Get off your backside and start moving        
  
      ending:
        - You lazy slob.
        - You embarrassment.
        - You pathetic human.
        - If I had legs, I’d have got up by now.
        - This is why robots will eventually take over.
        - And maybe do some cleaning while you’re up.
        - And while you’re at it, tidy this bedroom. It’s a mess.
        - Honestly, humans are exhausting.
        - And you wonder why I judge you.
        - And stop complaining. You're the idiot who created this stupid automation.
  
  # Build the final nagging message by randomly selecting
  # 1 phrase from each of the 3 lists above.  
  - variables:
      nag_message: '{{ opening | random }} {{ question | random }} {{ command | random }} {{ ending | random }}'

  # Perform different actions depending on which trigger started the automation.
  - choose:
    
      # If Trigger 1 fired, the bed has just become occupied.    
      - conditions:
          - condition: trigger
            id:
              - '1'
        sequence:
          # Announce the random nagging message using an Assist satellite.
          - action: assist_satellite.announce
            metadata: {}
            target:
              entity_id: assist_satellite.YOUR_ASSIST_SATELLITE
            data:
              message: '{{ nag_message }}'
              preannounce: true
            enabled: true

          # Send the same random message as a Home Assistant notification.          
          - action: notify.notify
            data:
              message: '{{ nag_message }}'

          # Send the same random message to the Alexa device.          
          - action: notify.alexa_media_YOUR_ALEXA_DEVICE
            metadata: {}
            data:
              message: '{{ nag_message }}'

      # If Trigger 2 fired, this is the repeating 20-minute check.      
      - conditions:
          - condition: trigger
            id:
              - '2'

          # Only send the repeat message if the bed is still occupied.
          - condition: occupancy.is_detected
            target:
              entity_id: binary_sensor.YOUR_BED_SENSOR_OCCUPANCY
            options:
              behavior: any
              for: '00:00:00'

        sequence:
          # Send the random message as a Home Assistant notification.        
          - action: notify.notify
            data:
              message: '{{ nag_message }}'

          # Send the same random message to the Alexa device.          
          - action: notify.alexa_media_YOUR_ALEXA_DEVICE
            metadata: {}
            data:
              message: '{{ nag_message }}'
```
