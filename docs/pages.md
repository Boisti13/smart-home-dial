# Pages in detail

How each page works, which Home Assistant actions it calls, and where to
change it. Entity IDs are set in `src/main/entities.yaml`; see
[configuration.md](configuration.md).

- [Clock](#clock)
- [AC](#ac)
- [Radiators](#radiators)
- [Lights](#lights)
- [Timer and buzzer](#timer-and-buzzer)
- [Motion Alert](#motion-alert)
- [Settings](#settings)
- [Menu items](#menu-items)
- [Dial sensitivity](#dial-sensitivity)
- [Menu navigation](#menu-navigation)
- [Tunable defaults](#tunable-defaults)
- [Battery and sleep](#battery-and-sleep)
- [NFC](#nfc)
- [Home Assistant offline](#home-assistant-offline)
- [Diagnostics](#diagnostics)

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
`climate.set_fan_mode`. (The upstream swing button is gone since v2.0.0:
it only ever changed the screen, and the Pinguino has no swing.)

The power button toggles between `hvac_mode: cool` and `off`; the fan
control cycles `fan_mode` through `auto` / `low` / `medium` / `high`. These
strings must match what your integration supports: check `hvac_modes` and
`fan_modes` on the entity under *Developer Tools → States*, and edit the
`btn_air_power` on-click handler and the `air_fan_up` / `air_fan_down`
scripts if yours differ.

The home screen's AC tile shows the live fan speed and room temperature, read
from Home Assistant, so it stays right when the AC is changed elsewhere.

## Radiators

`src/pages/radiators.yaml`. Up to five rooms (`radiator_count`; names and
entities in `src/main/entities.yaml`), each with a page for:

| On the room page | How |
|---|---|
| Target temperature | turn the knob (`climate.set_temperature`, 0.5° steps) |
| On/off | the power button (`climate.set_hvac_mode`, `heat` ↔ `off`; edit `radiator_toggle_power` for other strings) |
| Room temperature | the line under the target, from `current_temperature` |
| Heating now | the arc and knob turn **orange** and the line says *Heating* while `hvac_action` is `heating` (valve open) |
| Preset | shown on the same line; a **short press of the front button** steps through `radiator_presets` (default Comfort → Eco → Frost → Boost) with `climate.set_preset_mode` |

The submenu's centre tile shows the same at a glance: *Heating 19.5°*,
*On 21.4°* or *Off 21.4°*, and updates live.

Each room can show a **window icon** next to its name, driven by a
`binary_sensor` contact: frame with cross panes = closed, frame without =
open. A room whose `radiator_N_window` is `binary_sensor.none` (Bathroom by
default) never shows it. The window sensors use `trigger_on_initial_state: true`, so
the icon is right straight after a reboot. ESPHome otherwise fires
`on_state` only on *changes*, not on the value Home Assistant sends on
connect. Keep that if you copy the pattern.

To change rooms or their number, edit only `src/main/entities.yaml`:
`radiator_count` (1–5) and `radiator_N_name` / `_entity` / `_window` per
slot. Slots above the count are ignored but still need a valid entity ID.
With fewer than five rooms the carousel leaves the places that would repeat
a room empty. More than five rooms needs another slot in
`src/pages/radiators.yaml` (sensors, the `*_list` substitutions, the arrays
sized `[5]`) and another dot widget.

## Lights

`src/pages/ha_lights.yaml`. The submenu lists the Dial's own LED ring
("Dial Ring", with the full color/effects page in `src/pages/light.yaml`)
plus up to six Home Assistant entities (`light_count`; names and entities
in `src/main/entities.yaml`). A `light.*` gets on/off and brightness (no
color, since not every bulb has RGB); anything else, e.g. a `switch.*`, gets
on/off only. Each tile shows the live on/off state from Home Assistant.

To change lights or their number, edit only `src/main/entities.yaml`:
`light_count` (1–6) and `light_N_name` / `light_N_entity` per slot. Slots
above the count are ignored but still need a valid entity ID.

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

`src/pages/settings.yaml`, seven entries:

| Entry | What it does | Default | Range |
|---|---|---|---|
| **Display Timeout** | Backlight off after this many seconds without input. The first touch or turn afterwards only wakes the screen. | 30 s | 5–300 s, steps of 5 |
| **Menu Timeout** | Back to the clock after this many seconds without input, however deep you are | 60 s | 5–300 s, steps of 5 |
| **Buzzer** | All buzzer sounds on/off | on | |
| **Motion Alert** | See [above](#motion-alert) | | |
| **Dial Sensitivity** | See [below](#dial-sensitivity) | 4 | 1–10 |
| **Menu Items** | Which pages the main menu shows; see [below](#menu-items) | all on | |
| **Sleep After** | Minutes the screen stays dark before deep sleep; see [Battery and sleep](#battery-and-sleep) | 5 min | 0 (never) – 60 min |

All values are saved across reboots. Both timeouts share one activity
tracker that any touch or turn resets. Defaults are in
`src/main/globals.yaml`; the ranges are in the `*_up` / `*_down` scripts in
`src/pages/settings.yaml`.

## Menu items

Timer, Lights, Radiators and AC can each be hidden from the main
menu. **Settings and Home are always there**, so there's always a way back.
The carousel and its dots show only the items that are on; with fewer than
five, the empty slots stay blank instead of repeating an item.

- **On the Dial**: *Settings → Menu Items* has one row per item, plus
  *Auto Season* (below). Turn the
  knob to switch the focused row on or off (green = on); tap a row or press
  the button to focus the next one.
- **From Home Assistant**: the device has four switches, *Menu Timer*,
  *Menu Lights*, *Menu Radiators*, *Menu AC*, plus *Menu Follows Season* (entity IDs like
  `switch.m5stack_dial_menu_timer`, under the device's *Configuration*).
  Usable in automations too, e.g. hide AC in winter.

**Season mode**: the Dial follows `season_mode_entity` (an `input_select`
with the options `Summer` and `Winter`). When it changes to *Winter*,
Radiators is switched on and AC off; to *Summer*, the other way round. It
acts only on a change of season (the last one is saved in
`last_season_mode`), so a reboot or reconnect doesn't undo what you set by
hand in between; the next change of season does. Other values
(`unavailable`, a third option) are ignored.

This can be turned off: the **Auto Season** row on the Menu Items page, or
the switch *Menu Follows Season* in Home Assistant (on by default). While
it's off, season changes leave the menu alone; turning it back on applies
the current season straight away.

Both change the same setting, and it is saved on the Dial across reboots
(new installs start with everything on). The switches are template
switches `menu_show_*` in `src/pages/menu.yaml`; `menu_step` moves the
selection to the next shown item, and the menu jumps there by itself if
the item it was on gets hidden. A hidden page still works, it is only left
out of the menu.

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
| Menu Items' focused row | `menu_items_encoder_accum` |
| Display Timeout, Menu Timeout, Dial Sensitivity | `display_timeout_encoder_accum`, `menu_timeout_encoder_accum`, `dial_sensitivity_encoder_accum` |
| Main, Lights, Radiators and Settings carousels | `menu_scroll_encoder_accum` (`src/pages/main.yaml`; shared, only one carousel is open at a time) |

Radiator temperature, light brightness and the color picker
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
| `scroll_anim_speed` | 20 | Pixels/second for scrolling menu text that doesn't fit (LVGL 9 default: 40) |

Any label can reuse it: `lv_obj_set_style_anim_duration(label, lv_anim_speed(id(scroll_anim_speed)), LV_PART_MAIN)` (see the `on_boot` lambda in `dial.yaml`).

## Battery and sleep

The Dial runs on its LiPo. Awake and idle it lasted about 10½ hours
(measured 2026-10-15, before v2.1.0), so `src/main/power.yaml` saves in two
stages:

| Stage | When | What happens | Wakes on |
|---|---|---|---|
| **Screen dark** | *Display Timeout* (30 s) without input | backlight off, LVGL paused (nothing is drawn), NFC reader powered down and not polled; Wi-Fi in light power-save all the time | turn, touch or button, instantly |
| **Deep sleep** | *Sleep After* (5 min) after the screen went dark | the ESP32 sleeps; offline in Home Assistant | **touch only**, then about 8 s until values are live |

**Touch only**, because deep sleep can only be ended by an RTC pin: the
touch controller's interrupt is on GPIO14 (RTC), the ring on GPIO40/41 and
the button on GPIO42 are not. After a touch the Dial boots straight to the
clock (the RTC chip has the time) with the *HA offline* pill until Home
Assistant is connected, instead of the boot animation.

**It stays awake** while *Stay Awake* is on, a timer runs or rings, Motion
Alert rings, or an OTA update runs.

**While it sleeps** Motion Alert can't fire, Home Assistant shows the
Dial's entities as unavailable and can't control it, and NFC tags aren't
read. A change of `season_mode_entity` while it sleeps only shows in the
menu at the next change. For an OTA update, wake it and turn on *Stay
Awake* first (Home Assistant can't reach it while it sleeps).

| Setting | On the Dial | In Home Assistant |
|---|---|---|
| Sleep After | *Settings → Sleep After*, turn the knob (0 = never) | `number.m5stack_dial_sleep_after` |
| Stay Awake | – | `switch.m5stack_dial_stay_awake` |

**Power latch.** The Dial only stays on from battery while its hold pin
GPIO46 is high. `dial.yaml` drives it high first thing at boot, and before
deep sleep `power.yaml` latches it (`gpio_hold_en`, with GPIO9, the
backlight, latched low); the next boot releases both. Without the latch the
board would switch itself off instead of sleeping, and only the front
button would turn it on again.

## NFC

The RC522 reader (`src/main/hardware.yaml`) sends every scanned card or tag
to Home Assistant as a tag scan (`homeassistant.tag_scanned`). New tags show
up under *Settings → Tags* and can trigger automations ("Tag is scanned").
Needs *Allow the device to perform Home Assistant actions*.

## Home Assistant offline

When Home Assistant's connection drops (HA restarting or updating, Wi-Fi
gone), a red **HA offline** pill appears at the top of every page: the
values on screen are the last ones received and buttons can't reach Home
Assistant. It disappears when Home Assistant connects again and re-sends
every value (`dial.yaml`: `api` → `on_client_disconnected`, the
`on_ha_connected` script in `src/pages/load.yaml`).

## Diagnostics

`src/main/diagnostics.yaml` adds entities for the device page in Home
Assistant: **Wi-Fi Signal** (dBm, every 60 s) and **Uptime** under
*Diagnostic*, and the buttons **Restart** and **Safe Mode Boot** under
*Configuration*. Safe mode starts only Wi-Fi and OTA, for when a new build
crashes before it can be updated normally.

## Fridge (removed)

Upstream's Fridge page (food freshness cards) was switched off since v1.3.0
and removed in v2.0.0. It is in the git history (`src/pages/fridge.yaml`
before v2.0.0) if you want it back.
