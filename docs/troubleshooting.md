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
| **Wrong or missing light or room** | Check `light_count` / `radiator_count` and the slot's `*_name` / `*_entity` in `src/main/entities.yaml` ([Lights](pages.md#lights), [Radiators](pages.md#radiators)). |
| **Config fails after updating to v2.0.0** | The light, switch, radiator and window substitutions were replaced by numbered slots; rewrite that part of `src/main/entities.yaml` from the repository's copy ([configuration](configuration.md#home-assistant-entities)). |
| **Front button does nothing on a room page** | It sets the next preset from `radiator_presets`; the climate must offer those names in `preset_modes`, and the device needs *Allow the device to perform Home Assistant actions*. |
| **Red *HA offline* pill** | Home Assistant is unreachable: restarting, or the Dial lost Wi-Fi. It goes away by itself once HA is back ([details](pages.md#home-assistant-offline)). |
| **Window icon stuck on closed after a reflash** | The room's `binary_sensor` needs `trigger_on_initial_state: true` ([Radiators](pages.md#radiators)). |
| **One click moves more than one step** | Raise *Settings → Dial Sensitivity*. This encoder gives about 4 raw ticks per click ([Dial sensitivity](pages.md#dial-sensitivity)). |
| **Motion Alert too often / not often enough** | Change Cooldown and Duration on the Motion Alert page ([Motion Alert](pages.md#motion-alert)). |
| **No buzzer volume control** | There is only on/off ([Timer and buzzer](pages.md#timer-and-buzzer)). |
| **OTA upload: "requires an encrypted OTA connection" or "the device did not offer OTA encryption"** | The first message: the uploader has no key; build from the same `secrets.yaml` with `dial_api_key`. The second: the Dial runs firmware from before v1.6.0; update it once over USB or with a password build ([OTA](configuration.md#over-the-air-updates)). |
| **`esphome upload` ignores your edits** | `upload` reflashes the last build. Use `esphome run`, or `compile` first. |
| **`esphome config` fails on `image:`** | You're on an ESPHome older than 2026.9 (or an old checkout with `components/`). Install the pinned version: `pip install -r requirements.txt`. |
| **ESPHome app: image or font file not found** | The top-level file must set `dial_dir: ./smart-home-dial` ([configuration](configuration.md#home-assistants-esphome-app)). |
| **First build can't download fonts** | Connect to the internet once, or replace `gfonts://` fonts with local files. |
| **Turns off on battery** | The V1.1 power-hold pin (GPIO46) must be switched on at boot ([Battery and sleep](pages.md#battery-and-sleep)). |
| **Home Assistant can't reach the Dial** (actions do nothing, values don't change) | Probably in deep sleep: touch it. For updates or automations that need it, turn on *Stay Awake* or set *Sleep After* to 0 ([Battery and sleep](pages.md#battery-and-sleep)). |
| **Values take up to a minute after a touch-wake** | The fast-reconnect webhook isn't set up or doesn't match: `dial_wake_url` (ESPHome secrets) must end in the `dial_wake_webhook` ID (HA secrets), and `packages/smart_home_dial.yaml` must be loaded; its automation's trace shows each call ([setup](configuration.md#fast-reconnect-after-a-touch-wake)). |
| **Turning the ring doesn't wake it** | Deep sleep only wakes on touch; the ring and button only wake a dark screen. |
| **Doesn't wake from deep sleep at all** | The power latch didn't hold or the touch controller didn't signal: press the front button to power it on, set *Sleep After* to 0, and report it. |
