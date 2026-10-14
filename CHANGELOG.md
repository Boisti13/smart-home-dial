# Changelog

All notable changes, newest first. Versions follow semantic versioning
([docs/development.md](docs/development.md#versions)); releases are merged to
`main` and tagged `vX.Y.Z`.

## Unreleased

## v1.8.0 — 2026-10-14

### Added
- **The main menu follows the season**: when Home Assistant's season select (`season_mode_entity`, default `input_select.season_mode`, options *Summer* / *Winter*) changes to *Winter*, the menu shows Radiators and hides AC; on *Summer* the other way round. Only a change of season switches them, so anything you set by hand in between stays until the next change. **Add `season_mode_entity` to your `src/main/entities.yaml`.**
- **Auto Season on/off**: a row on *Settings → Menu Items* and the switch *Menu Follows Season* in Home Assistant (on by default). Turning it back on applies the current season at once.

## v1.7.0 — 2026-10-14

### Added
- **Choose what the main menu shows**: Timer, Lights, Radiators, AC and Music can each be switched off. On the Dial under *Settings → Menu Items* (turn to switch the focused row, tap for the next row), or in Home Assistant with the new switches *Menu Timer*, *Menu Lights*, *Menu Radiators*, *Menu AC* and *Menu Music*. Saved across reboots; Settings and Home always stay ([how](docs/pages.md#menu-items)).

### Changed
- The main menu's dots show one dot per item that is switched on, centred. With fewer than five items, the carousel leaves the empty places blank instead of repeating items.

## v1.6.0 — 2026-10-14

### Added
- **Build and flash from Home Assistant's ESPHome app**: the project goes into a `smart-home-dial/` subfolder of the app's config folder, with a three-line top-level file ([how](docs/configuration.md#home-assistants-esphome-app)).

### Changed
- **ESPHome 2026.9.1** (was 2026.3.3), the version of Home Assistant's ESPHome app. The music page uses ESPHome's built-in SendSpin: play/pause/next/previous and volume go through the standard `media_player` actions, the playback state and volume come from the media player, the track length from a `track_duration` sensor, and album art from an `image:` entry with `platform: sendspin`.
- **Secret names** are the Dial's own, to fit a shared `secrets.yaml`: `dial_api_key` (was `api_encryption_key`). The fallback hotspot is called *M5Stack Dial Fallback*; `ap_ssid` is gone. **Rename it in your `secrets.yaml` when updating.**
- **No OTA password any more**: over-the-air updates are encrypted with the API key (`dial_api_key`) and must be, as ESPHome 2026.9 recommends; it also saves about 3.5 KB of flash. `ota_password` is no longer used. Updating from firmware before v1.6.0 over the air needs one step in between ([how](docs/configuration.md#over-the-air-updates)).

### Under the hood
- ESPHome 2026.9 brings **LVGL 9.5** (was 8.x). Lambdas use the new names: `get_lv_image_dsc()`, and the rolling-text speed is set as `lv_obj_set_style_anim_duration(…, lv_anim_speed(px/s))`.
- The local `components/` (copies of ESPHome's `const`, `image` and SendSpin from before they were released) are gone.
- Every local file path starts with `${dial_dir}/` (`.` by default), so the same YAML builds as the main file and as a subfolder.
- Image entries carry `platform: file`; the LED ring uses `channel_colors: GRB` instead of the deprecated `rgb_order`.

## v1.5.0 — 2026-10-14

### Added
- **Silence a Motion Alert** with a touch, a turn of the knob or a press of the button, instead of waiting out its duration. That input does nothing else.
- **Your own Motion Alert text**: `motion_alarm_message` in `src/main/entities.yaml` (default *Check Bedroom*).
- **NFC tags in Home Assistant**: cards and tags held to the Dial show up under *Settings → Tags* and can trigger automations.

### Changed
- **The clock shows your own outdoor sensors**: temperature, humidity and pressure come from three new substitutions (`outdoor_temperature_entity`, `outdoor_humidity_entity`, `outdoor_pressure_entity`), set to the *Virtual Outdoor* template sensors. Before, all three came from the met.no forecast. Wind still comes from `weather_entity`.
- The log level is `INFO` instead of `DEBUG`: less log traffic over Wi-Fi and less CPU time spent formatting messages.

## v1.4.1 — 2026-10-14

### Fixed
- **Clock shows German time**: the time zone was still upstream's `Asia/Shanghai`, so the clock ran 6–7 hours ahead. Both time sources in `src/pages/main.yaml` (`rtctime`, `esptime`) now use `Europe/Berlin`, with summer time.

### Under the hood
- **`CLAUDE.md`** with the working rules for this repository: `dev` only, validate before committing, what to update with a change, the release routine.
- README rewritten: shorter, with *Highlights*, tables for pages, hardware, install steps and Home Assistant entities, and a worked example (a motion alarm for the bedroom). The per-page notes moved to **[docs/pages.md](docs/pages.md)**, the common issues to **[docs/troubleshooting.md](docs/troubleshooting.md)**.
- `docs/configuration.md` lists every entity, the *Allow the device to perform Home Assistant actions* switch, and the time zone (set twice in `src/pages/main.yaml`).
- `docs/development.md` replaces `docs/project-structure.md` and `docs/release-checklist.md`: layout, naming, checking a change, versions, releases, upstream.
- Documented that the config only validates with the pinned ESPHome 2026.3.3: on 2026.9.1 the local `components/const` and `components/image` overrides break the `esp32` platform ("Platform missing").
- This changelog now uses the same layout as the owner's other projects (*Added / Changed / Fixed / Under the hood*, newest first).

## v1.4.0 — 2026-07-16

### Added
- **Motion Alert**: the buzzer sounds and the screen flashes red with *Check Bedroom* when a Home Assistant motion sensor triggers. One Settings entry with three rows (Enabled, Duration, Cooldown). Tap the front button or touch a row to focus it, then turn the knob. All three are saved across reboots.
- **Dial Sensitivity** in Settings: how many raw encoder ticks make one step (default 4, range 1–10, saved).
- Every page and menu that adjusts a value now uses that one setting: AC temperature/fan, Motion Alert, Display Timeout, Menu Timeout, Dial Sensitivity, and scrolling the Main/Lights/Radiators/Settings carousels. Before, only AC was debounced, and inconsistently.
- `motion_alarm_entity` substitution; new bell icon `motion_alert.png`.

### Changed
- Backing out of a submenu (Settings, Lights, Radiators or a radiator room) returns to the item you came from instead of the top of the list. Only opening fresh from the parent resets it.

### Fixed
- One click of the knob moved more than one step on undebounced pages. This encoder produces about 4 raw ticks per click (found by logging raw ticks one detent at a time); Dial Sensitivity above fixes it everywhere at once.
- Motion Alert's red flash no longer tears: it fades in and out over 150 ms instead of snapping, which the small LVGL render buffer handles cleanly.
- Motion Alert's Settings subtitle showed a missing-glyph square (a middle dot that wasn't in the font).

### Under the hood
- `motion_alert_enabled`, `motion_alarm_duration_s`, `motion_alarm_cooldown_s` and `encoder_step_threshold` globals; `enter_*` (reset and open) vs. `open_*` (open only) scripts for every carousel.

## v1.3.0 — 2026-07-14

### Added
- **Radiators** in the main menu: five rooms (Kitchen, Bedroom, Arthur, Living Room, Bathroom), each with target temperature and power.
- Live **window open/closed icon** on each radiator page from a Home Assistant contact sensor (Bathroom has none).
- **Timer alarm**: an RTTTL tune on the buzzer at zero, repeating until dismissed, stopping by itself after 60 s. A touch or a turn cancels it and resets the timer.
- **Buzzer** on/off in Settings (saved).
- Turning the knob more than one click on the home screen opens the main menu, like the swipe.
- Icons `radiator.png`, `speaker.png`, `window_open.png`, `window_closed.png`; Display/Menu Timeout reuse `timer.png`.

### Changed
- Menu tiles with names too long to fit scroll slower, at an adjustable speed (`scroll_anim_speed`).
- The Timer and Lights menu entries lost their subtitle line.
- The home screen's AC tile shows the live fan speed and room temperature instead of placeholder text.
- The Fridge menu item and page are commented out in `dial.yaml` and `src/pages/menu.yaml`, kept for later.
- Buzzer volume control was removed: neither `output.set_level()` nor `set_gain()` made an audible difference, because the RTTTL component re-applies its own gain on every note. The on/off switch replaces it.

### Fixed
- Waking the screen with the knob no longer also opens the menu: the step counter resets on sleep/wake and a short cooldown swallows the rest of the turn.
- The degree sign no longer overlaps the digits (`17.5°`): it's part of the same label now.
- The window icon shows the real state right after a reboot. ESPHome's `homeassistant` binary sensor gets the current state on connect but doesn't fire `on_state` for it; `trigger_on_initial_state: true` fixes that.
- Wrong AC entity, `hvac_mode` and `fan_mode` from the first setup; the README now says to check *Developer Tools → States* first.

### Under the hood
- `src/main/globals.yaml` collects the user-tunable values (scroll speed, timeouts, buzzer default) that were spread over the page files.

## v1.2.0 — 2026-07-13

### Added
- `src/main/globals.yaml` for the first user-tunable values.

### Changed
- The scroll speed of rolling text is adjustable instead of hard-coded.

## v1.1.0 — 2026-07-13

### Added
- **Lights** submenu: the LED ring plus five Home Assistant lights and one switch, each with on/off and brightness.
- **Settings** submenu: Display Timeout (screen sleep) and Menu Timeout (back to the clock), both saved across reboots.
- `secrets.example.yaml` template.
- Demo video link in the README.

### Fixed
- The AC page pointed at a placeholder entity; now a real climate entity with matching `hvac_mode` strings.

## v1.0.0 — 2026-06-04

Initial public release by [Jasionf/smart-home-button](https://github.com/Jasionf/smart-home-button), which this repository forks.

### Added
- Clock page with time, date, weather, humidity, AQI, pressure and wind.
- Round menu carousel.
- LED ring control: brightness, color, presets.
- AC page: target temperature, power, fan speed.
- Music page: SendSpin playback state, cover art, progress, transport buttons.
- Fridge status page.
- Countdown timer.
- Battery power hold for the M5Stack Dial V1.1.
