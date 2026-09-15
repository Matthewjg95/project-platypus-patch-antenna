# Project Platypus — Rev 8 Plan

## Status
Rev 7.13.1 is manufactured, assembled, and functional; gain vs the M5Tab5 chip antenna
is not yet characterised. Rev 8 folds in every lesson learned from that build. This file
is the durable change list and the record of design decisions made while implementing it.

The known-good manufactured file (`patch_antenna_smp.kicad_pcb`, the 1×3 panel) is **not**
edited in place. Rev 8 work happens on a copy so the shipped design is always recoverable.

---

## Implementation status — 2026-07-05

Rev 8 is built on the working copy `patch_antenna_smp_rev8.kicad_pcb` (staged snapshots in
`rev8_backups/`). Validated with `kicad-cli 10.0.1` (delta check — it does not apply the
GUI `.kicad_dru` waivers or exclusions, so by-design items always appear).

**Done and verified:**
- MMCX GND pads enlarged to 1.5×2.0 mm; feed tapered 3.1→1.0 mm; 2.0×2.0 signal-pad mask
  aperture + patch F.Mask openings. DRC delta clean (no new categories) on the 1×3.
  (snapshots `rev8_02`)
- 3×3 panel 174.1×210.1 mm: 9 boards (JA–JI + 36 M3), 40 zones, 18 vias. Outer perimeter
  + 3 bottom C-notches + 6 internal A/B notches rebuilt clean. GND_PANEL extended to full
  panel and set to **keep-all islands** so every board keeps its own ground (inset 0.5 mm
  from score lines) — fill 34,909 mm² across all 9 boards. V-cut lines added on Edge.Cuts
  between all rows (y=113.5, 183.5) and columns (x=239, 297). (snapshots `rev8_04`, `rev8_05`)
- Broadened `.kicad_dru` edge-clearance waiver to all F.Cu GND pads (covers JA–JI).
- Title/rev strings updated to Rev 8.

**Final panel DRC (all expected / waived / by-design — no real defects):**
- `copper_edge_clearance` ×18 — MMCX GND pads on notch walls (waived by `.kicad_dru`).
- `unconnected_items` ×16 — 10 RF_IN (spring-pin contact, by design) + 6 GND (see below).
- `isolated_copper` ×32 — 23 RF_IN patch radiators (by design; reached only via connector
  pin) + 9 GND edge slivers.
- `invalid_outline` ×8 — the four V-cut lines touching the perimeter (the Edge.Cuts V-cut
  tradeoff; exclude).
- `silk_over_copper` ×6 — Design B/C labels (cosmetic; same as Rev 7.13).

**Known non-issue — 6 GND `unconnected_items`:** the left-side connector grounds in columns
1–2 (`CONN_GND_*_L_C1/C2`) show a ratsnest line to GND_PANEL. Proven a headless-fill
connectivity artifact: each via is geometrically inside both its F.Cu CONN_GND fill and the
B.Cu ground plane, on the same net (netcode 2), identical to the working column-0 / right-side
connections. It is functionally harmless regardless — the MMCX body shorts pads 2 & 3, and the
right-side via grounds every connector. Should resolve on GUI Fill All Zones. Do NOT "fix" by
adding in-pad vias near the notch (they violate edge clearance).

**Open decision — V-cuts on Edge.Cuts vs. continuous ground:** current design puts V-cuts on
Edge.Cuts (per the JLCPCB lesson that Dwgs.User is ignored). Cost: 8 `invalid_outline`
warnings + the ground fragmenting into per-board tiles (handled with keep-all). Alternative:
keep V-cuts off Edge.Cuts for a continuous plane + clean DRC, and communicate V-cut positions
to JLCPCB another way. Left as-is pending user preference.

## RF design parameters (locked — unchanged from Rev 7)
```
Frequency        2.4 GHz            Patch width W     38.04 mm
Substrate        FR4 1.6 mm         Patch length L    29.44 mm
εr 4.4, tanδ 0.02  εr_eff 4.086     ΔL (fringing)     0.742 mm
50 Ω feed width  3.1 mm             Rin at edge       ~200 Ω
Design A inset   y0 = 9.81 mm → 50 Ω (matched)
Design B inset   y0 = 7.50 mm → 97 Ω (deliberate mismatch)
Design C xfmr    Zt = 100 Ω, w = 0.709 mm, L = 17.98 mm (no inset)
```
No RF geometry changes in Rev 8 — the electrical design is validated. Rev 8 is purely
manufacturability + panelisation improvements.

