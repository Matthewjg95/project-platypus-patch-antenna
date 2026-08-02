# Project Platypus — 2.4 GHz Patch Antenna
## Design Notes Rev 7.2 — Release for Manufacture

---

## Project Overview

A panelised 2.4 GHz microstrip patch antenna PCB designed for external antenna integration with the M5Tab5. Three distinct impedance-matching designs are included on a single V-scored panel for comparative RF testing. The winning design will be used in Rev 8 production boards. Performance is quantified using the M5Tab5's own WiFi RSSI and the BMI270 gyroscope to render a live 3D radiation pattern on the display.

---

## Board Summary

| Parameter | Value |
|-----------|-------|
| Target frequency | 2.4 GHz (802.11b/g/n, Bluetooth, Zigbee) |
| Panel size | 58 × 210 mm (3 boards × 70 mm each) |
| Individual board | 58 × 70 mm |
| Substrate | FR4, h = 1.6 mm, εr = 4.4, tan δ = 0.02 |
| Copper layers | 2 (F.Cu patch + feed, B.Cu full ground plane) |
| Surface finish | ENIG |
| Solder mask | **Purple** |
| Silkscreen | **White** (both sides) |
| Panelisation | V-score between boards (2 score lines) |
| Mounting | M3 NPTPH, 44 mm square pattern (matches M5Tab5) |
| Connector | MMCX 135-3711-801 end-launch jack, board notch |

---

## Patch Antenna Calculations

All dimensions derived from first principles for 2.4 GHz on FR4 1.6 mm.

### Patch Width

```
W = c / (2f₀) × √(2 / (εr + 1))
W = 3×10⁸ / (2 × 2.4×10⁹) × √(2 / 5.4)
W = 38.04 mm
```

### Effective Dielectric Constant

```
εr_eff = (εr+1)/2 + (εr-1)/2 × (1 + 12h/W)^(-0.5)
εr_eff = 4.086
```

### Patch Length

```
ΔL = 0.412h × (εr_eff+0.3)(W/h+0.264) / ((εr_eff-0.258)(W/h+0.8))
ΔL = 0.742 mm

L_eff = c / (2f₀√εr_eff) = 30.93 mm
L = L_eff - 2ΔL = 29.44 mm
```

### 50 Ω Microstrip Feedline

```
Feed width W_feed = 3.1 mm  (Hammerstad-Jensen, FR4 1.6 mm)
εr_eff_feed ≈ 3.38
```

### Inset Slot Width (Salmony method)

Slot clearance on each side of the feedline must be ≥ substrate height h to prevent capacitive coupling into the patch copper:

```
Slot clearance = h = 1.6 mm each side
Total slot width = W_feed + 2h = 3.1 + 3.2 = 6.3 mm
```

This corrects the common mistake of using a slot barely wider than the feedline, which creates spurious resonances and degrades the impedance match.

---

## Three Designs

### Design A — Inset Feed, Calculated Match (Reference)

The inset feed exploits the cosine-squared current distribution of the patch. Input impedance varies from ~200 Ω at the radiating edge to 0 Ω at the patch centre. The inset depth y₀ selects the exact point where impedance equals 50 Ω.

```
Rin(y₀) = Rin_edge × cos²(π y₀ / L)
50 = 200 × cos²(π y₀ / 29.44)
y₀ = 9.81 mm
Rin at y₀ = 50.0 Ω  ✓ matched
Slot: 6.3 mm wide × 9.81 mm deep
```

**This is the primary design.** Expected best broadside gain.

### Design B — Inset Feed, Deliberate Mismatch (Experiment)

Identical geometry to Design A but with a shallower inset. The feed point is higher up the patch where impedance is ~97 Ω rather than 50 Ω. This controlled mismatch tests the sensitivity of the antenna to inset depth error and provides data to validate the matching formula experimentally.

```
y₀ = 7.50 mm  (shallower than calculated optimum)
Rin at y₀ = 97.0 Ω  → mismatch Γ = (97-50)/(97+50) = 0.32
Expected return loss ≈ 9.8 dB  (vs >20 dB for Design A)
```

