# Rev 9 — Dual SMT Connector Respin (u.FL + vertical MMCX)

*Created 2026-07-21. Working file: `patch_antenna_smp_rev9.kicad_pcb` (branched from Rev 8
snapshot `rev8_08`; Rev 8 remains the frozen fab-ready edge-launch panel). Snapshots in
`rev9_backups/`.*

## Concept

Every board carries **both** connector lands on the feed axis — populate either one:

- **u.FL / IPEX MHF1** (Hirose U.FL-R-SMT-1) — the de facto standard on Meshtastic/LoRa/
  ESP32 dev boards. Vertical mate, cable exits perpendicular to the board face.
- **Vertical SMT MMCX** (Würth WR-MMCX 66012102111404) — locking snap connector, mates the
  existing SMP-to-MMCX pigtail vertically. Fixes the edge-launch "double 90°" cable exit.

Both are placed using the **official KiCad 10 footprints** (datasheet-derived):
`Connector_Coaxial.pretty/U.FL_Hirose_U.FL-R-SMT-1_Vertical` and
`WR-MMCX_Wuerth_66012102111404_Vertical`.

## Default BOM (low-cost)

| Role | Default part | ~Cost (qty 1/100) | Notes |
|---|---|---|---|
| u.FL jack | Hirose U.FL-R-SMT-1(10) | $0.55 / $0.35 | LCSC clones (I-PEX MHF1 compatible, e.g. BWIPX-1-001E) ~$0.10 — same land pattern |
| SMT MMCX jack | Würth 66012102111404 | $3.40 / $2.60 | KiCad-official land. Cheaper LCSC MMCX SMT verticals exist but MUST be pattern-checked against this land before substituting |
| Pigtail (u.FL SKU) | u.FL → SMA-male, RG178, 100–200 mm | $3–5 | covers boxed-node users |
| Pigtail (Tab5 kit) | SMP-male → MMCX-male (existing) | — | mates the vertical MMCX directly |

## Per-board geometry (board coords: cx = column centre, ye = board bottom edge)

```
Feed:        3.1 mm (50 Ω) from patch bottom to ye−9.5
Neck:        0.6 mm wide from ye−9.5 to ye−3.5 (through MMCX wing channel, onto u.FL pad)
MMCX (JM_)   centre at (cx, ye−7), pad1 Ø1.47 on the neck
u.FL (JU_)   centre at (cx, ye−2.5), rotated 270° → signal pad at (cx, ye−4.03) on the neck
             (moved from ye−3.5 after DRC: pad-to-MMCX-wing gap is now 0.34 mm)
GND vias     2 per board inside the MMCX ground wings at (cx−1.71, ye−8.6), (cx+1.67, ye−8.6)
CONN_GND     Rev 8's edge pours + vias retained at (cx±1.75..±5, ye−3.5..ye) — u.FL ground
             pads overlap them directly (same net, solid connect)
```

**DRC accommodations (both required, both in place):**
- The project's `RF_50ohm` netclass (0.5 mm clearance) cannot be met inside the connector
  lands (wing channel 1.07 mm). `patch_antenna_smp_rev9.kicad_dru` adds rule
  `SMT_connector_local_clearance` → 0.2 mm for any pair involving `JU*`/`JM*`. Custom rules
  override netclass, and kicad-cli honors the .kicad_dru (verified empirically).
- `allow_soldermask_bridges_in_footprints = yes` — the Hirose footprint's own mask aperture
  intentionally bridges its pads; this board setting suppresses that false positive.
- FEED/XFORM zones use solid pad connection (thermal spokes starve on a 0.6 mm neck, and
  solid is correct for RF anyway).

## Verified DRC state (kicad-cli, 2026-07-21)

**16 violations, all expected panel artifacts — 0 real defects:**
- `unconnected_items` ×16 — 8× RF_IN feed-zone↔feed-zone across *different boards* (one net
  spanning 9 V-score-separated boards; each board's pad→feed→patch chain IS connected) and
  8× CONN_GND↔GND_PANEL headless-fill bookkeeping (via geometrically verified inside both
  fills; resolves on GUI Fill All Zones).
- `courtyards_overlap` ×9 — the JU/JM pair on each board; never both populated. Waive.
- `invalid_outline` ×7 — V-cut lines on Edge.Cuts, same as Rev 8. Waive.

Gone vs Rev 8: all copper_edge_clearance waivers, spring-pin unconnected pads,
isolated patch copper (patches now DC-connected through the connector pads), silk clipping.

**Why the 0.6 mm neck is required here** (unlike the reverted Rev 8 edge-launch taper): the
Würth MMCX ground wings leave only a 1.07 mm copper channel into the centre pad, and the
u.FL signal pad is 1 mm wide between its own grounds. A 3.1 mm feed physically cannot reach
either pad without shorting to ground. The neck is ~5 mm of ~110 Ω line ≈ 0.07 λg at
2.45 GHz — a small, acceptable discontinuity. **Verify with NanoVNA when it arrives.**

Unpopulated-land stub: whichever connector is not fitted leaves ≤2.5 mm of open 0.6 mm
stub on the feed (≈0.036 λg) — negligible at 2.4 GHz.

## What Rev 9 removed (vs Rev 8)

- 9× edge-launch MMCX 135-3711-801 footprints and their routed notches — **Edge.Cuts is now
  a plain 174×210 rectangle + 4 V-cut lines** (8 segments total). No internal slots at all.
- The 9 stale 2×2 mask apertures and MMCX silk labels; stale feed fab outlines.
- With no notch and no edge-flush pads: the old copper_edge_clearance waivers and
  "signal pad in the notch" unconnected items should disappear from DRC. Ground plane is 9
  clean per-board tiles, 35,008 mm² total (more copper than Rev 8 — no notch cutouts).

## Expected DRC残 items

- Courtyard overlap between JU_/JM_ pairs (never both populated) — waive.
- V-cut invalid_outline ×8 and isolated RF_IN patch copper — same as Rev 8, waive.
- u.FL pad1 connects to the feed neck via zone fill — confirm connection after GUI refill.

## Open items

- [ ] GUI review: fill zones (B), inspect connector cluster at 2× zoom
- [ ] DRC pass in GUI with exclusions
- [ ] Confirm Würth wing-channel clearance (0.235 mm each side of neck) passes the board's
      0.2 mm netclass clearance — if the fab wants more, narrow the neck to 0.5 mm
- [ ] NanoVNA S11 sweep of both connector variants vs the Rev 8 edge-launch baseline
- [ ] Pick the low-cost LCSC MMCX substitute only after overlaying its drawing on this land
