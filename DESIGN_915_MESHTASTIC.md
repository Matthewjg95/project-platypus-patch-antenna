# Project Platypus — 915 MHz Meshtastic/LoRa Patch (design study)

*Started 2026-07-05. First-principles design, same method as the proven 2.4 GHz panel.*

## Target

Directional gain antenna for Meshtastic / LoRa point-to-point links, US 902–928 MHz band.
Design frequency: **906.875 MHz** (Meshtastic US LongFast default primary), not 915 —
the patch is narrowband, so tune to where the traffic actually is. An EU868 variant
(869.525 MHz) is a later length-scale respin.

## First-principles dimensions (FR4, h = 1.6 mm, εr = 4.4, tan δ = 0.02)

```
f0 = 906.875 MHz            λ0 = 330.7 mm
Patch width W    = 100.66 mm
εr_eff           = 4.258
ΔL (fringing)    = 0.745 mm
Patch length L   = 78.67 mm
50 Ω feed width  = 3.06 mm   (Hammerstad-Jensen; same as 2.4 GHz design)
100 Ω xfmr width = 0.709 mm ; λg/4 = 47.6 mm  (transformer variant — long, see below)
Inset (200 Ω edge assumption):  y0 = 26.22 mm
Inset (Balanis edge ~376 Ω):    y0 = 29.99 mm
Bandwidth (VSWR<2): ~0.41 % ≈ 3.7 MHz
```

**Feed choice: inset feed.** The λ/4 transformer costs 47.6 mm of board height at this
frequency; the inset is free. (The 2.4 GHz A-vs-C test showed matching method makes <1 dB
difference — pick the compact one.)

## THE design risk at 915: FR4 εr tolerance vs. 3.7 MHz bandwidth

f0 sensitivity at fixed geometry (designed for εr = 4.4):

| FR4 εr actual | f0 lands at | shift |
|---|---|---|
| 4.2 | 927.9 MHz | +21 MHz |
| 4.4 | 906.9 MHz | 0 |
| 4.6 | 887.2 MHz | −20 MHz |

A ±0.2 εr batch spread moves resonance **±20 MHz against a 3.7 MHz bandwidth** — a 5×
overshoot. At 2.4 GHz we got away with this (BW ~25 MHz, same fractional shift); at 915
we will not. Mitigation, in the proven Project Platypus style — **bracket panel**:

| Variant | L | f0 if εr=4.4 | covers εr ≈ |
|---|---|---|---|
| A (short) | 77.09 mm (−2%) | 925.0 MHz | ~4.2 |
| B (nominal) | 78.67 mm | 906.9 MHz | ~4.4 |
| C (long) | 80.24 mm (+2%) | 889.4 MHz | ~4.6 |

One panel run tells us the batch's real εr; the winning variant becomes the product
geometry for that fab. Selection can be done with Meshtastic's own RSSI/SNR reports
between two nodes; a NanoVNA (on the roadmap) makes it a 5-minute S11 sweep instead.

## Board / panel

- Patch 100.7 × 78.7 mm → board ~**130 × 115 mm** (12–15 mm ground margin + feed run).
  Margins are only ~0.04 λ — front-to-back will be modest (~6–10 dB), same physics as
  the 2.4 GHz board. Fine for "point the gain at the far node"; don't market F/B.
- 3 variants stack into a 1×3 V-scored panel ~130 × 345 mm, or 2×3 (~260 × 345) for
  bracket + spare. Both exceed JLCPCB's 70 mm V-cut minimum in both axes. V-cuts on
  Edge.Cuts (GM1), keep-all-islands ground — all per the Rev 8 playbook.
- Solder mask: **openings over the patch** (bare ENIG radiator) — carry the Rev 8 fix.
  At 0.4 % bandwidth we cannot afford unmodelled mask detune.
- Mounting: keep the M3 hole pattern idea, sized for a mast clamp / zip ties instead of
  the M5Tab5 pattern.

## Connector (product-driven, differs from the Tab5 board)

- **On-board: u.FL/IPEX jack** (vertical SMT, ~$0.40). It is the de facto standard on
  Heltec/RAK/LILYGO Meshtastic nodes, needs no routed notch, no edge-launch DRC waivers,
  no spring-pin ambiguity — the feed simply ends in a pad, grounds via to the plane.
- **Ship with a u.FL → SMA-male pigtail** so it also screws straight onto boxed nodes
  (SMA female bulkhead), covering both halves of the market with one SKU.
- Optional second footprint position for a vertical SMT MMCX jack (e.g. Cinch
  135-3701-201) sharing the feed end — populate either. Decide after the 2.4 GHz
  connector experiments.

## Roadmap / open items

1. Freeze bracket-panel geometry; build the KiCad board (new project dir, not in the
   2.4 GHz tree).
2. **NanoVNA** purchase → measure S11 of 2.4 GHz Rev 7.13 A/B/C first (validates the
   whole design chain and the 200 Ω edge-resistance assumption), then use for 915
   variant selection.
3. EU868 respin (scale L for 869.525 MHz) once US geometry is proven.
4. Field range test: two Meshtastic nodes, omni-vs-patch A/B, logged SNR vs distance —
   that's also the marketing material.
