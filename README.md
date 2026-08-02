# Project Platypus — 2.4 GHz Microstrip Patch Antenna

A directional 2.4 GHz patch antenna on 2-layer FR4, designed from first principles —
no antenna generator, no reference design, no copied footprint. Every dimension is
derived from the cavity model and Hammerstad–Jensen microstrip synthesis, then
validated on manufactured hardware over a real RF link.

**Measured result: +12.5 dB over the host device's internal chip antenna.**

| | Internal chip antenna | External patch (Design C) | Delta |
|---|---|---|---|
| RSSI | −40.5 dBm (avg) | −28.0 dBm (best) | **+12.5 dB** (≈18× sensitivity) |

Measured on an M5Tab5 (ESP32-C6-MINI-1U) with its RF switch toggled to the external
MMCX antenna port. Full method in [`TEST_PROCEDURE.md`](TEST_PROCEDURE.md).

> Board photos: the panel is purple ENIG and photographs beautifully — see the Hackster
> project this repo accompanies.

---

## The board

One 58 × 210 mm V-scored panel, three antennas. Each board is 58 × 70 mm with an
M3 mounting pattern (44 mm square, matching the M5Tab5), a 50 Ω microstrip feed, and an
edge-launch MMCX jack (Cinch/Johnson 135-3711-801) seated in a routed notch.

### Why three designs on one panel

Impedance matching a patch has two textbook solutions. Rather than pick one on faith,
the panel carries **all three variants side by side** — same substrate, same fab run,
directly comparable:

| Design | Matching method | Geometry | Predicted Rin |
|---|---|---|---|
| **A** | Inset feed, calculated match | y₀ = 9.81 mm, slot 6.3 mm | 50 Ω (matched) |
| **B** | Inset feed, deliberate mismatch | y₀ = 7.50 mm, slot 6.3 mm | 97 Ω (control) |
| **C** | Quarter-wave transformer | Z_t = 100 Ω, w = 0.709 mm, ℓ = 17.98 mm | 50 Ω via λ/4 |

Design B is intentionally mismatched — the control that tests whether the matching
model is doing what the math claims.

---

## Design math (all first-principles)

Substrate: FR4, h = 1.6 mm, εr = 4.4, tan δ ≈ 0.02, 2 layers, ENIG.

```
Patch width      W = c/(2f₀)·√(2/(εr+1))                    = 38.04 mm
Effective εr     εr_eff = (εr+1)/2 + (εr−1)/2·(1+12h/W)^−½   = 4.086
Fringing ext.    ΔL = 0.412h·((εr_eff+0.3)(W/h+0.264)) /
                          ((εr_eff−0.258)(W/h+0.8))         = 0.742 mm
Patch length     L = c/(2f₀√εr_eff) − 2ΔL                    = 29.44 mm
50 Ω feed width  (Hammerstad–Jensen, FR4 1.6 mm)             = 3.1 mm
Inset depth      Rin(y₀) = Rin_edge·cos²(πy₀/L) → 50 Ω       = 9.81 mm
λ/4 transformer  Z_t = √(50·200) = 100 Ω, w = 0.709 mm, ℓ    = 17.98 mm
```

Resonance check: L_eff = 29.44 + 2(0.742) = 30.924 mm, √εr_eff = 2.021
→ **f₀ = 2.3996 GHz**.

**Inset slot width follows the Salmony method:** clearance each side of the feed must be
≥ the substrate height h, giving a 6.3 mm total slot (3.1 + 2×1.6). A slot barely wider
than the feed is a common mistake that creates spurious resonances and spoils the match.

### Honest analysis notes

- **Return loss is not signal loss.** A mismatch costs only 10·log₁₀(1−|Γ|²): even
  Design B's deliberate 97 Ω feed point loses well under 2 dB of delivered power. The
  A/B/C comparison therefore needs an S11 sweep (VNA), not RSSI — the +12.5 dB headline
  comes from directivity and escaping the host enclosure, not matching finesse.
- **Front-to-back is modest by design.** The ground plane extends only 10–12 mm
  (≈0.1 λ) past the patch, so realistic F/B is ~6–10 dB. Fine for "point the gain at
  the far node"; don't expect a deep rear null indoors.

---

## Manufacturing notes (learned on real hardware)

- **ENIG is required** — HASL's uneven surface degrades RF pads.
- JLCPCB parses V-score geometry from **Edge.Cuts/GM1 only** — order notes and
  Dwgs.User lines are ignored. Their V-cut minimum panel size is 70 × 70 mm.
- The Edge.Cuts outline must be closed loops with exactly two segments per vertex; the
  MMCX notches are separate internal rectangles sharing no vertices with the perimeter.
- The MMCX signal pad sits inside the routed notch — contact is made by the connector's
  spring pin, and the corresponding DRC "unconnected" items are by design (waived in
  `patch_antenna_smp.kicad_dru` / DRC exclusions).
- Solder mask over the radiator shifts resonance down slightly (~10–30 MHz) but does
  not prevent operation.

## Repository layout

```
patch_antenna_smp.kicad_pcb    KiCad board — the manufactured, field-tested revision
patch_antenna_smp.kicad_pro    KiCad project
patch_antenna_smp.kicad_dru    Custom DRC rules (MMCX edge-clearance waivers)
DESIGN_NOTES_v72.md            Full design derivation, BOM, fab notes
TEST_PROCEDURE.md              5-phase RF test protocol
DRC7.13.1.rpt                  DRC report of the released revision
gerbers/  +  gerbers.zip       Fab package as manufactured (JLCPCB)
BOM/Links.txt                  Component sourcing
```

## Bill of materials (per board)

| Ref | Part | Notes |
|---|---|---|
| J1 | MMCX 135-3711-801 (Cinch/Johnson) | Edge-launch jack, board notch |
| H1–H4 | M3 × 6 mm screw + 5 mm nylon standoff | Nylon preferred for RF isolation |
| — | Pigtail: SMP-male → MMCX-male, RG178, ~100 mm | Host-device dependent |

**Fab settings:** 2-layer FR4 1.6 mm, ENIG, purple mask, white silk,
V-score at 70 mm and 140 mm from panel top.

## License

Hardware and documentation are released under the
**[CERN Open Hardware Licence v2 — Strongly Reciprocal](LICENSE)** (CERN-OHL-S v2).
You may use, study, modify, manufacture and sell this design, provided derivative
hardware designs are shared under the same terms.

---

*Project Platypus — Open Source RF*
