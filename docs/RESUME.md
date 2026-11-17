# RESUME — mcfrs-desk-display

## Status (2026-11-17)

Repo created; design only. No firmware, no commits on GitHub.

Decisions:
- `docs/adr/0001-desk-display.md` (PROPOSED): a thin device and a smart
  server, the six pages, staleness rules, and the build sequence.
- `docs/adr/0002-system-context.md` (PROPOSED): this repo's role in the MCFRS
  system.

## Blocked on

- **The device endpoint in `mcfrs-incidents`** (ADR 0001, build step 1). The
  firmware has nothing to fetch until it exists. It can be built against the
  mock adapter now.

## Next steps

1. `mcfrs-incidents`: build `/api/device/pages` against the mock adapter
   (TDD), with real weather and system status.
2. This repo: PlatformIO project; reuse the Plane Spotter's display init and
   page-cycle patterns; fetch and draw. Hardware-test before commit.
3. Resolve ADR 0001's open questions (transfer phrasing, battalion chief
   designators, endpoint access, NWS zone, `apogee-sdr` schedule location).

## Open branches

None (no commits yet).
