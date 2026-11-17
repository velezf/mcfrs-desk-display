# ADR 0001 — Desk display: ESP8266 rotating pages fed by mcfrs-incidents

Status: PROPOSED 2026-11-17. Drafted in `mcfrs-incidents` as its ADR 0002,
never committed there; moved here when this repo was created. System map:
`mcfrs-incidents/docs/adr/0002-system-architecture.md`.

## Context

The end goal for the MCFRS projects is a desk display in the style of the Plane
Spotter (`esp8266-plane-spotter`): an ESP8266 driving a small OLED and
cycling through pages.

Fixed by Frank, 2026-11-17:
- **Hardware:** the same screen as the current Plane Spotter build, a 2.42"
  SSD1309, 128×64, monochrome, on a NodeMCU ESP8266 with U8g2. The
  authority is `esp8266-plane-spotter/firmware/plane_spotter/plane_spotter.ino`
  (display section).
- **Firmware repo:** this one, `mcfrs-desk-display`.
- **Server:** `mcfrs-incidents`, running on `apogee-gs` 24/7.
- **Pages:** six, below.

Constraints that shape the design:
- **ESP8266 resources.** Usable heap is tens of KB, and TLS is slow and
  memory-hungry. The Plane Spotter already budgets around both
  (`esp8266-plane-spotter/firmware/plane_spotter/config.example.h`: a small search box to avoid RAM exhaustion, and
  synchronous HTTPS on the render thread).
- **128×64 text budget.** About 21–25 characters × 7–8 lines, depending on
  font, with one line used for a page header.
- **The live feed is intermittent by design.** The scanner shares one dongle
  with other jobs under `apogee-sdr`'s time-sliced rotation
  (`apogee-sdr/docs/adr/0002-time-sliced-rotation.md`). Its consumer
  staleness contract applies here: carry a last-heard timestamp, show data
  age, and never treat silence during another job's slot as "nothing is
  happening".

## Decision

**A thin device and a smart server.** The server prepares every page. The
device fetches one small payload and draws it.

- **Endpoint:** `GET /api/device/pages` (name provisional) returns ready-to-draw
  page records: header, up to *n* short lines, a data-age field, and a
  skip flag. Target size: a few hundred bytes per page, under 2 KB in
  total. All filtering, truncation to the character budget, the
  focus-station logic, the major-incident rule and weather happen
  server-side. The payload's authority is the Pydantic model that serves it
  in `mcfrs-incidents` (alongside `mcfrs-incidents/api/mcfrs_api/models.py`);
  this firmware cites it.
- **Transport:** plain HTTP on the home LAN (device and Pi on the IoT VLAN),
  polled on a fixed interval. No TLS on the device. The endpoint carries no
  medical detail: the existing server-side privacy transforms apply first.
- **Rotation:** the device cycles pages with a configurable dwell. Pages
  marked skip are not shown, so the rotation has no empty screens.

### Pages

| # | Page | Content | Rule |
|---|---|---|---|
| 1 | **Active incidents** | One line per incident: call type, short address, first unit. Newest first; sub-pages if more than fit | All open incidents county-wide |
| 2 | **Station** | Each unit at the focus station: status, current incident, transfers in and out | Station from `focus_station` (`mcfrs-incidents/api/mcfrs_api/config.py`, default 14). Roster from `mcfrs-incidents/api/mcfrs_api/data/stations.json`. Units transferred *in* (cover move-ups) are listed; home units transferred *out* show where they went |
| 3 | **Major incident 1** | Type, address, elapsed time, battalion chief(s), unit count, working talkgroup | **Major = at least one battalion chief assigned.** Box alarms are always major: by MCFRS protocol a box is dispatched with two BCs (Frank, 2026-11-17). Oldest major first |
| 4 | **Major incident 2** | Same layout | Shown only while two or more majors are open; otherwise skipped |
| 5 | **Weather** | Current conditions, short forecast, active NWS alerts for the county | Fetched server-side from the National Weather Service API (no key). The server sends a generic client User-Agent, not personal contact details |
| 6 | **System status** | Dongle owner and slot (scanner or another job), last dispatch heard and its age, transcription backlog, server health, plus device Wi-Fi RSSI, heap and uptime | Server fields from `apogee-sdr`'s status/schedule and this server's own health; device fields drawn locally |

### Staleness rules (from `apogee-sdr` ADR 0002)

- Every page shows data age. When the scanner is off-air, the header shows
  an **OFF-AIR** marker plus the time since last heard.
- **No incident is closed by inference while the scanner is off-air.**
  "Talkgroup went quiet" only counts during the scanner's own slots.
- Unit status on page 2 is **inferred**. It comes from dispatch transcripts
  and control-channel metadata (`mcfrs-dispatch-transcribe` ADRs 0001 and
  0003), not from CAD. "Available" means "not seen assigned since last
  heard", and it is labelled that way.

## Build sequence (each step is a gate Frank clears)

1. **Server endpoint against the mock adapter** (in `mcfrs-incidents`). The
   existing seeded simulation (`INCIDENT_SOURCE=mock`) exercises pages 1–4. Weather and
   system status are real from day one. TDD per the working agreement.
2. **Firmware** in this repo: fetch, parse with a small fixed
   document, render with U8g2, rotate. Reuse the Plane Spotter's display
   init and page-cycle patterns. Hardware-test before commit (memory:
   Frank flashes and watches the board first).
3. **Deploy `mcfrs-incidents` on `apogee-gs`** (Docker Compose: db + api). Pick
   ports that avoid those already taken on the box (authority:
   `mcfrs-scanner/CLAUDE.md`, "Box facts"; check `ss -ltnp` first).
4. **Switch the incident source** from mock to scanner-derived when
   `mcfrs-dispatch-transcribe` produces events. The device does not change.

## Alternatives

- **Device calls upstream services directly** (weather, incident source):
  rejected. Too much JSON and TLS for an ESP8266, and business rules would
  be duplicated in firmware.
- **Bigger or colour display** (e.g. a 240×320 TFT): more room, but Frank
  chose to match the Plane Spotter. Revisit if pages 2–3 cannot fit.
- **ESP32:** more heap and easier TLS; not needed if the server does the
  work.

## Open questions

1. How do transfers sound on 7A2? The phrasing feeds page 2's parser
   (verify against recordings).
2. Battalion chief unit designators, and how they appear in dispatch and as
   radio IDs (needed for the page 3 rule; ADR 0003's alias learning).
3. Endpoint access: LAN-only with no auth, or a device token? LAN-only fits
   the IoT VLAN design. Decide when deploying (step 3).
4. Which NWS forecast zone and alert area to use for "the county", and the
   home point for current conditions.
5. Where `apogee-sdr` publishes its slot schedule (its ADR 0002, open
   question 2), so page 6 and the OFF-AIR marker can read it.

## Trigger to revisit

The 128×64 budget cannot carry pages 2–3 legibly, or a second dongle makes
the scanner feed continuous (the OFF-AIR rules then rarely fire).