**Expected measurably worse RSSI than Design A.** If the delta is not observed, it suggests the 50 Ω assumption for the feedline is the dominant error, not the inset depth.

### Design C — Quarter-Wave Transformer (Alternative Matching)

No inset notch. The 50 Ω feedline transitions to a higher-impedance transformer section before connecting to the centre of the patch's radiating bottom edge. The transformer impedance Z_t and length λ_g/4 perform the same 200 Ω → 50 Ω transformation as the inset, but via an external line section rather than feed positioning.

```
Z_t = √(Z₀ × Rin) = √(50 × 200) = 100 Ω
Transformer width W_t = 0.709 mm  (100 Ω microstrip, FR4 1.6 mm)
εr_eff_t = 3.118
λ_g/4 at 2.4 GHz = c / (4 × f₀ × √εr_eff_t) = 17.98 mm
```

The patch is a plain rectangle with no notch slots. The feed enters at the bottom-centre radiating edge. The step from 3.1 mm (50 Ω) to 0.709 mm (100 Ω) is an abrupt width change — the standard Salmony method. Bandwidth of this match is similar to the inset approach.

**Useful for comparison:** if Design C outperforms Design A it suggests the inset slots are introducing parasitic coupling despite the 1.6 mm clearance.

---

## Connector — MMCX 135-3711-801

End-launch jack receptacle, surface mount, Bel/Cinch group. Selected for compatibility with the M5Tab5's MMCX external antenna ports.

| Pad | Net | Size | Position (rel. to footprint origin) |
|-----|-----|------|--------------------------------------|
| 1 (signal) | RF_IN | Ø 0.71 mm | (0, +3.30 mm) — inside notch |
| 2 (GND left) | GND | 0.97 × 1.47 mm | (−2.235, −0.735 mm) |
| 3 (GND right) | GND | 0.97 × 1.47 mm | (+2.235, −0.735 mm) |

**Board notch:** 3.50 mm wide × 4.50 mm deep, rectangular, routed into the board bottom edge. The signal pad sits 3.30 mm below the board edge inside the routed slot. The feedline copper terminates exactly at the board edge — it does not extend into the notch.

**Pigtail cable:** SMP male to MMCX male, RG178, ~100 mm. Plugs into M5Tab5 MMCX port and snaps into J1 on this PCB.

---

## Panel & Fabrication

### V-Score Panelisation

The three boards share a single continuous `Edge.Cuts` outline (one panel). V-score lines run the full panel width at y = 113.5 mm and y = 183.5 mm in panel coordinates (= 70 mm and 140 mm from the panel top edge). JLCPCB will route the MMCX notch slots first, then apply the V-score blade.

**JLCPCB order note to include:**
> "V-score at 70 mm and 140 mm from panel top edge (panel y = 113.5 mm and y = 183.5 mm). Three MMCX slots (3.50 × 4.50 mm) pre-routed at each board bottom edge. No solder mask on F.Cu patch copper areas. Purple solder mask, white silkscreen, ENIG finish."

### Critical Fabrication Requirements

- **No solder mask on patch copper** — solder mask over the radiating element changes εr_eff and shifts the resonant frequency. Specify a solder mask opening over the patch polygon on each board (the Gerbers include this as a B.Mask opening).
- **ENIG finish** — required for RF performance. HASL creates uneven surface topology under the connector pads and patch copper.
- **Purple solder mask** — select in JLCPCB order form. No extra cost for 2-layer boards. Turnaround +2 days vs green.
- **White silkscreen** — default for purple mask, no action needed.

### Mounting

M3 NPTPH holes (3.2 mm drill, no plating) in a 44 mm square pattern, 7 mm from each board side, 5 mm from board top. This matches the M5Tab5's M3 mounting point positions exactly. Use M3 socket-head cap screws with nylon spacers to mount the antenna PCB above the M5Tab5 body with the patch face pointing away from the device.

