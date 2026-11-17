# ADR 0002 — System context: this repo's role

Status: PROPOSED 2026-11-17. The full system map is canonical in
`mcfrs-incidents/docs/adr/0002-system-architecture.md`; this ADR states only
this repo's place in it.

## Role

The last hop: a desk display that shows what `mcfrs-incidents` has already
decided. It owns firmware only, and no data formats.

```
 mcfrs-incidents (apogee-gs)
   └─ GET /api/device/pages  (LAN, plain HTTP; payload owned by mcfrs-incidents)
        ▼
 mcfrs-desk-display (ESP8266 + 128×64 OLED): fetch ─► draw ─► rotate
```

## Inputs and outputs

- **In:** one small payload of ready-to-draw pages from `mcfrs-incidents`
  (fields defined there).
- **Out:** pixels. Plus its own Wi-Fi RSSI, heap and uptime, drawn locally on
  the system-status page.

## Boundaries

- Never talks to the scanner, transcription, `apogee-sdr`, weather or any
  upstream service directly. Everything comes through `mcfrs-incidents`.
- Does not decide what is major, what is stale, or which station to show.
  It honours the payload's skip flags and data-age fields.

## Trigger to revisit

The display needs data the payload does not carry. Extend the payload in
`mcfrs-incidents`; do not add a second source to the device.
