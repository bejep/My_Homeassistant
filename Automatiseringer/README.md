# Automatiseringer
## 1. Personstyring (Device tracker)<br>
Inspiration:<br>
[Switch Based On Presence In Home Assistant](https://www.youtube.com/watch?v=J-b8BAefNGQ)<br>
[Geofencing in Home Assistant](https://www.youtube.com/watch?v=pjAyRN5UiBg)<br>
[Ultimate Presence Detection in Home Assistant](https://www.youtube.com/watch?v=AcxHt_bPlZQ)<br>
<br>
**Setup**
- Homeassistant<br>
  - Indstillinger<br>
    - Vælg Personer<br>
      - Tilføj person - vælg enhed som skal spores

<img src="./Images/Person.png" width=35% height=35%>

- Installer **HomeAssistant** på mobiltelefon<br>
  Under indstillinger vælges **Companion App**<br>
  Vælg **Lokalitet** og giv tilladelser

<img src="./Images/Companion1.jpg" width=35% height=35%>

- **ESPresense** (en anden Decive tracker)
  - se [**6. ESPresense og MQTT**](../README.md#6-espresense-og-mqtt)<br><br>

- **Definer gruppe** (/config/groups.yaml)<br>
  Hvis flere personer skal kunne trackes samtidig kan der oprettes en gruppe<br>
  Hvis 'all: false' skal kun 1 være hjemme<br>
  Hvis 'all: true' skal alle være hjemme<br>

<img src="./Images/groups.png" width=35% height=35%>

- entity: group.somebody home<br>
<img src="./Images/hjemme.png" width=50% height=50%>

- **Eksempel på brug af automatisering**
  - Person kommer hjem og lys tændes hvis sol er gået ned. (kunne også teste på ’somebody home’)
  
```YAML
alias: Bent hjemme
description: Bent hjemme eller ude
trigger:
  - platform: state
    entity_id:
      - person.bent
    from: not_home
    to: home
    id: bent hjemme
  - platform: state
    entity_id:
      - person.bent
    id: bent ude
    from: home
    to: not_home
condition: []
action:
  - choose:
      - conditions:
          - condition: trigger
            id: bent hjemme
        sequence:
          - if:
              - condition: state
                entity_id: sun.sun
                state: below_horizon
            then:
              - type: turn_on
                device_id: 15dce4b5bd0e93580a48dfcf0735788a
                entity_id: light.entre_ikea_806lm_level_on_off
                domain: light
                brightness_pct: 100
      - conditions:
          - condition: trigger
            id: bent ude
        sequence:
          - type: turn_off
            device_id: 4bc5cef82ee22c1271d1d25126d5f2e6
            entity_id: switch.light_relay_1
            domain: switch
    default:
      - stop: ukendt betingelse
mode: single
```

## 2. Entre
* [Link til esp32 projekt](./../ESPHome/README.md#1-styringspanel-esp8266-12-m-oled-lcd)
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
  - condition: or
    condition:
      - condition: nummeric_state
        entity_id: sensor.light_2
        below: 50
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
* YAML kode ved tryk på **button_1** tænd lys manuelt
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
- Aqara vibrationscensor Zigbee monteret på postkasselåg.<br>
  For at forbedre rækkevidden på Zigbee signalet, sidder der en Ikea signalforstærker på indermuren tæt på postkassen.<br>
  Ulempen ved vibrationssensor er at kraftig blæst kan give falske meldinger.<br>
  Ud over melding på Lovelace sendes en melding via Telegram.
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
- Shelly 1PM Power måler strømforbrug<br>
  Hvis strømforbrug falder til under 5W i 3 min. anses vask for færdig<br>
![](./Images/vaskemaskine.png)
- YAML kode for Blueprint
```YAML
alias: Vaskemaskine færdig 
description: ""
use_blueprint:
  path: >-
    sbyx/notify-or-do-something-when-an-appliance-like-a-dishwasher-or-washing-machine-finishes.yaml
  input:
    power_sensor: sensor.shellyplus1pm_d4d4da7c99f8_switch_0_power
    actions:
      - service: notify.notifier_agurk
        data:
          message: Vaskemaskine færdig
      - service: notify.mobile_app_lissi_iphone
        data:
          message: Vaskemaskine færdig
    starting_hysteresis: 3
    finishing_threshold: 5
    finishing_hysteresis: 3

```

## 5. Affaldstømning
- HACS for Affaldshåndtering DK installeret<br>
- Følg vejledning i dokumentation for Affaldshåndtering DK<br>
- Dagen før kl. 18 sendes en meddelelse via Telegram<br>
- YAML kode
```YAML
alias: affald
description: ""
triggers:
  - at: "18:00:00"
    trigger: time
conditions: []
actions:
  - choose:
      - conditions:
          - condition: template
            value_template: >-
              {{ is_state("sensor.Affalddk_fengersvej_31_pap_papir_glas_Metal",
              "1") }}
        sequence:
          - data:
              message: I morgen hentes mad  metal og papir
            action: notify.notifier_agurk
      - conditions:
          - condition: template
            value_template: >-
              {{
              is_state_attr("sensor.Affalddk_fengersvej_31_plast_mad_drikkekartoner",
              "1") }}
        sequence:
          - data:
              message: I morgen hentes mad og plast
            action: notify.notifier_agurk
    default: []
mode: single
```

## 6. Julelys automatik
- Julelys tænder ved solnedgang og slukker kl. 23
- Julelys tænder kl. 6.30 og slukker ved solopgang
- Bruger tænd/sluk relæ [Tænd/sluk relæ m. ESP8266-12](ESPHome/README.md#4-tændsluk-relæ-m-esp8266-12)
- YAML kode
```YAML
alias: Outdoor light on
description: Tænder udendørs lys ved skumring
trigger:
  - platform: sun
    event: sunset
    offset: "+0:00:00"
condition: []
action:
  - type: turn_on
    device_id: 4bc5cef82ee22c1271d1d25126d5f2e6
    entity_id: switch.light_relay_1
    domain: switch
  - service: notify.notifier_agurk
    data:
      title: message from the dark side
      message: esp8266-12 lys tændt
  - wait_for_trigger:
      - platform: time
        at: "23:00:00"
  - type: turn_off
    device_id: 4bc5cef82ee22c1271d1d25126d5f2e6
    entity_id: switch.light_relay_1
    domain: switch
  - wait_for_trigger:
      - platform: time
        at: "06:30:00"
  - type: turn_on
    device_id: 4bc5cef82ee22c1271d1d25126d5f2e6
    entity_id: switch.light_relay_1
    domain: switch
  - wait_for_trigger:
      - platform: sun
        event: sunrise
        offset: "+0:00:00"
  - type: turn_off
    device_id: 4bc5cef82ee22c1271d1d25126d5f2e6
    entity_id: switch.light_relay_1
    domain: switch
mode: single
```