# Games Machine

KiCad recreation of **"Games Machine — a programmed random number generator"** by D. Burn,
*Practical Electronics*, December 1976 (pp. 969–975). PCB by Sideburn Studios, rev 1.0,
fabricated 2026-07-09.

A TTL-logic random number generator with four selectable modes:

| Mode | Range |
|---|---|
| Random | 0–99 |
| Roulette | 0–36 |
| Pools | 1–59 |
| Dice | 1–6 per digit |

Randomness comes from an avalanche-noise transistor (TR1) feeding a two-stage amplifier
(TR2, TR3), squared by a Schmitt trigger, and sampled while a UJT relaxation oscillator
(TR4) clocks a pair of 7490 decade counters. Game-mode selection and the zero-detector
restart logic are built from a diode-resistor matrix (D5–D20) read by an interlocked
pushbutton switch bank (S1, S3–S6).

## Status

Rev 1.0 board fully bring-up tested and confirmed functional across all four game modes —
see `Docs/BringUp_TestPlan.md` and `Docs/GameMode_Jumper_Reference.md` for the full
procedure and results.

Known rev 1.0 defects and their fixes are tracked in `Docs/Rev2_Board_Fixes.md` and are
being applied here commit-by-commit ahead of a rev 2.0 board order.

## Repository contents

- `GamesMachine.kicad_sch` / `.kicad_pcb` / `.kicad_pro` — the KiCad project
- `GamesMachine.kicad_sym` — project-local symbol library (custom parts)
- `GamesMachine.stl` — 3D case/board render
- `Gerbers/` — fabrication output from the rev 1.0 order

## Original source

*Practical Electronics*, Vol. 12 No. 12, December 1976. Article by D. Burn, PhD.
