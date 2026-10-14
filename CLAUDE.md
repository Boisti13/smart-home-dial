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
  Reading is fine.
- **One change = one commit with everything it needs**: the YAML, the
  version bump, the docs and the CHANGELOG entry. No "docs later".
- **The owner compiles and flashes.** Validate locally; never upload to the
  device unasked.
- **No secrets in the repo.** `secrets.yaml` is ignored; entity IDs in
  `src/main/entities.yaml` are the owner's real ones and are fine to commit.
- `upstream` is Jasionf/smart-home-button. Don't push there.

## Checking a change

**All ESPHome testing runs on the dev box** (owner, 2026-10-14), not on
the Windows PCs. Push the branch, then:

```bash
ssh devbox 'cd ~/dev/smart-home-dial && git pull &&
  E=~/.venvs/esphome-2026.3/bin/esphome &&
  $E config dial.yaml &&     # every change
  $E compile dial.yaml'      # anything beyond text
```

- `secrets.yaml` there is a copy of `secrets.example.yaml` (dummy values,
  enough to validate and compile).
- ESPHome **2026.3.3** exactly (`requirements.txt`), venv
  `~/.venvs/esphome-2026.3` on the dev box. Newer ESPHome (2026.9.1, the
  shared `~/.venvs/esphome`) fails with "Platform missing" because the local
  `components/const` and `components/image` override the built-in ones.
  Upgrading ESPHome means updating or removing those overrides first.
- Uncommitted work can't be tested there. Commit it to `dev` and push (ask
  first), or copy the changed files over with `scp` for a dry run.
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
- Disabled features are commented out, not deleted (Fridge).
