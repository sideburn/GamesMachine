# Games Machine

KiCad recreation of **"Games Machine — a programmed random number generator"** by D. Burn,
*Practical Electronics*, December 1976 (pp. 969–975). PCB by Sideburn Studios.

A TTL-logic random number generator with four selectable modes:

| Mode | Range | Zero behaviour |
|---|---|---|
| Random | 0–99 | `00` allowed |
| Roulette | 0–36 | `00` allowed |
| Pools | 1–59 | single zero allowed, `00` blocked |
| Dice | 1–6 per digit | no zero in either digit |

Randomness comes from an avalanche-noise transistor (TR1) feeding a two-stage amplifier
(TR2, TR3), squared by a 7413 Schmitt trigger, and gated into a pair of 7490 decade counters
while a 2N2646 UJT relaxation oscillator (TR4) free-runs at 3–4 s. Because the operator's
press and the oscillator phase are uncorrelated, the stopping point is unpredictable. Game
ranges and the zero-detector restart logic come from a diode-resistor matrix (D5–D20) read
through an interlocked pushbutton bank (S1, S3–S6).

## Status

**Rev 1.0** — fabricated 2026-07-09, built and fully bench-tested. All four game modes verified
working against the behaviour described in the original article. Three defects were found and
worked around on the physical board (see below).

**Rev 2.0** — all fixes applied to the KiCad source, DRC clean (0 violations, 0 unconnected),
gerbers and drill files regenerated. Ready to order; not yet fabricated.

### Fixes applied for rev 2.0

| # | Issue | Resolution |
|---|---|---|
| 1 | `LOGIC_0` was a floating net, never tied to ground — board freezes at `99` | Merged into `GND` |
| 2 | Diode silkscreen band marked the anode, so all 17 diodes fitted backwards to the print | Symbol pin numbers swapped, diodes rotated 180° |
| 3 | UNITS/TENS displays placed in reverse reading order — digits read transposed | Placements exchanged |
| 4 | 2N2926 symbol declared E-B-C; the real part is E-C-B, so TR1–TR3 needed crossed leads | Pin numbers corrected |
| 5 | TR4 (2N2646) footprint mapping suspected wrong | Verified correct as drawn — no change |

Full detail, including what was deliberately left alone, is in `Docs/Rev2_Board_Fixes.md`.

## Notes for anyone building one

- **Identify a UJT's leads by measurement, never from a pinout diagram.** Ohms-mode resistive
  pair = the two bases, the leftover lead = emitter; the can-to-lead short then separates B1
  from B2. Two of the three 2N2646s sourced for this build were counterfeit or dead, and one
  passed a static meter test while being unable to fire under real operating bias.
- **Don't reuse old tantalums for C1, C2 and C4.** Only relevant if you're working from vintage
  stock — this build used period tantalums supplied with the project and all three had gone
  leaky. C1 is the one that hurts: leaky, it holds TR1's emitter down around 1.8 V, far below
  the ~6–9 V its base-emitter junction needs to avalanche, which silences the noise source and
  makes the whole machine look dead for no visible reason.
- TR4 wants rotating roughly 10° clockwise of the silkscreen tab mark when fitting — the TO-18
  leads sit on a circle and don't land dead-on the pad angles.

## Repository contents

- `GamesMachine.kicad_sch` / `.kicad_pcb` / `.kicad_pro` — the KiCad project
- `GamesMachine.kicad_sym` — project-local symbol library (custom parts)
- `GamesMachine.stl` — 3D case/board model
- `Gerbers/` — fabrication output, regenerated for rev 2.0
- `Docs/PartsList.md` — full parts list with sourcing notes
- `Docs/BOM.csv` — machine-readable BOM exported from the schematic
- `Docs/BringUp_TestPlan.md` — staged bring-up and test procedure
- `Docs/GameMode_Jumper_Reference.md` — switch/jumper configuration per game mode
- `Docs/Rev2_Board_Fixes.md` — defect record and how each was resolved

The first commit is the rev 1.0 project exactly as fabricated, so the whole set of fixes can be
read as a diff against the board that was actually built.

## Original source

*Practical Electronics*, Vol. 12 No. 12, December 1976. Article by D. Burn, PhD.
The magazine scan is not redistributed here.
