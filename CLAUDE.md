# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

ZMK firmware config for a **Lily58** split keyboard (58-key, 6x4+4 thumb, split halves). No local build — firmware compiles in GitHub Actions and is downloaded as an artifact, then flashed via UF2 drag-and-drop. There is no source code; this repo is pure ZMK/Zephyr device-tree config.

## Build & flash flow

- **No local toolchain.** Editing `config/lily58.keymap` (or `.conf`/`build.yaml`) and pushing triggers `.github/workflows/build.yml`, which calls the upstream `zmkfirmware/zmk` reusable workflow `build-user-config.yml@main`.
- Build matrix lives in `build.yaml` (note: `.yaml`, not `.yml`). It produces two halves: `lily58_left` and `lily58_right`. Left includes the ZMK Studio snippet (`studio-rpc-usb-uart` + `-DCONFIG_ZMK_STUDIO=y`).
- Board is pinned to `nice_nano@2.0.0//zmk` (nice!nano v2 / hardware-versioned syntax). The earlier `nice_nano_v2` form was migrated away from — keep the `@2.0.0//zmk` form.
- **Flash:** double-tap RST within 1s → `Nicenano` mass-storage volume appears → copy LEFT `.uf2` to left half, RIGHT to right half. Never delete existing files on the volume; flashing one half does not require the other.
- After flash a transient error on the keyboard is expected (ZMK quirk, not a bug).

## Editing the keymap

`config/lily58.keymap` is a Zephyr devicetree (`.dtsi` include) file. Three layers, referenced by `&mo <n>`:

- **0 = default_layer** (alpha + numbers row)
- **1 = raise_layer** (function keys, arrows, symbols) — entered via `&mo 1` on right thumb
- **2 = lower_layer** (Bluetooth select, shifted symbols, EP power) — entered via `&mo 2` on left thumb

Key mechanics in use:
- `&mt LCTRL ESC` — mod-tap: hold = Ctrl, tap = Esc (the left home-row key).
- Layer toggles `&mo 1` / `&mo 2` on the thumb cluster.
- **Combos** (`combos` node): `combo_clear_bluetooth` fires `&bt BT_CLR` when key positions `52 42 0` (LOWER + `[` + `~`) are pressed within 50ms. Key positions are 0-indexed across the full 58-key matrix (top-left = 0, row-major).
- `sensor-bindings = <&inc_dec_kp C_VOL_UP C_VOL_DN>` on each layer — rotary encoder volume control (encoder hardware itself is off; `CONFIG_EC11` is commented out in `.conf`).
- Bluetooth: `&bt BT_SEL 0..4` select profile, `&bt BT_CLR` clears current profile pairing.

When editing layer bindings, the comment block above each `bindings = <...>` is the visual ASCII layout — **keep it in sync** with the actual bindings; it's the only readable map of the physical layout.

`config/lily58.keymap.old` is a prior version kept for reference; not built.

## Feature flags (`config/lily58.conf`)

Toggling Kconfig here changes firmware features without touching the keymap:
- `CONFIG_ZMK_DISPLAY=y` — OLED enabled.
- `CONFIG_ZMK_STUDIO_LOCKING=n` — ZMK Studio editable on-the-go.
- `CONFIG_ZMK_POINTING=y` — mouse support.
- BLE: `CONFIG_ZMK_BLE_EXPERIMENTAL_CONN=y`, `CONFIG_BT_CTLR_TX_PWR_PLUS_8=y` (stronger TX power).
- Battery reporting + split central battery proxy enabled.

## GUI alternatives (no-code editing)

Per README, two GUI paths edit this same repo:
- **KeymapEditor** (nickcoutsos.github.io/keymap-editor) — recommended; writes changes back to GitHub and triggers the build automatically.
- **ZMK Studio** — beta; does not persist to GitHub, lacks tap-dance/some behaviors.

If a teammate's commits look auto-generated on the keymap, they likely came from KeymapEditor — preserve its formatting where practical.
