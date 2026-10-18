# Working on this repository

Notes for Claude Code (and anyone else) working on Smart Home Dial: ESPHome
firmware for an M5Stack Dial that controls Home Assistant. Details live in
[docs/development.md](docs/development.md); this is the checklist.
Infrastructure (Home Assistant, servers, credentials) is **not** in this
repository. If the folder above this one has its own `CLAUDE.md`, it has
those notes.

## Ground rules

- **Start with `git pull` on `dev`.** The owner works from more than one
  computer; never assume the checkout is current.
- **Work on `dev`, commit and push to `dev`.** Don't merge to `main`, tag or
  release unless asked. "Make a new release" means the
  [release routine](docs/development.md#releases).
- **Ask before side effects**: pushing, flashing, or touching Home Assistant.
  Reading is fine. One standing exception: copying a new version to Home
  Assistant's ESPHome folder (see *Home Assistant copy* below).
- **One change = one commit with everything it needs**: the YAML, the
  version bump, the docs and the CHANGELOG entry. No "docs later".
- **The owner compiles and flashes.** Validate locally; never upload to the
  device unasked.
- **No secrets in the repo.** `secrets.yaml` is ignored; entity IDs in
  `src/main/entities.yaml` are the owner's real ones and are fine to commit.
- `upstream` is Jasionf/smart-home-button. Don't push there.

## Checking a change

**ESPHome checks run on the build server**, CT 117 `esphome-builder` on
pve_rptu_dev, never locally as long as pve_rptu_dev is reachable (owner,
2026-10-14); only if it isn't, fall back to a local venv. From this folder,
on any PC or the dev box:

```bash
bash ../claude-setup/scripts/esphome-check.sh dial.yaml config    # every change
bash ../claude-setup/scripts/esphome-check.sh dial.yaml           # + compile: anything beyond text
```

- It sends the working tree (uncommitted changes included, ignored files
  such as `secrets.yaml` not) to the container and runs ESPHome there as
  user `esphome`; output ends with `== config OK` / `== compile OK in N s`
  and RAM/flash use. No push needed.
- Secrets there are a copy of `secrets.example.yaml` (dummy values, enough
  to validate and compile). The build cache stays between runs
  (`~esphome/checks/smart-home-dial/`): ~30 s for a small change, ~90 s
  from cold.
- ESPHome **2026.9.1** (`requirements.txt`), the version of Home
  Assistant's ESPHome app, which builds what gets flashed (on the same build
  server). The check uses the builder's own ESPHome
  (`/opt/esphome-builder`); when the app updates, bump `requirements.txt`
  and update the builder (`pip install -U 'esphome-device-builder[esphome]'`
  in CT 117), then check again.
- **Home Assistant copy**: the ESPHome app builds from
  `/homeassistant/esphome/smart-home-dial/` (`dial.yaml`, `src/`) via the
  stub `/homeassistant/esphome/m5stack-dialv1.yaml`, which sets
  `dial_dir: ./smart-home-dial`. **Every new version goes there unasked**
  (owner, 2026-10-14), so the owner can flash it straight away: once it is
  pushed to `dev` and compiles on the dev box, back up the old folder
  (`tar czf /config/project-backups/esphome/smart-home-dial/smart-home-dial-vX.Y.Z.tar.gz.bak-<timestamp>`),
  replace `dial.yaml` and `src/` with `git archive <commit> dial.yaml src`
  (remove the old `src/` first so deleted files go too), and check that the
  files match. Then the owner flashes. Every local file path in the YAML
  starts with `${dial_dir}/`, or the HA build breaks.
- Power (`src/main/power.yaml`): deep sleep wakes only from RTC pins
  (GPIO 0–21; touch INT is GPIO14, the ring and button are not). GPIO46
  must stay high or the board switches off: latch it before deep sleep and
  release it at boot (`dial.yaml` on_boot priority 1000). Anything that
  draws while the screen is dark goes into `sleep_display` / `wake_display`.
- **Home Assistant package** `packages/smart_home_dial.yaml` goes to
  `/config/packages/smart_home_dial.yaml` (back up into
  `/config/project-backups/home-assistant/` first), then `ha core check`
  and reload automations. Its webhook ID is the secret `dial_wake_webhook`
  (HA) / the end of `dial_wake_url` (ESPHome); never in the repo.
- Hardware facts: no PSRAM (keep images and LVGL buffers small); the encoder
  gives ~4 raw ticks per click (debounce value pages through
  `encoder_step_threshold`, see [docs/pages.md](docs/pages.md#dial-sensitivity)).

## Versions to bump

| Change | Bump |
|---|---|
| fix | patch: `dial.yaml` (`project.version`), README badge, CHANGELOG heading |
| new feature, page or setting | minor, same three places |
| change that needs a new `entities.yaml` or breaks setups | major |
| docs only | none; add to `## Unreleased` → *Under the hood* |

## What to update with a change

- `CHANGELOG.md`: a section `## vX.Y.Z — YYYY-MM-DD` with *Added / Changed /
  Fixed / Under the hood*, written for users, newest first. If there's a
  `## Unreleased` section, rename it to the new version and add to it.
- `README.md`: *Highlights*, the pages table, the entity table, whatever
  users notice.
- `docs/pages.md` for how a page works, `docs/configuration.md` for new
  entities or settings, `docs/troubleshooting.md` for new pitfalls,
  `docs/development.md` for layout changes.
- New entity → substitution in `src/main/entities.yaml`, never hard-coded in
  a page file.
- New user-tunable default → `src/main/globals.yaml`
  (`restore_value: true` if it's changeable on the device).
- New carousel → `enter_*` (reset + open) and `open_*` (open only) scripts,
  like the existing ones.
- Copied components, fonts or icons → `THIRD_PARTY_NOTICES.md`.

## Conventions

- One LVGL page per feature in `src/pages/`, included as a package in
  `dial.yaml`.
- IDs: `page_`, `lbl_`, `btn_`, `arc_`, `img_`, `col_`, `font_`.
- `homeassistant` binary sensors whose current state matters at boot get
  `trigger_on_initial_state: true`, except alarms (Motion Alert must not
  fire on boot).
- Lights and radiators are **slots** configured in `src/main/entities.yaml`
  (`*_count`, `*_N_name`, `*_N_entity`); page code reads them through the
  `*_list` substitutions at the top of `ha_lights.yaml` / `radiators.yaml`.
  Never write a room or light name into a page file.
- Unused features are deleted, not commented out (git history keeps them;
  the Fridge page went in v2.0.0).
