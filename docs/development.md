# Development

How this repository is laid out, named, checked and released. The checklist
for working on it is in [CLAUDE.md](../CLAUDE.md).

## Layout

| Path | What |
|---|---|
| `dial.yaml` | ESPHome entry point: includes every package, boot sequence, API, OTA, Wi-Fi, menu icons, LVGL setup, `project.version` |
| `secrets.example.yaml` | Public template for `secrets.yaml` |
| `requirements.txt` | Pinned ESPHome version |
| `src/main/hardware.yaml` | Dial hardware: I2C, display, touch, RTC, encoder, button, LED ring, buzzer/RTTTL, power hold, NFC |
| `src/main/entities.yaml` | Home Assistant entity IDs (substitutions) |
| `src/main/globals.yaml` | User-tunable defaults |
| `src/pages/*.yaml` | One LVGL page per feature, each an ESPHome package |
| `src/assets/fonts/`, `src/assets/images/` | Local fonts and small icons |
| `hardware/3d-models/` | STL files for the case |
| `docs/` | Guides; `docs/images/gallery/` holds the photos used in the README |

| Page file | Page |
|---|---|
| `load.yaml` | Boot/connecting screen |
| `main.yaml` | Clock and weather (home), encoder handling for the carousels |
| `menu.yaml` | Main menu carousel and its tiles |
| `light.yaml` | LED ring with color picker and effects |
| `ha_lights.yaml` | Lights submenu and per-light page |
| `radiators.yaml` | Radiators submenu and per-room page |
| `air.yaml` | AC |
| `music.yaml` | Music (SendSpin) |
| `timer.yaml` | Timer and alarm |
| `settings.yaml` | Settings submenu, Motion Alert, sleep/wake, timeouts |
| `fridge.yaml` | Fridge (disabled) |

## Naming

- Files: lowercase `snake_case`.
- LVGL IDs by prefix: `page_` pages, `lbl_` labels, `btn_` buttons, `arc_`
  arcs, `img_` images, `col_` colors, `font_` fonts.
- User-specific entity IDs only as substitutions in `src/main/entities.yaml`,
  never written into page files.
- Keep images small: the ESP32-S3 here has no PSRAM.

## Checking a change

```bash
cp secrets.example.yaml secrets.yaml   # once; dummy values are enough to validate
esphome config dial.yaml               # every change
esphome compile dial.yaml              # every change that isn't text-only
```

Use the pinned ESPHome (`requirements.txt`, the same version as Home
Assistant's ESPHome app). Build on Linux; in the owner's setup that's the
dev box (`~/.venvs/esphome`). Local file paths (images, fonts) start with
`${dial_dir}/` so the project also builds as a subfolder of HA's ESPHome
folder. On
Windows, ESP-IDF refuses Git Bash/MSYS and fails on long paths. The device
itself is flashed by the owner.

## Versions

Semantic versioning. The version lives in three places that must match:

- `dial.yaml` → `esphome: project: version:`
- the *version* badge at the top of `README.md`
- the heading in `CHANGELOG.md`

Patch for fixes, minor for new features or settings, major for changes that
need a new `entities.yaml` or break existing setups. Docs-only changes get
no bump; they go under `## Unreleased` in the changelog (*Under the hood*).

## Releases

Work happens on `dev`; `main` holds releases, tagged `vX.Y.Z`. A release:

1. On `dev`: rename `## Unreleased` in `CHANGELOG.md` to `## vX.Y.Z — date`,
   bump the version in the three places above, commit, push.
2. Merge `dev` into `main`, tag `vX.Y.Z`, push both.
3. `gh release create vX.Y.Z --notes-file <that changelog section>`.

Before anything goes public, check:

- `secrets.yaml`, `.esphome/`, virtual environments and editor folders are
  not committed (`.gitignore` covers them).
- No Wi-Fi names, passwords, API keys or tokens in the diff.
- `esphome config dial.yaml` passes with the example secrets.
- `THIRD_PARTY_NOTICES.md` still covers every copied component, font and icon.

## Upstream

This is a fork of [Jasionf/smart-home-button](https://github.com/Jasionf/smart-home-button),
set up as the `upstream` remote. Version 1.0.0 is upstream's initial release.
Everything after that was added here. Pull upstream fixes with
`git fetch upstream` and merge selectively.
