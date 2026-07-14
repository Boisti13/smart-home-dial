# Changelog

All notable changes to this project are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project uses [Semantic Versioning](https://semver.org/).

## [1.3.0] - 2026-07-14

### Added

- **Radiators** main menu item and submenu: five per-room climate control pages (Kitchen, Bedroom, Arthur, Living Room, Bathroom) with target temperature and power control.
- Live window open/closed indicator on each Radiators room page, driven by a Home Assistant `binary_sensor` contact sensor (Bathroom has none by default).
- Timer alarm: plays an RTTTL tune through the onboard buzzer when the countdown reaches zero, repeats until dismissed, and auto-stops after 60 seconds. Cancelable by touch or encoder rotation, which also resets the timer.
- **Buzzer** on/off toggle in Settings (`buzzer_enabled` global, persisted across reboots).
- Rotating the encoder more than one detent from the home screen now opens the main menu (in addition to the existing swipe gesture).
- New icons: `radiator.png`, `speaker.png`, `window_open.png`, `window_closed.png`; the Display/Menu Timeout Settings entries now reuse `timer.png`.
- `src/main/globals.yaml` centralizes user-tunable values (scroll speed, timeouts, buzzer default) that were previously scattered across page files.

### Changed

- Rolling/marquee label text (menu tiles with names too long to fit) now scrolls at a slower, user-tunable speed (`scroll_anim_speed`) instead of LVGL's default.
- Removed the subtitle line under the Timer and Lights main menu entries.
- Home screen's AC menu tile now shows live fan speed and room temperature instead of static placeholder text.

### Fixed

- Encoder wake-from-sleep gesture no longer bleeds into the "open menu" gesture; the step accumulator resets on sleep/wake and a short cooldown window swallows the rest of the waking turn.
- Degree symbol no longer collides with the temperature digits (e.g. `17.5°`) — the symbol is now part of the same label instead of a separately positioned one.
- Window indicator now reflects the sensor's real state immediately after a reboot/reflash. Root cause: ESPHome's `homeassistant` binary_sensor platform does receive the current state from Home Assistant right away on reconnect, but by default does not fire the `on_state` automation for that first value — only for state changes afterward. Fixed by setting `trigger_on_initial_state: true` on each window binary_sensor.
- Fixed wrong AC entity/hvac_mode/fan_mode assumptions from the initial setup (now documented as "check Developer Tools → States first" in this README).

### Removed

- Attempted buzzer volume control (`output.set_level()`, then `set_gain()`) — neither produced an audible change since the RTTTL component re-applies its own gain on every note. Replaced with the simple on/off toggle above.

### Disabled

- Fridge menu item and page are commented out (not deleted) in `dial.yaml` and `src/pages/menu.yaml`, kept for reference/restoration.

## [1.2.0] - Rolling text & globals cleanup

### Added

- `src/main/globals.yaml` for the first user-tunable values.

### Changed

- Rolling-text scroll speed is now configurable instead of hardcoded.

## [1.1.0] - Home Assistant lights, Settings, AC fix

### Added

- Home Assistant Lights submenu: onboard LED ring plus five `light.*` entities and one `switch.*` entity, each with a simple on/off + brightness page.
- Settings submenu: Display Timeout (backlight sleep) and Menu Timeout (idle return to home), both persisted across reboots.
- Example `secrets.yaml` template.
- Demo video link in the README.

### Fixed

- AC entity configuration pointed at a placeholder entity ID; corrected to a real climate entity and hvac_mode strings.

## [1.0.0] - 2026 - Initial public release

### Added

- Clock page with time, date, weather, humidity, AQI, pressure, and wind speed.
- Circular menu navigation for the round display.
- Onboard LED ring control (brightness, color, preset flows).
- AC (climate) control page: target temperature, power, fan speed.
- Music page with SendSpin playback state, cover art, progress, and transport controls.
- Fridge status page.
- Countdown timer page.
- Battery power-hold support for M5Dial V1.1.
