# Milestones

## 2026-09-04 — Tindie × NextPCB R&D Support application approved

Application status: **Approved**

The project has been accepted into the Tindie × NextPCB Hardware Creator R&D
Support Program.

### Support awarded

Three monthly support rounds are scheduled for September 5, October 5, and
November 5, 2026. Each round provides:

- **$30 PCB prototyping voucher** for a qualifying standalone PCB manufacturing
  order.
- **$20 international shipping discount code**.
- Each voucher/code is valid for three months from issuance.

Program rules state that the PCB voucher applies to standalone PCB fabrication
orders with no substrate-material or fabrication-spec restriction and no
minimum spend. It does **not** cover PCB cost embedded inside a PCBA order;
assembly and component procurement therefore remain separate project costs.

### Planned allocation

The support is treated as an engineering-iteration budget rather than an
obligation to spin the antenna three times:

1. **September — Rev 2 validation run.** Freeze the audited KiCad package,
   connector implementation, mounting geometry, labeling, and DFM details;
   manufacture the A/B/C comparison panel.
2. **October — evidence-gated Rev 2.1.** Order another antenna revision only if
   controlled RF measurements identify a specific change worth testing.
   Otherwise redirect the fabrication round to a Platypus One carrier/interface
   board.
3. **November — product candidate or Platypus One carrier.** If antenna
   validation closes early, use the final fabrication round to accelerate
   Platypus One hardware development rather than revising a solved antenna.

**Revision rule:** a new antenna spin must be justified by measurement. Coupon
availability alone is not a design-change requirement.

This creates a funded path from Rev 2 through measured validation while
preserving later program capacity for the broader Platypus hardware stack if
the antenna converges early.

### Next gate

Before the September fabrication order:

- complete KiCad source audit;
- run DRC and HQDFM/DFM review;
- verify panel geometry and connector fit;
- confirm Rev 2 fabrication outputs;
- define the exact before/after RF test matrix.

The sales listing remains gated on repeatable RF results, BOM/assembly
instructions, photographs, honest specifications, and pilot-readiness evidence.

---

## 2026-08-30 — Tindie × NextPCB R&D Support application submitted

Application status: **Superseded by approval on 2026-09-04**

Project submitted as:

- **Product:** Project Platypus 2.4 GHz Patch Antenna — Rev 2
- **Tindie status:** Store opened, no eligible orders yet
- **Development stage:** Functional prototype available and ready for optimization
- **Expected 12-month PCB quantity:** 50–200
- **Related product listing:** Not yet published

This was the project's first formal step from a documented open-source RF
prototype toward a small-batch hobbyist product.

### Application development plan

1. Audit the existing KiCad project and released manufacturing files.
2. Define Rev 2 requirements, including connector/cable compatibility,
   labeling, mounting, and manufacturability.
3. Run DRC and DFM review before ordering.
4. Produce 10–20 Rev 2 validation boards.
5. Compare Rev 2 variants against the manufactured revision using the
   repository test procedure and the Project Platypus Antenna Lab workflow.
6. Publish measured, reproducible results without relying on theoretical gain
   claims.
7. Release a 25–50 unit pilot batch on Tindie after the validation gate passes.
8. Reorder in demand-based batches, targeting 50–200 PCBs over 12 months.

### Product-release gate

Approval does **not** mean Rev 2 is production-ready. A public sales listing
waits until all of the following are complete:

- KiCad source and fabrication-output review.
- DRC/DFM acceptance.
- Mechanical and connector-fit verification.
- Repeatable RF comparison against the current boards and host antenna.
- Practical connectivity testing.
- BOM, assembly instructions, photographs, and honest specifications.
- Pilot pricing and fulfillment plan.

Program page:
https://www.tindie.com/campaigns/nextpcb-2026/
