switch:
  - platform: command_line
    switches:
      aquariumledmanual:
       friendly_name: Handmatige LED bediening
       command_off: 'curl -X POST -k -H "Content-Type: application/x-www-form-urlencoded" -d "action=14cswi=false" http://10.0.76.2/stat'

shell_command:
  aquariumled: 'curl -X POST -k -H "Content-Type: application/x-www-form-urlencoded" -d "action=3&ch1={{ (states("input_number.aquarium_slider_wit") | round(0)) }}&ch2={{ (states("input_number.aquarium_slider_blauw") | round(0)) }}&ch3={{ (states("input_number.aquarium_slider_groen") | round(0)) }}&ch4={{ (states("input_number.aquarium_slider_rood") | round(0)) }}" http://10.0.76.2/stat'
  aquariumledstate: 'curl -X POST -k -H "Content-Type: application/x-www-form-urlencoded" -d "action=14&ch5=843&tswi=false&ttime=01:00&cswi=true&ctime={{ (states("input_datetime.aquarium_led_timer")[:5]) }}" http://10.0.76.2/stat'
 
automation:
- alias: Helialux - control
  trigger:
    - platform: state
      entity_id:
        - input_number.aquarium_slider_wit
        - input_number.aquarium_slider_rood
        - input_number.aquarium_slider_groen
        - input_number.aquarium_slider_blauw
  action:
    service: shell_command.aquariumled
    
- alias: Helialux - manual
  trigger:
    - platform: state
      entity_id:
        - switch.aquariumledmanual
      from: 'off'
      to: 'on'
  action:
    service: shell_command.aquariumledstate

- alias: Helialux - sync
  trigger:
    - platform: state
      entity_id:
        - sensor.helialux_wit
        - sensor.helialux_rood
        - sensor.helialux_groen
        - sensor.helialux_blauw
  action:
    - service: input_number.set_value
      data_template:
        entity_id: input_number.aquarium_slider_wit
        value: '{{ states(''sensor.helialux_wit'') }}'
    - service: input_number.set_value
      data_template:
        entity_id: input_number.aquarium_slider_groen
        value: '{{ states(''sensor.helialux_groen'') }}'
    - service: input_number.set_value
      data_template:
        entity_id: input_number.aquarium_slider_blauw
        value: '{{ states(''sensor.helialux_blauw'') }}'
    - service: input_number.set_value
      data_template:
        entity_id: input_number.aquarium_slider_rood
        value: '{{ states(''sensor.helialux_rood'') }}'