---

## Change list

### 1. MMCX 135-3711-801 footprint — manufacturability fixes
Problems seen on Rev 7.13:
- **Feed crowding.** 3.1 mm feed in a 3.50 mm notch = 0.2 mm clearance per side; the
  connector GND legs nearly shorted to the signal feed. (Rev 7.13 workaround: solder
  mask over the feed insulated the legs.)
- **No usable mask opening on the signal pad.** Had to scrape mask by hand before
  soldering. (The pad *had* F.Mask in its layer list, but the 0.71 mm opening was too
  small/tight to be useful.)
- **GND pads too small** for Class 3 workmanship (0.97 × 1.47 mm).

Rev 8 fixes (geometry decided during implementation — see "Decisions" below):
- ~~Taper the feed from 3.1 mm to 1.0 mm.~~ **REVERTED 2026-07-05.** The routed connector
  notch (the "hole") already separates the feed from the GND legs — the feed fill is clipped
  at the notch edge and the spring pin bridges to the signal pad, so there is no real feed-to-
  GND crowding. Tapering just removed copper and added a needless impedance discontinuity.
  Feed is back to straight 3.1 mm microstrip (the proven Rev 7.13 geometry).
- **Signal-pad mask aperture 2.0 × 2.0 mm**, added as a mask-only pad centred on the
  signal pad, so the whole pin-contact area is bare ENIG.
- **GND pads enlarged to 1.5 × 2.0 mm.** Grown *outward in x* (inner edge kept exactly
  on the notch wall) and *upward in y* (bottom edge kept flush with the board edge).
  The connector's ground leg still lands well inside the enlarged pad.

### 2. Solder mask off the radiator — bare-vs-masked A/B test
Rev 7.13 shipped with **no** F.Mask openings over the patch copper (mask covered the
patches). It worked but detuned resonance ~10–30 MHz downward. Rev 8 makes patch masking a
deliberate experimental variable: **the LEFT column (all 3 designs A/B/C) is bare copper**
(F.Mask opening over the patch), and the **middle + right columns are masked**. That gives a
bare-vs-masked pair for every design so the mask detuning can be measured per design without
confounding it with the design difference. The bare patches read gold/ENIG, the masked ones
purple — self-identifying, no silk label needed (a note is added to the User.Drawings legend).
The **feed stays masked** everywhere on purpose — mask over a transmission line is negligible
and it insulates the connector GND legs from the feed. Only the left-column radiators go bare.
(Earlier interim state had all 9 patches bare; narrowed to one column on user request.)

### 3. Panelisation — 3×3, V-cuts on Edge.Cuts
Rev 7.13 was a 58 × 210 mm 1×3 panel. Two problems at JLCPCB:
- **Below V-cut minimum.** JLCPCB's V-cut minimum panel size is 70 × 70 mm; our 58 mm
  width was under it, so they could not V-score horizontally between designs.
- **V-scores on Dwgs.User were ignored.** JLCPCB parses geometry on Edge.Cuts / GM1,
  not Dwgs.User. Order notes are not a substitute.
- JLCPCB "helpfully" multiplied our panel into a 3×3 array (174 × 210 mm) anyway.

Rev 8 plan: design the full **3×3 panel = 174 × 210 mm** up front (both dimensions
> 70 mm), 3 columns × 3 rows = 9 boards, one design per row (A / B / C), three copies of
each across the columns. V-cut lines go on **Edge.Cuts** between every row and column.

> **Open technical caveat (V-scores on Edge.Cuts):** a V-score line that spans the full
> panel width and touches the perimeter creates T-junctions, which KiCad reports as
> `invalid_outline`. The clean board shape (outer perimeter + separate internal MMCX
> notch rectangles, no shared vertices) is what keeps DRC quiet. Putting full-span
> V-cut lines back on Edge.Cuts re-introduces those warnings. The resolution is to keep
> the true outline clean and add the V-cut lines as independent Edge.Cuts segments,
> accepting/【excluding】the resulting outline warnings — that is the tradeoff for JLCPCB
> reading them from GM1. This is validated with `kicad-cli pcb drc` during implementation.

