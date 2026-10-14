# Configuration

Everything you set before flashing. The short version is the
[Install table in the README](../README.md#install).

## ESPHome

Tested with ESPHome **2026.9.1** (pinned in `requirements.txt`), Python 3.11+.
Keep the virtual environment outside the project folder, and outside any
cloud-synced folder:

```bash
python3 -m venv ~/.venvs/esphome && source ~/.venvs/esphome/bin/activate
pip install -r requirements.txt
```

The first build needs internet access: ESPHome downloads the Google Fonts
(`gfonts://`) and the build libraries. To build offline, replace the
`gfonts://` fonts with local font files.

## Secrets

```bash
cp secrets.example.yaml secrets.yaml
```

```yaml
wifi_ssid: "YOUR_WIFI_SSID"
wifi_password: "YOUR_WIFI_PASSWORD"

ap_password: "YOUR_FALLBACK_AP_PASSWORD"   # "M5Stack Dial Fallback" hotspot when Wi-Fi fails

dial_api_key: "YOUR_API_ENCRYPTION_KEY"   # also encrypts OTA uploads
```

The names fit Home Assistant's ESPHome `secrets.yaml`, which all devices
share: `wifi_*` and `ap_password` are usually there already, `dial_*` are
this device's. (Before v1.6.0 they were `api_encryption_key`,
`ota_password` and `ap_ssid`; rename `api_encryption_key` when updating,
the other two are no longer used.)

`secrets.yaml` (and `secrets.*.yaml`) are ignored by Git. The API key in the
example is a public dummy so `esphome config` runs; generate your own before
flashing a real device (e.g. `openssl rand -base64 32`).

### Over-the-air updates

There is no OTA password. Uploads are encrypted with `dial_api_key`, the
same key Home Assistant uses, and the Dial refuses unencrypted ones
(`ota: encryption: {}` in `dial.yaml`). Whoever builds from the same
`secrets.yaml` (ESPHome app or `esphome run`) uploads without extra setup.

Coming from firmware **before v1.6.0** (ESPHome 2026.3), the Dial can't do
encrypted uploads yet, so the first update won't go over the air. Either
flash it once over USB, or build v1.6.0 once with
`password: !secret dial_ota_password` instead of `encryption: {}` under
`ota:` and switch back for the next update. Firmware built with ESPHome
2026.9 or later, password or not, already accepts encrypted uploads with
the API key, so the switch then needs nothing special.

## Home Assistant entities

Edit `src/main/entities.yaml`. Find the IDs under *Developer Tools → States*.

```yaml
substitutions:
  outdoor_temperature_entity: sensor.your_outdoor_temperature
  outdoor_humidity_entity: sensor.your_outdoor_humidity
  outdoor_pressure_entity: sensor.your_outdoor_pressure
  weather_entity: weather.your_location        # only wind speed is used
  aqi_entity: sensor.your_air_quality_index
  climate_entity: climate.your_ac
  music_player_entity: media_player.your_player
  light_kitchen_entity: light.your_light_1
  light_bedroom_strip_entity: light.your_light_2
  light_bedroom_bulb_entity: light.your_light_3
  light_living_entity: light.your_light_4
  light_test_strip_entity: light.your_light_5
  switch_test_entity: switch.your_switch
  radiator_kitchen_entity: climate.your_kitchen_radiator
  radiator_bedroom_entity: climate.your_bedroom_radiator
  radiator_arthur_entity: climate.your_arthur_radiator
  radiator_living_entity: climate.your_living_room_radiator
  radiator_bathroom_entity: climate.your_bathroom_radiator
  window_kitchen_entity: binary_sensor.your_kitchen_window_contact
  window_bedroom_entity: binary_sensor.your_bedroom_window_contact
  window_arthur_entity: binary_sensor.your_arthur_window_contact
  window_living_entity: binary_sensor.your_living_room_window_contact
  motion_alarm_entity: binary_sensor.your_motion_sensor
  motion_alarm_message: "Check Bedroom"         # text on the red alarm screen
```

