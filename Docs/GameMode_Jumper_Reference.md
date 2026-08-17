# Games Machine — Game Mode Jumper Reference

Bench-test jumper configurations for each game mode, confirmed working 2026-08-17. These
simulate the real interlocked pushbutton switches (S3-S6) until the physical switch loom is
wired in.

General pattern for every S3/S4/S5/S6 pole: pin 1 = input from previous switch in the chain,
pin 2 = "released" (pass-through to next switch), pin 3 = "pressed" (this mode's own decode
network or ground). Only one mode's switch should be "pressed" (1&3) at a time — everything
else stays in pass-through (1&2) or fully open, matching the real hardware's mechanical
interlock (pressing one button releases whichever was previously pressed).

S1 (Play) is a separate 2-pole momentary button, not part of this chain — pressed = pin 1&2
connected.

**On "all others open" below:** with real physical switches, no pole is ever truly
disconnected — every unpressed switch mechanically rests at 1&2 (released), every pressed
one sits at 1&3. "Open" in the tables below is a bench-jumper shortcut for switches outside
the active chain for that mode; jumpering them to 1&2 instead is equally correct and in fact
more accurately mirrors how the finished hardware behaves.

---

## Random — range 0–99

| Switch | Position |
|---|---|
| S3a, S3b | 1&3 closed |
| S5c, S6d | 1&2 closed |

All others open.

## Roulette — range 0–36

| Switch | Position |
|---|---|
| S3a, S3b | 1&2 closed |
| S4a, S4b | 1&3 closed |
| S5c | 1&2 closed |
| S6d | 1&2 closed |

All others open.

## Pools — range 1–59

| Switch | Position |
|---|---|
| S3a, S3b | 1&2 |
| S4a, S4b | 1&2 |
| S5a, S5b, S5c | 1&3 |
| S6c | 1&2 |

All others open.

## Dice — 1–6 each digit

| Switch | Position |
|---|---|
| S3a, S3b | 1&2 |
| S4a, S4b | 1&2 |
| S5a, S5b, S5c | 1&2 |
| S6a, S6b, S6c, S6d | 1&3 |

---

## Zero-digit behavior by mode

- **Random**: 00 fully allowed (confirmed stable, non-blanking stop).
- **Roulette**: 00 fully allowed (per article: "a selection of 00 is perfectly acceptable").
- **Pools**: single zero in either digit allowed (e.g. 02, 20); double-zero (00) blocked,
  triggers blank-and-respin.
- **Dice**: no zero allowed in either digit — cannot stop while either digit shows 0.

Source: original article, *Practical Electronics*, Dec 1976, "Random Number Gamesmachine"
by D. Burn, p.972 (Zero Detector section) and p.971-972 (circuit description).
