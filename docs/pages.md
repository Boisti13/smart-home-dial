# Pages in detail

How each page works, which Home Assistant actions it calls, and where to
change it. Entity IDs are set in `src/main/entities.yaml`; see
[configuration.md](configuration.md).

- [Clock](#clock)
- [AC](#ac)
- [Radiators](#radiators)
- [Lights](#lights)
- [Music](#music)
- [Timer and buzzer](#timer-and-buzzer)
- [Motion Alert](#motion-alert)
- [Settings](#settings)
- [Dial sensitivity](#dial-sensitivity)
- [Menu navigation](#menu-navigation)
- [Tunable defaults](#tunable-defaults)
- [Battery](#battery)
- [NFC](#nfc)
- [Fridge (disabled)](#fridge-disabled)

## Clock

`src/pages/main.yaml`. Time and date come from the PCF8563 RTC, which is set
from Home Assistant whenever the Dial connects. Temperature, humidity and
pressure are the states of three sensors (`outdoor_temperature_entity`,
`outdoor_humidity_entity`, `outdoor_pressure_entity`); in the owner's setup
these are the *Virtual Outdoor* template sensors, which take the coldest of
three outdoor sensors (the sun warms the others) and Netatmo's pressure. Wind
is the `wind_speed` attribute of `weather_entity`. AQI is the **state** of its own
sensor (`aqi_entity`), because most weather integrations have no `aqi`
attribute. The time zone is set twice in this file (`time:` → `rtctime` and
`esptime`), see [configuration.md](configuration.md#time-zone).

## AC

`src/pages/air.yaml`. Target temperature, power and fan speed call
`climate.set_temperature`, `climate.set_hvac_mode` and
`climate.set_fan_mode`. Swing is a local UI toggle only and is not sent to
Home Assistant.

The power button toggles between `hvac_mode: cool` and `off`; the fan
control cycles `fan_mode` through `auto` / `low` / `medium` / `high`. These
strings must match what your integration supports: check `hvac_modes` and
`fan_modes` on the entity under *Developer Tools → States*, and edit the
`btn_air_power` on-click handler and the `air_fan_up` / `air_fan_down`
scripts if yours differ.

The home screen's AC tile shows the live fan speed and room temperature, read
from Home Assistant, so it stays right when the AC is changed elsewhere.

## Radiators

`src/pages/radiators.yaml`. Five rooms (Kitchen, Bedroom, Arthur, Living
Room, Bathroom), each with a page for target temperature
(`climate.set_temperature`) and power (`climate.set_hvac_mode`, `heat` ↔
`off`; edit the `radiator_toggle_power` script for other strings).

Each room can show a **window icon** next to its name, driven by a
`binary_sensor` contact: frame with cross panes = closed, frame without =
open. A room without a `window_*_entity` substitution (Bathroom by default)
never shows it. The window sensors use `trigger_on_initial_state: true`, so
the icon is right straight after a reboot. ESPHome otherwise fires
`on_state` only on *changes*, not on the value Home Assistant sends on
connect. Keep that if you copy the pattern.

To change rooms or their number, update consistently:

- the `radiator_*_entity` / `window_*_entity` substitutions in `src/main/entities.yaml`
- the `binary_sensor` / `text_sensor` / `sensor` blocks and the `ENTITY_IDS` / `NAMES` arrays in `src/pages/radiators.yaml`
- the `TITLES` / `ACCENT_COLORS` / `PILL_COLORS` arrays in `src/pages/radiators.yaml` and `src/pages/menu.yaml`
- the item count in the submenu's carousel code

## Lights

`src/pages/ha_lights.yaml`. The submenu lists the Dial's own LED ring
("Dial Ring", with the full color/effects page in `src/pages/light.yaml`)
plus five `light.*` entities and one `switch.*`. Home Assistant lights get
on/off and brightness only, no color, since not every bulb has RGB; the
switch gets on/off only. Each tile shows the live on/off state from Home
Assistant.

Fewer real lights? Point the spare substitutions at any entity you don't mind
seeing greyed out. To change entities, names or count, update consistently:

- the `light_*_entity` / `switch_*_entity` substitutions in `src/main/entities.yaml`
- the `ENTITY_IDS` arrays and per-entity `sensor` / `text_sensor` blocks in `src/pages/ha_lights.yaml`
- the `NAMES` / `TITLES` arrays in `src/pages/ha_lights.yaml` and `src/pages/menu.yaml`
- the item count (`ITEM_N`) in the submenu's carousel code

## Music

`src/pages/music.yaml`. Media state, transport commands and album art come
from the local `sendspin` component (`components/sendspin/`). The Home
Assistant media player (`music_player_entity`) is also read for status,
volume, duration and progress when it has those attributes.

- Album art is kept small on purpose: the Dial has no PSRAM, and bigger
  images or a bigger LVGL buffer can make it reboot when artwork arrives.
- If volume does nothing, check that the media player actually exposes
  writable volume. Some only report playback state.
- For a Home Assistant-only music page, replace the SendSpin command scripts
  with standard `media_player` actions.

## Timer and buzzer

`src/pages/timer.yaml`. Set the duration with the knob and start it. At zero
the buzzer plays an RTTTL tune (`rtttl:` component), repeating until you
touch the screen or turn the knob, or until 60 s pass. Cancelling also resets
the timer to its editable state. While a timer runs, the screen doesn't sleep
and the Dial doesn't return home.

**Buzzer** in Settings turns all buzzer sounds (timer and Motion Alert) on or
off (`buzzer_enabled`, saved). There is no volume control: the RTTTL
component re-applies its own gain on every note, so `output.set_level()` and
`set_gain()` had no audible effect and were dropped.

## Motion Alert

`src/pages/settings.yaml`. When `motion_alarm_entity` turns on, the buzzer
sounds and the screen flashes red with `motion_alarm_message` (default
*Check Bedroom*, set in `src/main/entities.yaml`). One Settings entry
with three rows: **Enabled**, **Duration**, **Cooldown**. Tap the front button
or touch a row to focus it, then turn the knob (on/off, or ±5 s).

- **Duration** (default 20 s, `motion_alarm_duration_s`): how long the buzzer
  and flash run. The alarm stops by itself after that.
- **Silence it sooner** with a touch, a turn of the knob or a press of the
  button. That input does nothing else: the overlay catches the touch, and
  the turn or press is swallowed like the one that wakes the screen.
- **Cooldown** (default 10 s, `motion_alarm_cooldown_s`): the minimum time
  between two alarms, so continued motion doesn't restart it each time the
  sensor fires again.
- The red overlay **fades** in and out every 500 ms. A hard show/hide tore
  visibly on this display's small LVGL render buffer. The color is in the
  `motion_alarm_overlay` widget under `lvgl: top_layer:`.
- Unlike the window sensors, this `binary_sensor` deliberately does **not**
  set `trigger_on_initial_state`: a sensor that is already on at boot should
  not set off an alarm, only new motion should.

## Settings

`src/pages/settings.yaml`, five entries:

| Entry | What it does | Default | Range |
|---|---|---|---|
| **Display Timeout** | Backlight off after this many seconds without input. The first touch or turn afterwards only wakes the screen. | 30 s | 5–300 s, steps of 5 |
| **Menu Timeout** | Back to the clock after this many seconds without input, however deep you are | 60 s | 5–300 s, steps of 5 |
| **Buzzer** | All buzzer sounds on/off | on | |
| **Motion Alert** | See [above](#motion-alert) | | |
| **Dial Sensitivity** | See [below](#dial-sensitivity) | 4 | 1–10 |

All values are saved across reboots. Both timeouts share one activity
tracker that any touch or turn resets. Defaults are in
`src/main/globals.yaml`; the ranges are in the `*_up` / `*_down` scripts in
`src/pages/settings.yaml`.

## Dial sensitivity

One physical click of this encoder produces about **4 raw ticks**, not 1
(confirmed by logging every `on_clockwise` / `on_anticlockwise` while turning
one detent). Unhandled, one click would change a value by 3–4.

`encoder_step_threshold` (`src/main/globals.yaml`, default 4, saved) is the
shared fix: pages collect raw ticks and take one real step when the count
reaches the threshold, in either direction. **Dial Sensitivity** in Settings
changes it on the device: lower = fewer clicks per step. If a click still
skips more than one item, raise it.

Every debounced interaction uses the same threshold and pattern:

| Interaction | Accumulator |
|---|---|
| AC temperature and fan | `air_encoder_step_accum` (`src/pages/air.yaml`) |
| Motion Alert's focused row | `motion_alert_encoder_accum` |
| Display Timeout, Menu Timeout, Dial Sensitivity | `display_timeout_encoder_accum`, `menu_timeout_encoder_accum`, `dial_sensitivity_encoder_accum` |
| Main, Lights, Radiators and Settings carousels | `menu_scroll_encoder_accum` (`src/pages/main.yaml`; shared, only one carousel is open at a time) |

Radiator temperature, light brightness, music volume and the color picker
are deliberately **not** debounced: they should react to every tick.

## Menu navigation

Backing out of a submenu (Settings, Lights, Radiators, or a radiator room)
returns to the item you came from. Each carousel has two scripts:

- `enter_*` (`enter_menu`, `enter_settings_menu`, `enter_lights_menu`,
  `enter_radiators_menu`): resets the selection, then opens. Used only when
  coming fresh from the parent.
- `open_*` (`open_menu`, `open_settings_menu`, …): opens without resetting.
  Used on every "back" path.

A new carousel should follow the same split.

From the home screen, a swipe left or turning the knob past one step opens
the main menu. When the screen wakes from sleep, the rest of that turn is
swallowed (short cooldown) so it doesn't also open the menu.

## Tunable defaults

`src/main/globals.yaml` collects the values you're most likely to change:

| Global | Default | What |
|---|---|---|
| `display_timeout_s` | 30 | Screen sleep, seconds (saved) |
| `menu_timeout_s` | 60 | Return to clock, seconds (saved) |
| `buzzer_enabled` | true | Buzzer on/off (saved) |
| `motion_alert_enabled` | true | Motion Alert on/off (saved) |
| `motion_alarm_duration_s` | 20 | Alarm length, seconds (saved) |
| `motion_alarm_cooldown_s` | 10 | Minimum gap between alarms, seconds (saved) |
| `encoder_step_threshold` | 4 | Raw ticks per step (saved) |
| `scroll_anim_speed` | 20 | Pixels/second for scrolling menu text that doesn't fit (LVGL default: 25) |

Any label can reuse `id(scroll_anim_speed)` instead of its own speed.

## Battery

The M5Stack Dial V1.1 needs its hold pin high to stay on from battery.
`dial.yaml` turns on GPIO46 first thing at boot; the `power_hold` switch is
defined in `src/main/hardware.yaml`.

## NFC

The RC522 reader (`src/main/hardware.yaml`) sends every scanned card or tag
to Home Assistant as a tag scan (`homeassistant.tag_scanned`). New tags show
up under *Settings → Tags* and can trigger automations ("Tag is scanned").
Needs *Allow the device to perform Home Assistant actions*.

## Fridge (disabled)

`src/pages/fridge.yaml` (small food freshness cards) is kept but commented
out in `dial.yaml` and `src/pages/menu.yaml`. Un-comment both to bring it
back.
