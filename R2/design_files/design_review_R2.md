# GPS/GSM Tracker (tracker.kicad_pro) — Design Review, Board Rev R2

**Date:** 2026-09-28
**Analysis run:** `analysis/2026-09-28_2106/` (schematic, PCB, cross-domain, EMC, thermal)

## Issues

| # | Severity | Issue | Detail |
|---|----------|-------|--------|
| 1 | BLOCKER | Sourcing gate fails (SS-001) | Only 2/94 components (D1, L3) carry an MPN; no `datasheets/` directory exists. Every pin-level/electrical claim elsewhere is consistency-only, not datasheet-verified. Must reach ≥50% MPN coverage before ordering. |
| 2 | WARNING | R1 known issue never fixed: U2 flash footprint too big | U2 (GD25QxxxEY / GD25Q256EYIGR) uses `Package_SON:WSON-8-1EP_8x6mm_P1.27mm_EP3.4x4.3mm` — byte-for-byte identical to R1. The designer's own R1 README flagged this as wrong; it was not addressed in R2. |
| 3 | WARNING | R1 known issue never fixed: programming port still 2.0mm pitch | J3 is still `Connector_JST:JST_PH_B5B-PH-SM4-TB_1x05-1MP_P2.00mm_Vertical` in both R1 and R2. README asked for 1.5mm pitch. |
| 4 | WARNING | Battery connector polarity swapped vs. R1 — needs physical check | R1: J6 pin1=`+BAT`, pin2=`GND`. R2: pin1=`GND`, pin2=`+BAT` (deliberate swap, not a mirror side-effect). Correct polarity depends on the physical battery pack's own wiring convention, which isn't derivable from the schematic — confirm against the harness before fab. |
| 5 | WARNING | Q1 has zero thermal vias | Reverse-polarity/load-switch MOSFET (Q1, PowerDI3333-8) has 0 of 5 recommended thermal-pad vias. Will run hotter than necessary under sustained load current. |
| 6 | WARNING | Insufficient thermal vias on U1/U2 | U1 (MCU, QFN-48): 8/15 recommended. U2 (flash, WSON-8): 4/9 recommended. Low urgency at the board's modeled 0.067W dissipation, but cheap to fix now. |
| 7 | WARNING | No fiducials on either SMD side (FD-001) | Both F.Cu (50 SMD parts) and B.Cu (42 SMD parts) carry fine-pitch parts (U1 QFN-48 @0.5mm pitch, U12 LGA-12 @0.5mm pitch) with zero fiducials. Add ≥3 per side before ordering assembly. |
| 8 | WARNING | J5 (SIM socket) has no ESD protection | Genuinely user-exposed, hot-pluggable connector with no TVS array on CLK/IO/RST/VCC. Also flagged at a 5:1 signal-to-ground pin ratio (only 1 GND reference for 5 signal/power pins). |
| 9 | WARNING | U11 PCB value field wrong (XV-002) | Schematic value is correctly "BQ25622E"; PCB footprint's value property reads "Untitled_1". Risks wrong-part confusion if a BOM is ever generated from PCB properties. |
| 10 | WARNING | Keepout zones sit under U1 and U12 | Two "no copper pour" zones: one (62.0mm², F.Cu) spans U1 (MCU) and its crystal load-cap/pull-up network (C1–C4, R18–R20); another (6.25mm², B.Cu) spans U12 (accelerometer) and nearby passives (C8/C9/C31/C42, L3, R5/R6, U10). Excludes ground pour from directly under the MCU's decoupling/crystal network unless intentional (e.g. RF/antenna keepout) — confirm design intent before fab. |
| 11 | WARNING | Board-edge clearance below safe minimum | R1 (0.28mm) and D1 (0.31mm) are at/below the ~0.3mm typical safe minimum. |
| 12 | WARNING | EMC: switching harmonics in VHF bands (SW-001) | U8 (LMR51430, 500kHz) harmonics land in the 30–88MHz and 88–216MHz VHF bands. Direct overlap with GNSS L1 (1575.42MHz) or GSM/LTE receive bands is unlikely, but low-VHF radiation from board edges is possible. |
| 13 | WARNING | EMC: filter cutoff too close to switching frequency (EF-001) | Reported for both U8 and U9 — filter component values should be revisited against each regulator's actual switching frequency (500kHz is `freq_source=topology_default`, i.e. assumed, not datasheet-confirmed). |
| 14 | WARNING | EMC: large hot loop on U9 (SW-003) | Switch-node loop area for U9 flagged as large — worth a layout pass to tighten it. |
| 15 | WARNING | EMC: L3 proximity to RTC crystal (ML-001) | L3 (assumed unshielded) sits 13.6mm from Y1 (32.768kHz crystal) — plausible minor coupling risk into the RTC crystal at low urgency given the distance and low switching current. |
| 16 | WARNING | EMC: split ground/power plane stackup (SU-001, GP-001) | All 4 copper layers are typed `signal` rather than having a dedicated GND/PWR plane pair; GND is pour-filled per layer but shared/split with +3V3/+VSYS/+5V zones rather than a continuous reference plane. Return-path continuity is not guaranteed as clean as a classic signal/GND/PWR/signal stackup. Stitching (458 vias present) mitigates this but isn't independently confirmed under the OCTOSPI bus or GSM UART traces. |
| 17 | WARNING | U9 regulator Vref is a heuristic guess, not verified | AP61100Z6 is not in the ~60-family Vref lookup table; the assumed 0.6V reference (giving est. 3.3V output via R13/R14) is plausible but unconfirmed — a wrong assumption here would produce a wrong core rail. |
| 18 | WARNING | BG96 (M1) thermal/current behavior is unmodeled | Classified `type=other` by the analyzer, so it's excluded from every power/thermal model. BG96 GSM transmit bursts are the largest realistic heat/current-transient source on the board and are invisible to this review's tooling — manual check against Quectel's hardware design guide is required. |
| 19 | SUGGESTION | Zero test points (TE-001) | 0 of 149 nets have a dedicated test point. Add test points on key rails (+VBAT, +VSYS, +3V3, +5V, GND) for bring-up/production test. |
| 20 | SUGGESTION | EN pull-ups unusually high (PU-001) | R8/R12 (200kΩ) pull-ups on U8/U9 EN pins are high-impedance — plausible intentional choice for sleep-current minimization, but confirm against each regulator's EN pin leakage spec once datasheets are available. |
| 21 | SUGGESTION | J3/J4 have no ESD protection | Lower risk than J5 (J3 is a board-internal debug header; J4 is an RF path where a TVS would degrade matching), but worth a conscious decision either way. |
| 22 | SUGGESTION | Crystal load caps not verified against spec | C1–C4 (18pF) not checked against Y1/Y3's specific CL requirement — no MPN/datasheet available for either crystal. |

## Not Performed / Review Limits

- Datasheet sync/extraction: not performed (no `datasheets/` directory, no MPNs beyond D1/L3) — every pin-level claim above is consistency-only, not datasheet-verified.
- SPICE simulation: not performed (no ngspice/ltspice/xyce installed).
- Lifecycle audit: not performed (no network/API credentials, and MPN coverage too low to be useful).
- Gerber/drill analysis: not applicable (no gerber export in the project).
