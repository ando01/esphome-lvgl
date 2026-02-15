# Guition ESP32-S3 4848S040 Music Dashboard (4.0")

4-inch touch LCD panel with ESP32-S3, running ESPHome and LVGL as a now-playing music dashboard with dynamic media player switching.

![Guition ESP32-S3 4848S040](guition-esp32-s3-4848s040.jpg)

## Features

- Full-screen album art with song title, artist, and playback controls
- **Swipe down** to reveal a media player picker that auto-discovers all `media_player` entities from Home Assistant
- Select a different speaker/device and all controls instantly switch to it
- TV mode detection (hides music UI when source is TV)

## Configuration

### 1. ESPHome Device

Use the contents of [esphome/template.yaml](esphome/template.yaml) as your ESPHome config in the dashboard or CLI. Update the substitutions to match your setup.

### 2. Home Assistant Helpers (required)

The entity picker needs a small set of HA helpers to dynamically discover and proxy media player data. Add the following to your Home Assistant `configuration.yaml` (or via the UI/packages as you prefer):

#### Input Text — stores the currently selected media player

```yaml
input_text:
  esphome_active_player:
    name: "ESPHome Active Media Player"
    initial: "media_player.office"
    max: 255
```

> Change `initial` to your default media player entity ID.

#### Template Sensor — auto-discovers all media players

```yaml
template:
  - sensor:
      - name: "Media Players List"
        state: "{{ states.media_player | list | length }}"
        attributes:
          entity_ids: "{{ states.media_player | map(attribute='entity_id') | join('\n') }}"
          friendly_names: "{{ states.media_player | map(attribute='name') | join('\n') }}"
```

#### Trigger-based Template Sensors — follow the active player's attributes

These sensors poll every 2 seconds and also react immediately when you switch the active player.

```yaml
template:
  - trigger:
      - platform: state
        entity_id: input_text.esphome_active_player
      - platform: time_pattern
        seconds: "/2"
    sensor:
      - name: "Active Media Title"
        state: "{{ state_attr(states('input_text.esphome_active_player'), 'media_title') | default('', true) }}"
      - name: "Active Media Artist"
        state: "{{ state_attr(states('input_text.esphome_active_player'), 'media_artist') | default('', true) }}"
      - name: "Active Entity Picture"
        state: "{{ state_attr(states('input_text.esphome_active_player'), 'entity_picture') | default('', true) }}"
      - name: "Active Media State"
        state: "{{ states(states('input_text.esphome_active_player')) }}"
      - name: "Active Media Source"
        state: "{{ state_attr(states('input_text.esphome_active_player'), 'source') | default('', true) }}"
      - name: "Active Media Duration"
        state: "{{ state_attr(states('input_text.esphome_active_player'), 'media_duration') | default(0) }}"
        unit_of_measurement: "s"
      - name: "Active Media Position"
        state: "{{ state_attr(states('input_text.esphome_active_player'), 'media_position') | default(0) }}"
        unit_of_measurement: "s"
```

After adding these, restart Home Assistant and verify the sensors appear under **Developer Tools > States**.

## Where to Buy

- **Panel**: [AliExpress](https://s.click.aliexpress.com/e/_c3sIhvBv) (~£16)

## Stand

- **Desktop stand** (3D printable): [MakerWorld](https://makerworld.com/en/models/2327976-touch-screen-desktop-stand-for-guition-4848s040#profileId-2543111)

## Folder Structure

```
guition-esp32-s3-4848s040-music/
├── addon/          # Time, network, backlight, music (album art), entity picker
├── assets/         # Fonts and icons
├── device/         # device.yaml, sensors.yaml, lvgl.yaml
├── esphome/        # template.yaml
├── theme/          # Button and UI styling
└── README.md
```

Customize for your setup by editing the YAML files under `device/`, `addon/`, and `theme/`. See the [main README](../README.md) for full quick start and ESPHome setup.