### 4. Consider soldermask-defined openings for all connector pads
Evaluate SMD-defined vs mask-defined for the GND and signal pads. Low priority; only if
the enlarged pads still show mask slivers in the fab preview.

---

## Decisions made during implementation

### MMCX footprint geometry (origin = connector centre, at the board edge; +y = toward/past the edge)
| Element | Rev 7.13 | Rev 8 | Rationale |
|---|---|---|---|
| Signal pad (1) | Ø 0.71 @ (0, +3.30) | unchanged copper | pin contact point is fixed |
| Signal mask | 0.71 (= pad) | **2.0 × 2.0 aperture @ (0, +3.30)** | bare area for hand/reflow solder |
| GND pad L (2) | 0.97 × 1.47 @ (−2.235, −0.735) | **1.5 × 2.0 @ (−2.5, −1.0)** | bigger, inner edge on notch wall, bottom flush to edge |
| GND pad R (3) | 0.97 × 1.47 @ (+2.235, −0.735) | **1.5 × 2.0 @ (+2.5, −1.0)** | mirror of pad 2 |

Notch wall at rel x = ±1.75 (abs 208.25 / 211.75). Enlarged GND pad inner edge = 2.5 −
0.75 = 1.75 → sits exactly on the notch wall (same as Rev 7.13's 2.235 − 0.485 = 1.75),
covered by the existing `edge_clearance (min 0mm)` DRC rule. Bottom edge at rel y = 0 =
board edge (V-score line); grows upward into the board to rel y = −2.0.

### Feed geometry — straight 3.1 mm (taper reverted)
Feed zones are plain 3.1 mm rectangles x [208.45+dx, 211.55+dx] from the patch bottom to the
board edge (dx = 0 / 58 / 116 per column). The fill is clipped by the routed MMCX notch, and
the connector spring pin bridges to the signal pad — no copper crowding, no taper needed.
- Design A feed: y 85.2212 → 113.5
- Design B feed: y 155.2212 → 183.5
- Design C feed: y 243.201 → 253.5 (plus the unchanged XFORM_C 0.709 mm transformer)

### Patch mask openings
Filled polygon on F.Mask over each patch bounding box (radiator only, not feed):
- A: (190.982, 55.779)–(229.018, 85.221)
- B: (190.982, 125.779)–(229.018, 155.221)
- C: (190.982, 195.779)–(229.018, 225.221)

---

## Zone priority hierarchy (unchanged — prevents zones_intersect)
```
PATCH_A/B/C   priority 3   XFORM_C      priority 2
FEED_A/B/C    priority 1   GND_PANEL    priority 0   CONN_GND_*  priority 0
```
- ONE B.Cu GND_PANEL zone covering the whole panel (spans all 3 columns in Rev 8).
- CONN_GND zones: solid fill, `connect_pads yes (clearance 0)`, one via each to GND_PANEL.
- No stitching vias — all F.Cu copper is RF_IN, nothing to stitch.

## Build / verify workflow
1. Edit the Rev 8 copy.
2. **Fill All Zones (press B)** in KiCad — filled_polygon data in the file is stale after
   any zone-outline edit until refilled. `kicad-cli pcb drc` refills before checking.
3. `kicad-cli pcb drc --schematic-parity ... ` — expect only the waived items:
   - `unconnected_items` on RF_IN signal pads (spring-pin contact) — exclude.
   - MMCX GND edge clearance — waived by `.kicad_dru`.
   - Edge.Cuts outline warnings from V-cut lines — exclude (see caveat above).
4. Re-export gerbers to `gerbers/` and re-zip for JLCPCB.

## Fab order (JLCPCB)
Purple mask, white silk, ENIG, FR4 1.6 mm, 2-layer. 3×3 panel 174 × 210 mm. V-cut on
GM1 between all rows and columns. No mask on patch copper. MMCX slots 3.5 × 4.5 mm routed.

## KiCad S-expr pitfalls (carried forward)
- `(justify center)` invalid in older formats — omit (centre is default).
- `fill solid` invalid — use `fill yes`.
- `;;` comments break the parser.
- `gr_rect` cannot carry a net — use `zone` for copper pours.
- Edge.Cuts outline must be closed loops, exactly 2 segments per vertex, no T-junctions.
- Same-net zones touching on the same layer need different priorities.
