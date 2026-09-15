# Project Platypus — VNA Test Plan & Milestones

*Bench characterization of the 2.4 GHz patch antennas. Complements `TEST_PROCEDURE.md`
(over-the-air RSSI testing). Written for two instrument tiers:*

- **Tier 1 — NanoVNA at home.** Buy a **NanoVNA-V2 Plus4 / SAA-2N or LiteVNA-64** —
  these measure 2.4 GHz on the fundamental. The original NanoVNA (900 MHz fundamental,
  harmonic mode above) has poor dynamic range at 2.4 GHz; don't buy that one for this.
- **Tier 2 — the good stuff at work.** Any calibrated lab VNA covering 1–3 GHz, and
  whatever radar test infrastructure exists (signal generator + horn, absorber panels,
  possibly a chamber). Tier 2 runs settle the science; Tier 1 is for fast iteration.

## Fixture & calibration rules (apply to every milestone)

1. **Cal plane = the far end of the pigtail.** Calibrate SOL (open/short/load) at the
   SMA/SMP end with the cal standards, then connect the SMP→MMCX pigtail and either
   (a) accept the pigtail inside the measurement (fine — it's part of the deployed
   system), or (b) on the work VNA, use port extension / de-embed the pigtail's delay
   so the reference plane lands at the MMCX. Record which convention every dataset uses.
2. **Antennas are sensitive to their surroundings.** Measure with ≥30 cm clearance to
   metal/bodies: prop the board on a foam block, patch facing up into clear space,
   hands off during sweep. At work, a panel of absorber under/behind is ideal.
3. **Sweep setup:** 2.0–3.0 GHz, ≥201 points for survey; 2.35–2.55 GHz narrow for
   resonance metrology. 915 MHz work later: 0.85–0.98 GHz.
4. **Data hygiene:** save Touchstone `.s1p` for every sweep. Naming:
   `s11_<rev>_<design><board#>_<bare|mask>_<instrument>_<date>.s1p`, e.g.
   `s11_r7.13_C1_mask_litevna_2026-08-09.s1p`. Keep them in a `measurements/` folder
   (add to the repo only for the released revision). Log temperature and fixture notes.

---

## Milestone 0 — Instrument sanity check

Before trusting anything: cal, then re-measure the load standard (should sit at centre
of Smith chart, |S11| < −30 dB across band), an open pigtail (|S11| ≈ 0 dB, phase
spinning), and repeat one antenna sweep twice with a re-mate in between.

**Gate:** load < −30 dB; two re-mated sweeps of the same antenna agree on f₀ within
±5 MHz and on |S11| depth within 2 dB. If not, fix the fixture before proceeding.

## Milestone 1 — Rev 7.13 A/B/C: does the antenna resonate where the math says?

Sweep all three manufactured boards (masked patches, edge-launch MMCX). Extract per
board: f₀ (minimum |S11|), return loss at f₀, −10 dB bandwidth, and the impedance at
f₀ from the Smith chart.

**Predictions to test:**
| Board | Predicted | Pass band |
|---|---|---|
| A (inset, matched) | f₀ ≈ 2.40 GHz minus mask detune (expect ~2.37–2.39), RL > 15 dB | f₀ within 2.35–2.45 GHz |
| B (inset, mismatch) | same f₀, RL ≈ 7–10 dB | RL clearly worse than A |
| C (λ/4 transformer) | same f₀, RL > 12 dB | f₀ within 2.35–2.45 GHz |

**Gate:** all three resonate in-band. If a board shows *no* resonance dip at all →
connector/feed fault (this is the definitive version of the patch-A question from field
testing — a dead joint shows a near-flat |S11| ≈ 0 dB trace).

## Milestone 2 — The edge-resistance experiment (the real science)

The one soft input in the whole design chain is Rin_edge (assumed 200 Ω; Balanis says
~300–400 Ω). Design B exists to measure it:

1. Read R + jX at f₀ for boards A and B from the Smith chart.
2. Rin_edge = R_measured / cos²(π·y₀/L) with y₀ = 9.81 mm (A) and 7.50 mm (B),
   L = 29.44 mm. Two boards → two independent estimates; they should agree.

**Gate/outcome:** a consistent Rin_edge number. Write it into the design notes — it
recalibrates the inset formula for every future design (including 915 MHz: y₀ scales
directly). This single measurement retires the biggest open assumption in the project.

## Milestone 3 — Mask detune quantification (needs Rev 8/9 boards)

When the Rev 9 panel arrives: identical designs exist bare (column 1) and masked
(columns 2–3). Sweep one bare + one masked copy of the same design, same fixture.

**Deliverable:** Δf₀ in MHz (prediction: mask pulls resonance down 10–30 MHz). This
number decides whether future revisions pre-compensate patch length for masked builds,
and it's a great plot for the Hackster/Tindie story.

## Milestone 4 — Rev 9 connector-land comparison

Same panel: sweep a u.FL-fitted board vs an MMCX-fitted board of the same design, and
compare both against a Rev 7.13 edge-launch board. This measures the 0.6 mm neck +
unpopulated-land stub discontinuity.

**Gate:** RL at f₀ within ~2 dB and f₀ within ~10 MHz across connector variants → the
SMT lands are RF-transparent enough, ship whichever connector the market wants. A big
deviation on one variant → look at that land geometry before productizing it.

## Milestone 5 — Radiation pattern & gain (work facility)

The VNA gives matching, not gain — this milestone needs the radar bench: signal source
+ standard-gain horn (or any characterized 2.4 GHz antenna) at a few metres in the
clearest space available, patch on a rotator or hand-stepped in 15° increments.

1. **Gain by comparison:** received power through the patch vs through a reference
   antenna of known gain, same position/polarization → G_patch = G_ref + ΔP.
   Prediction: 4–6 dBi realized (5–7 dBi directivity minus FR4 loss).
2. **Azimuth + elevation cuts:** normalized pattern, front-to-back ratio.
   Prediction: F/B only ~6–10 dB (small ground plane — this measurement closes out the
   "no difference at 180°" field observation with real numbers).
3. **Polarization check:** rotate the patch 90° about boresight — cross-pol should drop
   >15 dB. Confirms the linear polarization axis for mounting guidance.

**Deliverable:** pattern plots + an absolute realized-gain figure (dBi) for the product
page — the characterisation the project does not yet have.

## Milestone 6 — (later) 915 MHz bracket-panel selection

When the Meshtastic bracket panel exists: sweep variants A/B/C (L = 77.09/78.67/
80.24 mm), pick whichever lands closest to 906.875 MHz, and back out the batch's true
εr from the winner. Five-minute job with the NanoVNA that replaces a whole field-test
campaign — this is the milestone that justifies the instrument purchase by itself.

---

## Decision map (what each milestone unblocks)

| Milestone | Decision it feeds |
|---|---|
| M1 | Is patch A's connector joint dead? (reflow or exonerate) |
| M2 | Corrected Rin_edge → 915 MHz inset depth, all future insets |
| M3 | Pre-compensate L for masked builds? Bare-patch SKU story |
| M4 | Ship u.FL, MMCX, or both — with data |
| M5 | Absolute gain + patterns for product listing & Hackster |
| M6 | 915 production geometry + batch εr |
