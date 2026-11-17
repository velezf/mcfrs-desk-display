# CLAUDE.md — working memory for mcfrs-desk-display

ESP8266 + 128×64 OLED that rotates six pages drawn from `mcfrs-incidents`.
Purpose and pages: `README.md`. Design: `docs/adr/0001-desk-display.md`.

**Resume from `docs/RESUME.md`.**

## Working agreement

Gates, TDD and commit format: `~/.claude/CLAUDE.md`. Cross-project conventions
(cite, don't restate; hollow checks): `~/code/CLAUDE.md`.

Authorities for literal values:
- **Display and board** (controller, bus, pins, U8g2 setup): the Plane Spotter
  firmware, `esp8266-plane-spotter/firmware/plane_spotter/plane_spotter.ino`,
  until this repo has its own firmware.
- **Page payload** (fields, sizes): the Pydantic model behind
  `/api/device/pages` in `mcfrs-incidents`. Firmware cites it and never
  defines its own.
- **System roles and data flow:** `mcfrs-incidents/docs/adr/0002-system-architecture.md`.

## What must never happen

- **No business logic on the device.** Filtering, the major-incident rule,
  station logic and weather belong to the server (ADR 0001).
- **Never commit `config.h`** (Wi-Fi credentials, server address). Ship
  `config.example.h` only, as the Plane Spotter does.
- **Hardware-test before commit.** Frank flashes and watches the board before
  a firmware commit.
