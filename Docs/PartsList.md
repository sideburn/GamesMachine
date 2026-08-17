# Games Machine — parts list

Generated from the rev 2.0 schematic. Quantities are for one board.

Resistors are ¼ W carbon film throughout; the footprints are 10.16 mm pitch axial.

## On-board parts

### Resistors

| Refs | Value | Qty | Function |
|---|---|---|---|
| R1 | 47 kΩ | 1 | Noise generator, TR1 emitter feed |
| R2 | 4.7 kΩ | 1 | TR2 collector load |
| R3 | 100 kΩ | 1 | TR3 collector-base bias |
| R4, R5 | 2.2 kΩ | 2 | TR3 collector load, noise output |
| R6 | 150 kΩ | 1 | UJT clock timing (with C4) |
| R7 | 220 Ω | 1 | UJT clock output |
| R8, R9, R10 | 4.7 kΩ | 3 | Open-collector pull-ups for IC11 |
| R11–R24 | 270 Ω | 14 | Display segment current limiting |
| R25 | 1 kΩ | 1 | `LOGIC_1` pull-up |
| R26–R30 | 1 kΩ | 5 | Game-mode decode pull-ups |

### Capacitors

| Refs | Value | Qty | Notes |
|---|---|---|---|
| C1, C2, C4 | 10 µF | 3 | Article specifies 16 V tantalum; modern electrolytics are fine and more reliable. Polarity: **+ toward the higher-voltage node** |
| C3 | 0.1 µF | 1 | Ceramic disc |

### Semiconductors

| Refs | Part | Qty | Notes |
|---|---|---|---|
| TR1–TR3 | 2N2926 | 3 | Noise generator. Real pinout is **E-C-B** |
| TR4 | 2N2646 | 1 | UJT clock oscillator. Article also allows TIS43 |
| D5–D20 | 1N914 | 16 | Game-mode decode matrix |
| Z1 | 1N4733A (5.1 V) | 1 | Rail clamp |
| IC1 | 7413 | 1 | Dual 4-input Schmitt NAND — squares the noise, gates it |
| IC2 | 7472 | 1 | JK flip-flop — start/stop bistable |
| IC3, IC4 | 7490 | 2 | Decade counters (tens, units) |
| IC5, IC6 | 7447 | 2 | BCD to 7-segment decoders |
| IC9 | 7425 | 1 | Dual 4-input NOR — zero detector |
| IC10 | 7480 | 1 | Gated full adder, used as switchable AND/OR |
| IC11 | 7412 | 1 | Triple 3-input open-collector NAND — restart logic |

### Connectors and other

| Refs | Part | Qty | Notes |
|---|---|---|---|
| PWR1 | JST PH 3-pin vertical | 1 | +17 V, +5 V, GND from the supply |
| S1 | JST PH 2-pin vertical | 1 | Play button |
| S3a–S6d | JST PH 3-pin vertical | 11 | Game-mode switch bank, one per pole |
| F1 | 0.5 A resettable PTC | 1 | Radial, 5 mm |
| TENS1, UNITS1 | 14-pin DIP socket | 2 | The DL707 displays plug in here |

IC sockets are strongly recommended for all of IC1–IC11 — most of these are obsolete parts and
you'll want to be able to swap them while testing.

## Off-board parts

| Part | Qty | Notes |
|---|---|---|
| DL707 7-segment display, common anode | 2 | 0.3 in red. Plug into TENS1/UNITS1 |
| Interlocked pushbutton bank | 1 | 5 latching buttons plus one Play. Article used a Doram 8-switch frame with latching bar and return spring; pressing one button releases the others |
| Power supply | 1 | +17 V and +5 V. See below |

### Power

The article's own supply (its Fig. 3) is a mains transformer, bridge rectifier, 2200 µF filter
and a µA7805 regulator, giving ~17 V unregulated plus a regulated 5 V rail. None of that lives
on this board — it arrives through PWR1.

The simplest modern equivalent is a single regulated **18 V, 1 A universal-input (100–240 V)
switching adapter**, with the 5 V rail derived from it. Typical measured draw is around 120 mA
on the 5 V rail with everything populated.

## Sourcing notes

Most of the logic is obsolete but still easy to find from surplus dealers. Two parts deserve
real caution.

**2N2646 (TR4)** — this is the part most likely to waste your time. Of three obtained for this
build, one was a relabelled PNP bipolar (not a UJT at all), one was genuine but so degraded by
age that it passed every static meter test while being unable to fire under real operating bias,
and one was good. Buy from a dealer with verifiable vintage stock, buy spares, and **test each
one before fitting**:

1. Ohms mode: the two leads showing a resistive 4.7–9 kΩ both ways are the bases; the leftover
   lead is the emitter.
2. The base that reads ~0 Ω to the metal can is B2.
3. Diode mode from the emitter: a normal forward drop into each base, open in reverse.

A genuine part that passes this can still be too aged to oscillate, so the real test is fitting
it and looking for the sawtooth at the emitter. **2N6027** programmable UJTs are still in
production and can substitute, but need a gate-bias divider added and R6 changed to ~330 kΩ.

**2N2926 (TR1–TR3)** — real pinout is **E-C-B**, which trips up a lot of pinout diagrams online.
Rev 2.0 of this board is drawn for the correct order, so parts drop straight in.

**Vintage electrolytics** — if you're building from an old kit, replace C1, C2 and C4 rather
than trusting them. All three were leaky in this build. C1 is the nastiest: it sits across
TR1's emitter bias node, and when leaky it holds that node down around 1.8 V, far below the
6–9 V the base-emitter junction needs to avalanche. The noise generator then produces nothing
and the machine looks completely dead, with no obvious clue as to why.
