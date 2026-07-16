<div align="center">

<img src="src/assets/images/logo.png" width="96" alt="M5Stack Dial Smart Button logo" />

# Smart Home Button

### A Circular HMI for Home Assistant, built with M5Stack Dial, ESPHome, and LVGL.

[![ESPHome](https://img.shields.io/badge/ESPHome-tested%202026.3.3-blue?style=flat-square&logo=esphome)](https://esphome.io/)
[![Platform](https://img.shields.io/badge/Platform-ESP32--S3-red?style=flat-square&logo=espressif)](https://www.espressif.com/)
[![Display](https://img.shields.io/badge/Display-GC9A01A%20240x240-purple?style=flat-square)](#hardware)
[![Version](https://img.shields.io/badge/Version-1.4.0-orange?style=flat-square)](CHANGELOG.md)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](#license)

</div>

## What is this?

This is my M5Stack Dial firmware for controlling a Home Assistant setup from a small round desktop device.

I built it because some smart home actions feel better with a real knob and a small screen. Turning a dial to change brightness, tapping a button for playback, or checking the room status at a glance is faster than opening a phone dashboard every time.

The project is based on ESPHome and LVGL. Most of the UI is split into separate page files, so it is easier to change one feature without touching the rest of the firmware.

## What it can do

- Show time, date, and weather data from Home Assistant.
- Navigate with the rotary encoder, touch gestures, and the front button. From the home screen, swiping left or rotating past one detent opens the main menu.
- Control the built-in LED ring with brightness, color, and preset color flows.
- Control multiple Home Assistant lights/switches from a scrollable Lights submenu (on/off and brightness).
- Control per-room radiators (climate entities) from a Radiators submenu, with a live open/closed window indicator per room where a contact sensor is configured.
- Adjust an air conditioner entity from Home Assistant, including target temperature, power, and fan speed (auto/low/medium/high).
- Show a music page with playback buttons, volume, progress, and album art.
- Run a countdown timer with a rotary-adjustable duration and an audible alarm at zero (cancelable by touch or encoder rotation, with a 60s auto-stop).
- Toggle the buzzer on/off from Settings if you'd rather have a silent timer.
- Sound a buzzer alarm and flash the display red with a custom message when a Home Assistant motion sensor triggers, with configurable duration and cooldown, all adjustable from Settings.
- Adjust display sleep and idle return-to-home timeouts from a Settings submenu; the screen dims automatically and the first touch/turn after sleep only wakes it.
- Tune how many encoder clicks it takes to register one step, shared across every adjustable page and menu, so the dial feels the same everywhere.
- Return to exactly where you left off when backing out of a submenu — only opening fresh from the home screen resets a menu to its default item.
- Keep the device alive on battery power on M5Dial V1.1 by enabling the power-hold pin.

## Gallery

| | | | |
| --- | --- | --- | --- |
| <img src="docs/images/gallery/music-page.jpg" alt="Music page" width="240"> | <img src="docs/images/gallery/clock-weather-page.jpg" alt="Clock and weather page" width="240"> | <img src="docs/images/gallery/ac-page.jpg" alt="AC page" width="240"> | <img src="docs/images/gallery/timer-page.jpg" alt="Timer page" width="240"> |
| <img src="docs/images/gallery/ac-power-page.jpg" alt="AC power control page" width="240"> | <img src="docs/images/gallery/light-page.jpg" alt="Light page" width="240"> | <img src="docs/images/gallery/fridge-page.jpg" alt="Fridge page" width="240"> | <img src="docs/images/gallery/menu-page.jpg" alt="Menu page" width="240"> |

## Pages

| Page | What it does |
| --- | --- |
| Clock | Time, date, weather, humidity, AQI, pressure, and wind speed |
| Menu | Circular page navigation for the small round screen |
| Lights | Submenu listing the onboard LED ring ("Dial Ring") plus your Home Assistant lights/switches; each opens its own simple on/off + brightness page (the LED ring keeps its full color/effects page) |
| Radiators | Submenu for five rooms (Kitchen, Bedroom, Arthur, Living Room, Bathroom), each with its own climate control page (target temperature, power) and a window open/closed icon where a contact sensor is configured |
| AC | Target temperature, power, and fan speed (auto/low/medium/high) control; swing is a local UI toggle only |
| Music | SendSpin media state, cover art, progress, volume, and transport controls |
| Timer | Countdown timer with rotary adjustment and an audible alarm at zero |
| Settings | Submenu for Display Timeout, Menu Timeout, Buzzer on/off, Motion Alert (on/off, duration, cooldown), and Dial Sensitivity |

The Fridge page (small food freshness/status cards) still exists in `src/pages/fridge.yaml` but is currently disabled/commented out of `dial.yaml`; see "Project structure" below.

## Hardware

The firmware is written for M5Stack Dial V1.1.

Main parts used by the project:

- ESP32-S3 controller
- 240 x 240 GC9A01A round display
- FT5x06 capacitive touch
- PCF8563 RTC
- RC522 NFC module on I2C
- Rotary encoder and front button
- SK6812 RGB LED ring
- Buzzer and display backlight
- USB-C for flashing and power

## Project structure

```text
m5dial-smart-button/
|-- dial.yaml                    # ESPHome entry point
|-- CHANGELOG.md                 # Version history
|-- secrets.example.yaml         # Copy this to secrets.yaml
|-- requirements.txt             # ESPHome version used for this project
|-- THIRD_PARTY_NOTICES.md        # Third-party code/font/icon notes
|-- src/
|   |-- main/
|   |   |-- hardware.yaml        # M5Dial pins, display, touch, RTC, power hold, buzzer/rtttl
|   |   |-- entities.yaml        # Home Assistant entity IDs
|   |   `-- globals.yaml         # User-tunable values: scroll speed, timeouts, buzzer, Motion Alert, dial sensitivity
|   |-- pages/                   # One LVGL page per feature
|   |   |-- ha_lights.yaml       # Lights submenu + per-light on/off/brightness page
|   |   |-- radiators.yaml       # Radiators submenu + per-room climate control page
|   |   |-- fridge.yaml          # Fridge page (currently disabled, not included from dial.yaml)
|   |   `-- settings.yaml        # Settings submenu: timeouts, buzzer, Motion Alert, Dial Sensitivity, backlight sleep/wake
|   `-- assets/                  # Fonts and small image assets
|-- components/                  # Local ESPHome components, including SendSpin
|-- hardware/                    # 3D-printable enclosure files
`-- docs/                        # Setup notes and release checklist
```

## Setup

### 1. Install ESPHome

I tested this project with ESPHome `2026.3.3`. The first build may need internet access because ESPHome downloads Google Fonts and build libraries.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 2. Create your secrets file

```bash
cp secrets.example.yaml secrets.yaml
```

Then edit `secrets.yaml`:

```yaml
wifi_ssid: "YOUR_WIFI_SSID"
wifi_password: "YOUR_WIFI_PASSWORD"

ap_ssid: "Dial Fallback Hotspot"
ap_password: "YOUR_FALLBACK_AP_PASSWORD"

api_encryption_key: "YOUR_API_ENCRYPTION_KEY"
ota_password: "YOUR_OTA_PASSWORD"
```

`secrets.yaml` is ignored by Git. Do not publish it.

The API key in `secrets.example.yaml` is only a public dummy value so `esphome config` can run after copying the file. Generate your own key before flashing a real device.

### 3. Set your Home Assistant entities

Edit `src/main/entities.yaml`:

```yaml
substitutions:
  weather_entity: weather.your_location
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
```

You can find these entity IDs in Home Assistant under **Developer Tools -> States**.

The Radiators submenu is fixed at five rooms (Kitchen, Bedroom, Arthur, Living Room, Bathroom). The window indicator is optional per room — Bathroom intentionally has no `window_*_entity` substitution and never shows the indicator; if a room has no contact sensor, point it at any `binary_sensor` you don't mind being ignored, or see "Notes about Radiators" below for how to remove a room's indicator entirely. Room names are hardcoded in `src/pages/radiators.yaml` and `src/pages/menu.yaml` — edit the `NAMES`/`TITLES` arrays there to match your own rooms.

The Lights submenu is fixed at five `light.*` entities plus one `switch.*` entity (on/off only, no brightness). If you have fewer real lights than that, point the unused substitutions at any entity you don't mind seeing greyed out, or see "Notes about the Lights submenu" below for how to change the count. The display names shown on the Dial ("Kitchen", "Bed Underglow", etc.) are hardcoded in `src/pages/ha_lights.yaml` and `src/pages/menu.yaml` — edit the `TITLES`/`NAMES` arrays there to match your own naming.

Most weather integrations do not expose an `aqi` attribute, so AQI is read from its own sensor entity instead (its state, not an attribute). The [World Air Quality Index (WAQI)](https://www.home-assistant.io/integrations/waqi/) integration is a simple way to get one.

For the music page, choose the media player that actually plays audio. It should report useful attributes such as `volume_level`, `media_title`, `media_duration`, and `media_position`. If the entity is `unavailable`, the Dial cannot sync it.

### 4. Check, build, and flash

```bash
esphome config dial.yaml
esphome compile dial.yaml
esphome upload dial.yaml --device /dev/cu.usbmodemXXXX
```

For later OTA updates:

```bash
esphome upload dial.yaml
```

## Notes about the music page

The music page uses the local `sendspin` component for media state, transport commands, and album art. The Home Assistant media player entity is also read for status, volume, duration, and progress when those attributes are available.

Album art is intentionally kept small because the M5Dial does not have PSRAM. If you increase the image size or LVGL buffer too much, the device may reboot when artwork is received.

If volume control does not work, first check the Home Assistant media player entity. Some players expose playback state but do not expose writable volume control. If you want a HA-only music page, replace the SendSpin command scripts with standard `media_player` services.

## Notes about AC controls

Climate entities are not all the same. Temperature, power, and fan speed are wired to Home Assistant services (`climate.set_temperature`, `climate.set_hvac_mode`, `climate.set_fan_mode`) in this project. Swing is a local UI toggle only and is not sent to Home Assistant.

The power button toggles between `hvac_mode: cool` and `hvac_mode: off`, and the fan control cycles `fan_mode` through `auto` / `low` / `medium` / `high`. These strings must match what your specific AC integration actually supports — check `hvac_modes` and `fan_modes` on your climate entity in **Developer Tools -> States**, and edit the mode strings in `src/pages/air.yaml` (the `btn_air_power` on-click handler, and the `air_fan_up`/`air_fan_down` scripts) if your AC uses different values.

The home screen's AC menu tile and the Lights submenu's per-light tiles read their current on/off state directly from Home Assistant, so they stay in sync even if the AC or a light is changed from elsewhere.

## Notes about Radiators

The Radiators submenu (`src/pages/radiators.yaml`) lists five rooms, each with its own page for target temperature (`climate.set_temperature`) and power (`climate.set_hvac_mode`, toggling between `heat` and `off` — edit the `radiator_toggle_power` script if your radiator integration uses different `hvac_mode` strings).

Each room can optionally show a small window icon next to the room name, driven by a `binary_sensor` contact sensor: a plain frame icon (with the cross panes) means closed, a plain frame without the cross means open. A room without a `window_*_entity` substitution (Bathroom, by default) simply never shows the icon. The binary sensors use `trigger_on_initial_state: true` so the icon reflects the sensor's real state immediately after a reboot/reflash, not just after the next state change — worth keeping if you copy this pattern elsewhere, since ESPHome's default behavior only fires `on_state` automations on state *changes*, not on the initial value pushed from Home Assistant on connect.

To change which rooms appear or how many there are, update the `radiator_*_entity`/`window_*_entity` substitutions in `src/main/entities.yaml`, the `binary_sensor`/`text_sensor`/`sensor` blocks and `ENTITY_IDS`/`NAMES` arrays in `src/pages/radiators.yaml`, and the `TITLES`/`ACCENT_COLORS`/`PILL_COLORS` arrays in `src/pages/radiators.yaml` and `src/pages/menu.yaml`. Adding or removing a room means updating the item count in the submenu's carousel code too.

## Notes about the Timer and Buzzer

The Timer page plays an alarm tune through the onboard buzzer (`rtttl:` component) when the countdown reaches zero, repeating until dismissed or until 60 seconds pass (auto-stop). Touching the screen or turning the encoder while the alarm is active cancels it and resets the timer back to its editable state.

The Buzzer entry in Settings is a simple on/off toggle (`buzzer_enabled` in `src/main/globals.yaml`, persisted across reboots). When disabled, the timer alarm is silently skipped instead of playing. Volume control was attempted and dropped — the RTTTL component re-applies its own gain on every note, overriding any manual output-level change — so this is enable/disable only.

## Notes about Motion Alert

Motion Alert (`src/pages/settings.yaml`) sounds the buzzer and flashes the display red with a "Check Bedroom" message whenever `motion_alarm_entity` turns on. It's a single Settings entry with three focusable rows — **Enabled**, **Duration**, **Cooldown** — switch between them by tapping the front button or touching a row directly, then turn the encoder to adjust whichever is focused (toggle on/off, or ±5s).

- **Duration** (default 20s, `motion_alarm_duration_s`) controls how long the buzzer and red flash run once triggered.
- **Cooldown** (default 10s, `motion_alarm_cooldown_s`) is the minimum time between two triggers, so continued motion doesn't restart the alarm every time the sensor re-fires.
- The red overlay fades in/out every 500ms rather than snapping on/off — an abrupt full-screen show/hide visibly tore on this display's small LVGL render buffer, and the fade avoids it. Edit the message text or color in the `motion_alarm_overlay` widget under `lvgl: top_layer:`.
- Like the window sensors, this binary_sensor deliberately does **not** set `trigger_on_initial_state: true` — you don't want a surprise alarm just because the sensor happened to already be on when the device rebooted; only new motion events should trigger it.

## Notes about the Lights submenu

The Lights submenu (`src/pages/ha_lights.yaml`) lists the onboard LED ring ("Dial Ring") plus five `light.*` entities and one `switch.*` entity from Home Assistant. Unlike the LED ring, these Home Assistant lights only get a simple on/off + brightness page — no color picker or effects, since not every bulb supports RGB.

To change which entities appear or how many there are, you need to touch three places consistently: the `light_*_entity`/`switch_*_entity` substitutions in `src/main/entities.yaml`, the `ENTITY_IDS` arrays and per-entity `sensor`/`text_sensor` blocks in `src/pages/ha_lights.yaml`, and the `NAMES`/`TITLES` display-name arrays in `src/pages/ha_lights.yaml` and `src/pages/menu.yaml`. Adding or removing an entity means updating the item count (`ITEM_N`) in the submenu's carousel code too.

## Notes about Settings (Display Timeout / Menu Timeout / Buzzer / Motion Alert / Dial Sensitivity)

The Settings submenu (`src/pages/settings.yaml`) has five entries:

- **Display Timeout** turns off the backlight after this many seconds of inactivity. The first touch or encoder turn afterward only wakes the screen back up — it is swallowed and not treated as a normal button press or menu scroll.
- **Menu Timeout** automatically returns to the home/clock screen after this many seconds of inactivity, no matter how deep in the menus you are.
- **Buzzer** toggles all buzzer sounds (timer alarm and Motion Alert) on or off.
- **Motion Alert** — see "Notes about Motion Alert" above.
- **Dial Sensitivity** — see below.

Display Timeout and Menu Timeout are both in seconds and persisted across reboots, and share the same activity tracker, which resets on any touch or encoder input anywhere in the UI. Their default values (30s and 60s) live in `src/main/globals.yaml` as `display_timeout_s`/`menu_timeout_s`; adjust the range/step in the `display_timeout_up`/`down` and `menu_timeout_up`/`down` scripts in `src/pages/settings.yaml` if 5–300 seconds in steps of 5 does not suit you.

## Notes about Dial Sensitivity (encoder debounce)

On this hardware, one physical click of the rotary encoder produces around 4 raw ticks, not 1 — confirmed by temporarily logging each `on_clockwise`/`on_anticlockwise` firing and turning the dial exactly one detent. Left unhandled, that means "one click" changing a value by 3-4 at once.

`encoder_step_threshold` (`src/main/globals.yaml`, default 4, persisted) is the shared fix: pages that adjust a value accumulate raw ticks and only apply one real step once the accumulator reaches this threshold, in either direction. Dial Sensitivity (`src/pages/settings.yaml`) lets you tune that shared threshold from the device itself (range 1–10) — lower is more sensitive (fewer clicks needed per step), higher is less sensitive. If your dial feels like it still skips more than one item/step per click, raise this value.

Every debounced interaction shares the same `encoder_step_threshold` and the same accumulate-then-step pattern, so the feel is consistent instead of each page inventing its own sensitivity:

- AC temperature/fan (`air_encoder_step_accum` in `src/pages/air.yaml`)
- Motion Alert's focused row (`motion_alert_encoder_accum`)
- Display Timeout, Menu Timeout, Dial Sensitivity itself (`display_timeout_encoder_accum`, `menu_timeout_encoder_accum`, `dial_sensitivity_encoder_accum`)
- Scrolling the Main/Lights/Radiators/Settings carousels (`menu_scroll_encoder_accum` in `src/pages/main.yaml` — shared across all four since only one carousel is ever open at a time)

Radiator temperature, light brightness, music volume, and color-picker hue are intentionally **not** debounced — they're continuous-feeling adjustments where reacting to every raw tick is the desired behavior, not a bug.

## Notes about user-tunable globals

`src/main/globals.yaml` collects the small set of values you're most likely to want to change without hunting through page files: the Display/Menu Timeout defaults above, `buzzer_enabled` (persisted on/off state for the Buzzer setting), `motion_alert_enabled`/`motion_alarm_duration_s`/`motion_alarm_cooldown_s` (Motion Alert defaults), `encoder_step_threshold` (shared dial debounce sensitivity), and `scroll_anim_speed` (pixels/second for the rolling/marquee text used on menu tiles whose name is too long to fit — default 20, LVGL's own default is 25). Any label anywhere in the UI can reuse `id(scroll_anim_speed)` the same way rather than hardcoding its own speed.

## Notes about menu navigation

Backing out of a submenu (Settings, Lights, Radiators, or an individual Radiator room) returns you to exactly the item you came from, not the top of the list. Each carousel's "open" script (`open_menu`, `open_settings_menu`, `open_lights_menu`, `open_radiators_menu`) no longer resets the selection — only a fresh entry from its parent does that, via a separate `enter_*` script (`enter_menu`, `enter_settings_menu`, `enter_lights_menu`, `enter_radiators_menu`) that resets the selection first and then opens. If you add a new carousel, follow the same split: reset-and-open only on the "fresh from parent" path, plain open (no reset) on every "back" path.

## Notes about battery power

M5Dial V1.1 needs the hold pin to stay enabled when running from battery. This project turns on GPIO46 during boot in `dial.yaml` and defines the `power_hold` switch in `src/main/hardware.yaml`.

## Common issues

- **Device is unavailable in Home Assistant**: make sure the `api` section is enabled and port `6053` is reachable.
- **Wrong weather values**: change `weather_entity` in `src/main/entities.yaml`.
- **AQI stuck on `--`**: `aqi_entity` in `src/main/entities.yaml` must point to an air-quality sensor's own state, not an attribute on the weather entity.
- **Wrong AC entity**: change `climate_entity` in `src/main/entities.yaml`.
- **AC power/temperature/fan calls do nothing in Home Assistant**: check Settings -> Devices & Services -> ESPHome -> your device for "Allow the device to perform Home Assistant actions", and confirm your `hvac_mode`/`fan_mode` strings actually exist on the entity (see "Notes about AC controls").
- **Uploading with `esphome upload` seems to ignore your latest edits**: run `esphome compile dial.yaml` first — `upload` alone reflashes whatever binary is already built, it does not recompile.
- **Wrong or missing Lights submenu entity**: change the matching `light_*_entity`/`switch_*_entity` substitution in `src/main/entities.yaml` (see "Notes about the Lights submenu").
- **Window indicator stuck on closed after a reflash**: make sure the room's `binary_sensor` config keeps `trigger_on_initial_state: true` (see "Notes about Radiators") — without it, the icon only updates on the sensor's next state change, not on the value already pushed at boot.
- **Buzzer volume**: there is no volume control, only on/off — see "Notes about the Timer and Buzzer".
- **One click moves more than one step/item**: raise Dial Sensitivity in Settings (see "Notes about Dial Sensitivity"). This hardware's encoder produces around 4 raw ticks per click, and the default threshold assumes that; a looser mechanical feel may need a higher value.
- **Motion Alert fires too often / not often enough**: adjust Cooldown and Duration on the Motion Alert page (tap the front button or touch a row to focus it, then turn the encoder).
- **Music page shows unavailable**: choose the real player entity, not the Dial entity itself.
- **Album art causes reboot**: keep the artwork small and do not increase LVGL memory use too much.
- **First build cannot download fonts**: connect to the internet once so ESPHome can fetch Google Fonts, or replace `gfonts://` fonts with local font files.
- **Battery does not keep the device on**: check the V1.1 power-hold configuration.

## What you may want to change

Most people will need to edit only these files:

- `secrets.yaml` for Wi-Fi and ESPHome credentials.
- `src/main/entities.yaml` for Home Assistant entity IDs.
- `src/pages/main.yaml` if you want a different timezone or weather layout.
- `src/pages/music.yaml` if you want to customize the music UI.
- `src/pages/air.yaml` if your AC's `hvac_mode`/`fan_mode` strings differ from `cool`/`off`/`auto`/`low`/`medium`/`high`.
- `src/pages/ha_lights.yaml` and `src/pages/menu.yaml` for the Lights submenu's entities and display names.
- `src/pages/radiators.yaml` and `src/pages/menu.yaml` for the Radiators submenu's rooms, entities, and window sensors.
- `src/main/globals.yaml` for the default Display/Menu Timeout values, the buzzer/Motion Alert defaults, the shared dial debounce sensitivity, and the rolling-text scroll speed.
- `src/pages/settings.yaml` for the min/max/step bounds of the Display Timeout, Menu Timeout, and Dial Sensitivity adjustments.

## License

MIT License for the original project files. See `LICENSE`.

Third-party code, fonts, icons, and build dependencies keep their own licenses. See `THIRD_PARTY_NOTICES.md`.

## Demo Video

Watch the demo on YouTube: [Smart Home Button demo](https://www.youtube.com/watch?v=51bXRBuSLpM)
