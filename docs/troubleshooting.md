# Troubleshooting

| Problem | Fix |
|---|---|
| **Device unavailable in Home Assistant** | The `api:` section must be in `dial.yaml` and port 6053 reachable from Home Assistant. |
| **Buttons do nothing in Home Assistant** (AC, lights, radiators) | Turn on *Allow the device to perform Home Assistant actions* in the ESPHome device's options. For AC and radiators, also check that the `hvac_mode` / `fan_mode` strings exist on the entity ([AC](pages.md#ac), [Radiators](pages.md#radiators)). |
| **Clock is hours off** | Firmware older than v1.4.1 still used upstream's `Asia/Shanghai`; update. Outside Germany, set your own zone in both entries in `src/pages/main.yaml` ([configuration](configuration.md#time-zone)). |
| **Wrong temperature, humidity or pressure** | Change `outdoor_*_entity` in `src/main/entities.yaml`; each must be a sensor whose state is the number. |
| **Wrong wind speed** | Change `weather_entity` in `src/main/entities.yaml`. |
| **AQI stuck on `--`** | `aqi_entity` must be an air-quality sensor whose *state* is the index, not an attribute of the weather entity. |
| **Wrong AC** | Change `climate_entity` in `src/main/entities.yaml`. |
| **Wrong or missing light** | Change the matching `light_*_entity` / `switch_*_entity` ([Lights](pages.md#lights)). |
| **Window icon stuck on closed after a reflash** | The room's `binary_sensor` needs `trigger_on_initial_state: true` ([Radiators](pages.md#radiators)). |
| **One click moves more than one step** | Raise *Settings → Dial Sensitivity*. This encoder gives about 4 raw ticks per click ([Dial sensitivity](pages.md#dial-sensitivity)). |
| **Motion Alert too often / not often enough** | Change Cooldown and Duration on the Motion Alert page ([Motion Alert](pages.md#motion-alert)). |
| **No buzzer volume control** | There is only on/off ([Timer and buzzer](pages.md#timer-and-buzzer)). |
| **Music page shows unavailable** | Use the player that really plays audio, not the Dial entity ([configuration](configuration.md#music-player)). |
| **Reboots when album art arrives** | Keep artwork small and don't raise LVGL memory; the Dial has no PSRAM. |
| **`esphome upload` ignores your edits** | `upload` reflashes the last build. Use `esphome run`, or `compile` first. |
| **`esphome config` fails on `image:` or `sendspin`** | You're on an ESPHome older than 2026.9 (or an old checkout with `components/`). Install the pinned version: `pip install -r requirements.txt`. |
| **ESPHome app: image or font file not found** | The top-level file must set `dial_dir: ./smart-home-dial` ([configuration](configuration.md#home-assistants-esphome-app)). |
| **First build can't download fonts** | Connect to the internet once, or replace `gfonts://` fonts with local files. |
| **Turns off on battery** | The V1.1 power-hold pin (GPIO46) must be switched on at boot ([Battery](pages.md#battery)). |