---

## Bill of Materials

| Ref | Part | Qty (per board) | Notes |
|-----|------|-----------------|-------|
| J1 | MMCX 135-3711-801 (Bel/Cinch) | 1 | DigiKey, Mouser |
| H1–H4 | M3 × 6 mm socket head cap screw | 4 | Nylon preferred for RF isolation |
| H1–H4 | M3 nylon standoff, 5 mm | 4 | Between this PCB and M5Tab5 |
| — | SMP male to MMCX male pigtail, RG178, 100 mm | 1 | Pasternack, Amazon |
| PCB | This file, JLCPCB 2-layer | 1 panel (3 boards) | Purple, ENIG |

**Note:** Order 2 panels (10 boards) for testing redundancy. At JLCPCB pricing this is approximately $8–12 shipped.

---

## Key Design Decisions & Revision History

| Rev | Change |
|-----|--------|
| 1.0 | Initial inset-feed patch, SMP edge connector |
| 2.0 | Updated to MMCX 135-3711-801, M3 holes, 44 mm pattern |
| 3.0 | Three-design panel (A/B/C), V-score panelisation |
| 4.0 | Patch moved inside mounting hole square, board notch corrected to rectangular |
| 5.0 | Single panel Edge.Cuts outline (boards actually connected), patch U-polygon with visible notch slots |
| 5.1 | Feed copper capped at board edge (was extending 3.3 mm past edge into routed notch — fabrication error) |
| 6.0 | Panel properly connected, V-score lines on Dwgs.User |
| 7.0 | Slot width corrected from 4.1 mm to 6.3 mm per Salmony method (h clearance each side). Purple solder mask specified. Project Platypus branding. |
| 7.1 | Logo moved to B.SilkS back silkscreen, notes box placed outside board boundary |
| 7.2 | Logo replaced with "Project Platypus" text in lower-right corner. Notes box corrected. |
| 7.3–7.13 | DRC iteration: zone priorities fixed, GND zone unified, vias removed (no F.Cu GND to stitch to), connector GND zones with vias added, MMCX edge clearances waived, Edge.Cuts rebuilt as outer perimeter + 2 internal notch rectangles (no T-junctions), V-score positions verified at y=113.5 and y=183.5. **Release for manufacture.** |

---

## Expected Performance

| Metric | Design A | Design B | Design C |
|--------|----------|----------|----------|
| Resonant frequency | 2.40 GHz | 2.40 GHz | 2.40 GHz |
| Return loss (S11) | > 20 dB | ~10 dB | > 15 dB |
| Bandwidth (−10 dB) | ~80 MHz | ~30 MHz | ~70 MHz |
| Gain (broadside) | 5–7 dBi | 3–5 dBi | 4–6 dBi |
| RSSI delta vs stock | +4 to +7 dBm | +1 to +3 dBm | +3 to +5 dBm |

All values are estimates based on standard FR4 patch theory. Actual performance will be measured per the TEST_PROCEDURE.md protocol.

---

## Next Steps After Receipt

1. **Visual inspection** — confirm purple mask, white silk, ENIG finish, MMCX notch clean
2. **VNA measurement** — if available, measure S11 on each board before RF testing. Resonance within ±50 MHz of 2.4 GHz is acceptable without rework
3. **Connector installation** — solder J1 (MMCX 135-3711-801) with paste and reflow or careful iron work. Pads are small — use flux
4. **RF switch** — confirm M5Tab5 GPIO for antenna switching (external vs chip antenna). Check Tab5 schematic for GPIO number controlling the RF switch on the ESP32-C6-MINI-1U
5. **Follow TEST_PROCEDURE.md** — baseline first, then each design in order A → B → C
6. **3D pattern capture** — run the full hemisphere sweep on the winning design, process with the Python pipeline, render on M5Tab5 display via Three.js + BMI270 gyro

---

*Project Platypus — Open Source RF*
*Rev 7.2 — Ready to manufacture*
