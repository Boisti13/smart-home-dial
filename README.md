<div align="center">

<img src="src/assets/images/logo.png" width="96" alt="Smart Home Dial logo" />

# Smart Home Dial

[![Version](https://img.shields.io/badge/version-1.8.0-orange)](CHANGELOG.md)
[![ESPHome](https://img.shields.io/badge/ESPHome-tested%202026.9.1-blue?logo=esphome)](https://esphome.io/)
[![Platform](https://img.shields.io/badge/platform-ESP32--S3-red?logo=espressif)](#hardware)
[![Display](https://img.shields.io/badge/display-GC9A01A%20240x240-purple)](#hardware)
[![License: MIT](https://img.shields.io/github/license/Boisti13/smart-home-dial)](LICENSE)

</div>

A small round knob for your desk that controls Home Assistant. Turn it to
dim a light or set a radiator, tap it to start or stop the music, glance at
it for the time and the weather. It's ESPHome firmware for the
**M5Stack Dial**, with an LVGL interface built for the 240 × 240 round
screen. It also runs a kitchen timer and can sound an alarm when a motion
sensor goes off.

Based on [Jasionf/smart-home-button](https://github.com/Jasionf/smart-home-button),
with radiators, Home Assistant lights, settings, Motion Alert and a
reworked dial feel added on top.

<img src="docs/images/gallery/menu-page.jpg" alt="The main menu on the Dial" width="320">

## Highlights

- **Clock and weather** at a glance: time, date, outdoor temperature, humidity and pressure from your own sensors, air quality and wind
- **Lights**: the Dial's own LED ring (brightness, colors, effects) plus five Home Assistant lights and a switch, each with on/off and brightness
- **Radiators**: five rooms with target temperature and on/off, and a **window open/closed icon** per room from a contact sensor
- **Air conditioner**: target temperature, power and fan speed (auto/low/medium/high)
- **Music**: cover art, progress, volume and play/pause/skip through SendSpin
- **Timer** with an alarm tune on the buzzer: stop it with a touch or a turn, and it stops by itself after 60 s
- **Motion Alert**: buzzer and a red flashing screen with your own message when a Home Assistant motion sensor triggers, with adjustable duration and cooldown; a touch, turn or press silences it
- **Settings on the device**: screen timeout, back-to-home timeout, buzzer on/off, Motion Alert, **dial sensitivity**, and **which pages the main menu shows**. Everything is saved across reboots
- **Hide what you don't use**: switch Timer, Lights, Radiators, AC or Music off in the main menu, on the Dial or from Home Assistant
- **Follows the season** (can be turned off): a Summer/Winter select in Home Assistant shows the AC in summer and the radiators in winter
- **Feels the same everywhere**: one click is one step on every page and menu, and backing out of a submenu returns you to where you were
- **Three ways to navigate**: turn the knob, touch, or press the front button. From the home screen a swipe or a turn opens the menu
- **Battery-friendly**: the screen sleeps by itself, and the first touch only wakes it
- **3D-printable case** for desk or fridge door ([hardware/3d-models](hardware/3d-models))

| | | | |
|---|---|---|---|
| ![Clock and weather](docs/images/gallery/clock-weather-page.jpg) | ![Music](docs/images/gallery/music-page.jpg) | ![Air conditioner](docs/images/gallery/ac-page.jpg) | ![Timer](docs/images/gallery/timer-page.jpg) |
| ![AC power](docs/images/gallery/ac-power-page.jpg) | ![LED ring](docs/images/gallery/light-page.jpg) | ![Fridge (disabled)](docs/images/gallery/fridge-page.jpg) | ![Menu](docs/images/gallery/menu-page.jpg) |

Demo video: [Smart Home Button on YouTube](https://www.youtube.com/watch?v=51bXRBuSLpM) (the upstream project).

## Pages

| Page | What it does |
|---|---|
| **Clock** | Time, date, weather, humidity, AQI, pressure, wind; home screen |
| **Menu** | Round carousel of the pages below; each of Timer, Lights, Radiators, AC and Music can be hidden |
| **Lights** | LED ring ("Dial Ring") with colors and effects, plus 5 HA lights and 1 switch with on/off and brightness |
| **Radiators** | Kitchen, Bedroom, Arthur, Living Room, Bathroom: target temperature, on/off, window icon |
| **AC** | Target temperature, power, fan speed; swing is a local toggle only |
| **Music** | SendSpin player: cover art, progress, volume, transport buttons |
| **Timer** | Countdown set with the knob, alarm at zero |
| **Settings** | Display Timeout, Menu Timeout, Buzzer, Motion Alert, Dial Sensitivity, Menu Items |

The Fridge page still exists in `src/pages/fridge.yaml` but is switched off
in `dial.yaml`. How each page works and how to change it:
**[docs/pages.md](docs/pages.md)**.

## Hardware

Written for the **M5Stack Dial V1.1**.

| Part | On the Dial |
|---|---|
| Controller | ESP32-S3 (StampS3), no PSRAM |
| Display | 240 × 240 round GC9A01A, backlight dimmable |
| Touch | FT5x06 capacitive |
| Input | Rotary encoder (about 4 raw ticks per click) and front button |
| Clock | PCF8563 RTC, set from Home Assistant |
| Light | SK6812 RGB LED ring |
| Sound | Buzzer (RTTTL tunes) |
| NFC | RC522 on I2C: scanned cards and tags go to Home Assistant as tag events |
| Other | USB-C for flashing and power, battery hold pin GPIO46 |

## Install

| Step | What to do |
|---|---|
| 1. ESPHome | Python 3.11+, then `pip install -r requirements.txt` (ESPHome 2026.9.1) in a virtual environment |
| 2. Secrets | `cp secrets.example.yaml secrets.yaml`, fill in Wi-Fi, fallback hotspot, API key, OTA password |
| 3. Entities | Put your Home Assistant entity IDs into `src/main/entities.yaml` ([table below](#home-assistant-entities)) |
| 4. Check | `esphome config dial.yaml` |
| 5. Flash | first time over USB: `esphome run dial.yaml --device <port>`; later over Wi-Fi: `esphome run dial.yaml` |
| 6. Home Assistant | Optional: build and flash from HA's ESPHome app instead ([how](docs/configuration.md#home-assistants-esphome-app)). Adopt the device under *Settings → Devices & services → ESPHome*, and turn on **Allow the device to perform Home Assistant actions** |

```bash
python3 -m venv ~/.venvs/esphome && source ~/.venvs/esphome/bin/activate
pip install -r requirements.txt
cp secrets.example.yaml secrets.yaml      # then edit it
esphome config dial.yaml
esphome run dial.yaml --device /dev/ttyACM0
```

The first build needs internet: ESPHome downloads Google Fonts and the
build libraries. `esphome run` compiles and uploads in one step. A plain
`esphome upload` only reflashes the last build. The API key in
`secrets.example.yaml` is a public dummy; generate your own before flashing.
More detail: **[docs/configuration.md](docs/configuration.md)**.

## Home Assistant entities

All in `src/main/entities.yaml`; find the IDs under *Developer Tools → States*.

| Substitution | Entity | Used by |
|---|---|---|
| `outdoor_temperature_entity`, `outdoor_humidity_entity`, `outdoor_pressure_entity` | `sensor.*` (e.g. the *Virtual Outdoor* template sensors) | Clock |
| `weather_entity` | `weather.*` (only `wind_speed` is used) | Clock |
| `aqi_entity` | `sensor.*` air quality index (its state), e.g. from [WAQI](https://www.home-assistant.io/integrations/waqi/) | Clock |
| `climate_entity` | `climate.*` air conditioner | AC page, menu tile |
| `music_player_entity` | `media_player.*` that actually plays audio | Music |
| `light_kitchen_entity`, `light_bedroom_strip_entity`, `light_bedroom_bulb_entity`, `light_living_entity`, `light_test_strip_entity` | `light.*` × 5 | Lights |
| `switch_test_entity` | `switch.*` | Lights (on/off only) |
| `radiator_kitchen_entity` … `radiator_bathroom_entity` | `climate.*` × 5 | Radiators |
| `window_kitchen_entity`, `window_bedroom_entity`, `window_arthur_entity`, `window_living_entity` | `binary_sensor.*` door/window contact | Radiators (Bathroom has none) |
| `motion_alarm_entity` | `binary_sensor.*` motion | Motion Alert |
| `motion_alarm_message` | text, e.g. `"Check Bedroom"` | Motion Alert screen |
| `season_mode_entity` | `input_select.*` with the options `Summer` and `Winter` | Main menu: AC in summer, Radiators in winter |

Display names ("Kitchen", "Bed Underglow" …) and the number of lights and
rooms are fixed in the page files. See
[docs/pages.md](docs/pages.md#lights) for how to change them.

## Example: a motion alarm for the bedroom

You want the Dial in the living room to beep and flash when the camera in
the bedroom sees motion, but not more than once every half minute.

1. In `src/main/entities.yaml` point Motion Alert at the camera's motion
   sensor and set the message:
   ```yaml
   motion_alarm_entity: binary_sensor.cam_pt2_motion_alarm
   motion_alarm_message: "Check Bedroom"
   ```
2. Flash: `esphome run dial.yaml`.
3. On the Dial: turn to **Settings → Motion Alert**. The page has three rows.
   Tap a row (or press the front button to move between them) and turn the
   knob:
   - **Enabled** → on
   - **Duration** → 20 s (how long it beeps and flashes)
   - **Cooldown** → 30 s (in steps of 5 s)
4. When the sensor turns on, the screen fades red with *Check Bedroom* and the
   buzzer sounds for 20 s, then stops by itself. Touch the screen, turn the
   knob or press the button to silence it sooner; that touch or turn does
   nothing else. The values are kept across reboots. A sensor that is already on when the Dial boots does
   not set it off.

To change the color, edit the `motion_alarm_overlay` widget in
`src/pages/settings.yaml`.

## Documentation

| | |
|---|---|
| [docs/configuration.md](docs/configuration.md) | Secrets, entities, music player, time zone, which files to edit |
| [docs/pages.md](docs/pages.md) | Every page in detail: AC modes, radiators, lights, timer and buzzer, Motion Alert, settings, dial sensitivity, menu navigation |
| [docs/troubleshooting.md](docs/troubleshooting.md) | Common problems and their fixes |
| [docs/development.md](docs/development.md) | Layout, naming, validating, versions and releases |
| [hardware/3d-models](hardware/3d-models) | The printable case |
| [CHANGELOG.md](CHANGELOG.md) | What changed in each version |

## Layout and development

```text
smart-home-dial/
├── dial.yaml               ESPHome entry point: includes every package below, boot, API, Wi-Fi
├── secrets.example.yaml    copy to secrets.yaml (ignored by Git)
├── requirements.txt        pinned ESPHome version
├── src/
│   ├── main/
│   │   ├── hardware.yaml   pins, display, touch, RTC, encoder, LED ring, buzzer, power hold
│   │   ├── entities.yaml   Home Assistant entity IDs
│   │   └── globals.yaml    defaults you may want to tune (timeouts, buzzer, Motion Alert, dial sensitivity, scroll speed)
│   ├── pages/              one LVGL page per feature (main = clock, menu, light, ha_lights, radiators, air, music, timer, settings, load, fridge)
│   └── assets/             fonts and small icons (no PSRAM: keep them small)
├── hardware/3d-models/     STL files for the case
└── docs/                   the guides above and gallery images
```

Work happens on `dev`; `main` holds releases. Before every commit run
`esphome config dial.yaml` (with a `secrets.yaml`, the example works), and
`esphome compile dial.yaml` for anything beyond text changes. Each change
updates `CHANGELOG.md`, this README if users notice it, and the version in
`dial.yaml` and the badge. The full routine is in
[docs/development.md](docs/development.md).

## License

MIT for the original project files, see [LICENSE](LICENSE). Third-party
code, fonts, icons and build dependencies keep their own licenses, see
[THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
