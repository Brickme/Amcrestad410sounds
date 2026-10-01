# Home Assistant Integration

This guide details how to integrate custom audio playback for the Amcrest AD410 doorbell camera into Home Assistant using shell commands and automations. 

> **Note:** Paths used in examples assume your `.raw` files are placed inside Home Assistant's local web directory (e.g., `/config/www/uploads/welcome_wonderful.raw`). Update entity IDs and paths to match your setup.

## 1. Shell Command Configuration

Add the following to your `configuration.yaml` in Home Assistant:

```yaml
shell_command:
  play_doorbell_sound: >
    curl -u admin:YOUR_PASSWORD "http://DOORBELL_IP/cgi-bin/audio.cgi?action=postAudio&httptype=singlepart" \
      --header "Content-Type: Audio/G.711A" \
      --data-binary "@{{ file_path }}"
```
## 2. Dynamic Audio Automation Example
Example automation that selects different audio files based on seasonal sensors, holiday states, or custom entities:
```
alias: "Doorbell: Play Dynamic Seasonal Sound"
trigger:
  - platform: state
    entity_id: binary_sensor.doorbell_call_button
    to: "on"
action:
  - service: shell_command.play_doorbell_sound
    data:
      file_path: >-
        {% if is_state('sensor.halloween_day', 'True') %}
          /config/www/uploads/Halloween/halloween_day.raw
        {% elif is_state('sensor.halloween_season', 'True') %}
          /config/www/uploads/Halloween/spooky_chime.raw
        {% elif is_state('sensor.independence_day_season', 'True') %}
          /config/www/uploads/Independence_Day/independace_day.raw
        {% elif is_state('sensor.christmas_day', 'True') %}
          /config/www/uploads/Christmas/christmas_day.raw      
        {% elif is_state('sensor.christmas_season', 'True') %}
          /config/www/uploads/Christmas/merry_christmas.raw
        {% elif is_state('sensor.easter_season', 'True') %}
          /config/www/uploads/Easter/easter.raw
        {% elif is_state('binary_sensor.sidewalk_cat_occupancy_2', 'on') %}
          /config/www/uploads/Cat/meow_porch.raw
        {% else %}
          /config/www/uploads/welcome_wonderful.raw
        {% endif %}
```
## 3. Push Notification Automation   
Trigger the doorbell audio and send a high-priority mobile notification with camera snapshot simultaneously:

```
action: notify.my_local_notification_group
data:
  title: Doorbell Audio Plus
  message: Human detected at the front door.
  data:
    entity_id: camera.ad410_sub
    url: /dashboard-porch/notifications
    group: doorbell-alerts
    push:
      sound:
        name: |-
          {% if is_state('sensor.halloween_day', 'True') %}
            halloween_day_uncut.wav
          {% elif is_state('sensor.halloween_season', 'True') %}
            spooky_chime.wav
          {% elif is_state('sensor.independence_day_season', 'True') %}
            independence_day.wav
          {% elif is_state('sensor.christmas_day', 'True') %}
            christmas_day.wav
          {% elif is_state('sensor.christmas_season', 'True') %}
            merry_christmas.wav
          {% elif is_state('sensor.easter_season', 'True') %}
            easter.wav
          {% elif is_state('binary_sensor.sidewalk_cat_occupancy_2', 'on') %}
            meow_porch.wav
          {% else %}
            welcome_wonderful.wav
          {% endif %}
        critical: 1
        volume: 0.8
    actions:
      - action: URI
        title: Open Camera
        uri: /doorbell-intercom/default-
      - action: DISMISS
        title: Dismiss
        destructive: true

```
