# Project Platypus — Status & Transfer Guide

*Last updated 2026-09-14. This file exists so the whole project moves between machines
with one `git clone` and no tribal knowledge. Everything below is in this repo.*

## What this project is

First-principles 2.4 GHz microstrip patch antennas (KiCad, FR4 1.6 mm), plus a product
track (Tindie) and a 915 MHz Meshtastic design study. Full derivations in
`DESIGN_NOTES_v72.md` and the README.

**Measurement status:** the boards are manufactured and functional. An early RSSI
delta vs the host chip antenna was anomalously high and has been withdrawn pending a
controlled re-test — do not cite it. Bench characterisation plan: `VNA_TEST_PLAN.md`.

## Revision map

| Revision | File(s) | State |
|---|---|---|
| **Rev 7.13.1** | `patch_antenna_smp.kicad_pcb` (+`.kicad_pro`, `.kicad_dru`), `gerbers/`, `gerbers.zip`, `DRC7.13.1.rpt` | **Manufactured.** 1×3 panel, edge-launch MMCX. Never edit in place. |
| **Rev 8** | `patch_antenna_smp_rev8.*`, `REV8_PLAN.md`, snapshots in `rev8_backups/` | Complete + DRC-verified. 3×3 panel 174×210 mm, V-cuts on Edge.Cuts, keep-all-islands ground, bigger MMCX pads, mask experiment (left column bare / two masked). Superseded by Rev 9 for ordering. |
| **Rev 9** | `patch_antenna_smp_rev9.*`, `REV9_CONNECTORS.md`, snapshots in `rev9_backups/`, fab package `rev9_fab/` + `rev9_gerbers_jlcpcb.zip` | Complete + DRC-verified. Dual SMT lands per board: Hirose U.FL (JUA–JUI) + Würth vertical MMCX (JMA–JMI), no notches. **This is the revision to order.** |
| **915 MHz** | `DESIGN_915_MESHTASTIC.md` | Design study done, KiCad board not started. Key: 3-length bracket panel (FR4 εr tolerance vs 3.7 MHz bandwidth). |

## Pending / open decisions

1. **Rev 9 fabrication order** — package is upload-ready. Funding (see
   `MILESTONES.md`): the Tindie × NextPCB vouchers ($30/mo ×3 + shipping) cover
   **standalone PCB fabrication at NextPCB only — not PCBA**; the separate $20
   JLCPCB credit can offset a JLCPCB assembly order. Options: (a) NextPCB
   bare-panel run on the voucher + hand-solder connectors, or (b) JLCPCB PCBA —
   u.FL from their library (Hirose **C88373**), Würth MMCX via Global Sourcing
   (~$4–5/pc, +1 wk) or hand-soldered. Population map in `rev9_fab/BOM_rev9.csv`
   / `CPL_rev9.csv` (MMCX cols 1&3, u.FL col 2). If using JLC assembly, check
   connector rotations in their preview (u.FLs are at −90°). Per MILESTONES.md,
   any new spin beyond this must be justified by measurement, not coupon availability.
2. **VNA milestones** (`VNA_TEST_PLAN.md`): buy a NanoVNA-V2 Plus4 / LiteVNA-64 (NOT
   the original NanoVNA); M1–M2 on the Rev 7.13 boards settle the patch-A connector
   question and the Rin_edge assumption. M5 (gain/pattern) uses the work radar bench —
   this produces the number that replaces the withdrawn claim.
3. **915 bracket panel**: start KiCad board after the Rev 9 order is in flight.

## Toolchain & workflow (as used to build Rev 8/9)

- **KiCad 10.0.1.** GUI plus headless: `kicad-cli` and the bundled Python
  (`C:\Program Files\KiCad\10.0\bin\python.exe`, module `pcbnew`) for scripted edits.
- **Validation is a DRC delta-check**: `kicad-cli pcb drc` does NOT apply GUI
  exclusions, so compare violation *categories* against the documented expected set
  (see `REV9_CONNECTORS.md` § verified DRC state), not against zero. It DOES honor
  `.kicad_dru` custom rules.
- Always **Fill All Zones (B) in the GUI before export** after any zone edit.
- pcbnew scripting gotchas (hard-won): `LoadBoard` twice in one process returns a raw
  SwigPyObject — run phases as separate processes; removing tracks in a loop can
  segfault — prefer add-only edits; `Duplicate()` needs `.Cast()` and loses nothing
  but check nets anyway.
- Windows: ignore SWIG "memory leak" stderr noise from pcbnew scripts.

## Moving to a new machine

```
git clone https://github.com/Matthewjg95/project-platypus-patch-antenna.git
```

Then: install KiCad ≥10, `gh auth login`, set git identity, and open
`patch_antenna_smp_rev9.kicad_pcb`. The `rev8_backups/`/`rev9_backups/` folders are
staged design snapshots (rollback points); git history is authoritative. Not in the
repo (by design): `.history/` editor cache, `*.kicad_prl` window state, vendor
datasheet PDFs (links in `BOM/Links.txt`).

## Product / licensing

CERN-OHL-S v2 (`LICENSE`). Commercial sale is permitted by the license, and as sole
copyright holder you may also dual-license. Tindie plan: bare / assembled / kit SKUs;
market via Meshtastic + ELRS communities once characterisation data exists.
