# Automatiseringer
## 1. Personstyring
- Installer HomeAssistant på mobiltelefon
  

## 2. Entre
* Link til esp32 projekt  *(kommer senere)*
* YAML kode ved bevægelse i entre
```YAML
alias: Bevægelse Entre 
description: ""
trigger:
  - platform: state
    entity_id: binary_sensor.pir_sensor
    from: "off"
    to: "on"
    for:
      hours: 0
      minutes: 0
      seconds: 0
condition:
  - condition: state
    entity_id: sun.sun
    state: below_horizon
action:
  - choose:
      - conditions:
          - condition: time
            after: "00:00"
            before: "06:00"
        sequence:
          - type: turn_on
            device_id: 15dce4b5bd0e93580a48dfcf0735788a
            entity_id: light.entre_ikea_806lm_level_on_off
            domain: light
            brightness_pct: 10
    default:
      - type: turn_on
        device_id: 15dce4b5bd0e93580a48dfcf0735788a
        entity_id: light.entre_ikea_806lm_level_on_off
        domain: light
        brightness_pct: 80
  - wait_for_trigger:
      - platform: state
        entity_id: binary_sensor.pir_sensor
        from: "on"
        to: "off"
        for:
          hours: 0
          minutes: 0
          seconds: 0
  - type: turn_off
    device_id: 15dce4b5bd0e93580a48dfcf0735788a
    entity_id: light.entre_ikea_806lm_level_on_off
    domain: light
mode: single
```
* YAML kode ved tryk på button_1 tænd lys manuelt
```YAML
alias: Entre lys
description: ""
trigger:
  - platform: state
    entity_id: binary_sensor.entre_button_1
    from: "off"
    to: "on"
condition: []
action:
  - choose:
      - conditions:
          - condition: device
            type: is_off
            device_id: 15dce4b5bd0e93580a48dfcf0735788a
            entity_id: light.entre_ikea_806lm_level_on_off
            domain: light
        sequence:
          - type: turn_on
            device_id: 15dce4b5bd0e93580a48dfcf0735788a
            entity_id: light.entre_ikea_806lm_level_on_off
            domain: light
            brightness_pct: 100
    default:
      - type: turn_off
        device_id: 15dce4b5bd0e93580a48dfcf0735788a
        entity_id: light.entre_ikea_806lm_level_on_off
        domain: light
mode: single
```

## 3. Postkasse alarm
- Aqara vibrationscensor Zigbee monteret på postkasselåg
- YAML kode
```YAML
alias: Post
description: ""
trigger:
  - type: vibration
    platform: device
    device_id: 720f09636613695b19483d8538c5e0ab
    entity_id: binary_sensor.lumi_lumi_vibration_aq1_ias_zone
    domain: binary_sensor
condition: []
action:
  - service: notify.notifier_agurk
    data:
      message: Ny post
  - service: input_boolean.turn_on
    data: {}
    target:
      entity_id: input_boolean.postkasse
mode: single
```

## 4. Vaskemaskine færdig
- Shelly 1PM Power måler strømforbrug
- Hvis strømforbrug falder til under 5W i 3 min. anses vask for færdig
![](/My_Homeassistant/Automatiseringer/Images/vaskemaskine.png)
- YAML kode for Blueprint
```YAML
alias: Vaskemaskine færdig
description: ""
use_blueprint:
  path: >-
    sbyx/notify-or-do-something-when-an-appliance-like-a-dishwasher-or-washing-machine-finishes.yaml
  input:
    power_sensor: sensor.shelly1pm_8caab55fd8f1_power
    actions:
      - service: notify.notifier_agurk
        data:
          message: Vaskemaskine færdig
      - service: notify.mobile_app_lissi_iphone
        data:
          message: Vaskemaskine færsig
    starting_hysteresis: 3
    finishing_threshold: 5
    finishing_hysteresis: 3

```