- **Lights and radiators** are fixed at five lights + one switch and five
  rooms, with names written into the page files. Fewer devices, other names
  or another count: see [pages.md → Lights](pages.md#lights) and
  [pages.md → Radiators](pages.md#radiators).
- **Outdoor values** are plain sensor states. Without own sensors, a template
  sensor on your weather entity works (`{{ state_attr('weather.home', 'temperature') }}`).
- **AQI** is read from a sensor's state, not a weather attribute. The
  [WAQI](https://www.home-assistant.io/integrations/waqi/) integration is a
  simple source.
- **AC modes**: the AC page uses `cool`/`off` and `auto`/`low`/`medium`/`high`.
  If your entity's `hvac_modes` / `fan_modes` differ, see
  [pages.md → AC](pages.md#ac).

After adopting the Dial in Home Assistant (*Settings → Devices & services →
ESPHome*), open its options and turn on **Allow the device to perform Home
Assistant actions**. Without it, every button that changes something in
Home Assistant does nothing.

The Dial also brings its own entities into Home Assistant: among others the
five **Menu** switches (*Menu Timer*, *Menu Lights*, …) that hide pages from
the main menu ([pages.md → Menu items](pages.md#menu-items)).

**Season mode** (`season_mode_entity`): an `input_select` with the options
`Summer` and `Winter`. The Dial shows the AC in summer and the radiators in
winter, unless the switch *Menu Follows Season* is off. Without such a helper, create one under *Settings → Devices &
services → Helpers → Dropdown* with exactly those two options.

## Music player

Choose the entity that actually plays audio, not the Dial itself. A good
target exposes:

- `state`: `playing` or `paused`
- `volume_level`
- `media_title`, `media_artist`
- `media_duration`, `media_position`
- `entity_picture`

If the entity is `unavailable`, Home Assistant disables its controls and the
Dial can't sync it. The page itself talks to SendSpin; see
[pages.md → Music](pages.md#music).

## Time zone

The clock's time zone is set in `src/pages/main.yaml`, **twice**: on the
`pcf8563` time (`rtctime`) and on the `homeassistant` time (`esptime`). It is
`Europe/Berlin` (since v1.4.1; upstream used `Asia/Shanghai`). Elsewhere,
change both to your own [tz database name](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones),
e.g. `Europe/London`; if only one is changed, the clock jumps after every sync.

## Flashing

```bash
esphome config dial.yaml                       # check
esphome run dial.yaml --device /dev/ttyACM0    # first time, over USB (COM3 etc. on Windows)
esphome run dial.yaml                          # later, over Wi-Fi
```

`esphome run` compiles and uploads. `esphome upload` alone reflashes the last
build and ignores edits made since.

Linux or macOS is easiest. On Windows, a full compile only works from
PowerShell (ESP-IDF refuses Git Bash/MSYS) and from a short path such as
`C:\sd`, because ESP-IDF builds fail on long paths.

## Home Assistant's ESPHome app

To build and flash from the ESPHome app (*Settings → Apps → ESPHome Device
Builder*) instead of a computer, the project lives in a subfolder of its
config folder, with a small top-level file the app can see:

```text
/homeassistant/esphome/
├── m5stack-dialv1.yaml       the file the app shows (below)
├── secrets.yaml              shared; add dial_api_key
└── smart-home-dial/          dial.yaml and src/ from this repository
```

```yaml
# m5stack-dialv1.yaml
substitutions:
  dial_dir: ./smart-home-dial   # where images and fonts are, from here

packages:
  dial: !include smart-home-dial/dial.yaml
```

`!include` paths are relative to the including file, but image and font
paths are relative to the top-level file. That's why every local path in
the project starts with `${dial_dir}/`, which is `.` when `dial.yaml` itself
is the main file. Use the ESPHome version of the app (`requirements.txt`
follows it).

## Files you may want to change

| File | For |
|---|---|
| `secrets.yaml` | Wi-Fi and ESPHome credentials |
| `src/main/entities.yaml` | Home Assistant entity IDs |
| `src/main/globals.yaml` | Default timeouts, buzzer, Motion Alert, dial sensitivity, scroll speed ([table](pages.md#tunable-defaults)) |
| `src/pages/main.yaml` | Time zone, clock and weather layout |
| `src/pages/air.yaml` | AC mode strings |
| `src/pages/ha_lights.yaml`, `src/pages/menu.yaml` | Lights: entities and display names |
| `src/pages/radiators.yaml`, `src/pages/menu.yaml` | Radiators: rooms, entities, window sensors |
| `src/pages/music.yaml` | Music page layout |
| `src/pages/settings.yaml` | Ranges of the Settings entries, Motion Alert color |
