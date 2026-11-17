# mcfrs-desk-display

A desk display for Montgomery County (MD) Fire/Rescue activity, in the style
of the Plane Spotter (`esp8266-plane-spotter`): an ESP8266 and a 128×64 OLED
cycling through six pages:

1. Active incidents (county-wide)
2. Station status (focus station, default Station 14, with transfers in and out)
3. Major incident 1 (any incident with a battalion chief assigned)
4. Major incident 2 (only while two or more majors are open)
5. Weather
6. System status (including whether the scanner is on-air)

The device is deliberately thin. It fetches ready-to-draw pages from
`mcfrs-incidents` (running on `apogee-gs`) over the home LAN and draws them.
All the logic lives on the server.

- Design: `docs/adr/0001-desk-display.md`
- Where this fits among the MCFRS repos: `docs/adr/0002-system-context.md`,
  and the canonical map in `mcfrs-incidents/docs/adr/0002-system-architecture.md`
- Status and next steps: `docs/RESUME.md`

Not affiliated with Montgomery County or MCFRS. Unit status shown is
inferred from radio, not taken from CAD.

## Status

Design only. No firmware yet. The server endpoint it reads is not built yet
either (build sequence in ADR 0001).
