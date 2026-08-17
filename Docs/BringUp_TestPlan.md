# Games Machine — Board Bring-Up & Sectional Test Plan

Random Numbers Game Machine, Sideburn Studios
Source design: "Games Machine — a programmed random number generator", D. Burn,
*Practical Electronics*, December 1976, pp. 969–975.

Published reference copy: https://claude.ai/code/artifact/c53d6dda-828d-4b43-869a-0b2889a355c7

---

> ## ⚠ Check your board revision first
>
> This plan was written against **rev 1.0**. The two workarounds under "Before you assemble"
> apply **only to rev 1.0** — both defects are fixed in the rev 2.0 design.
>
> **On a rev 2.0 board, skip findings 1 and 2 entirely:**
> - Do **not** fit the `LOGIC_0` → ground bodge wire. That net is part of `GND` now.
> - Do **not** fit the diodes opposite the silkscreen. The band marks the **cathode** correctly
>   on rev 2.0, so "backwards to the print" would put all 17 diodes in the wrong way round.
>
> Everything from "Staged bring-up" onward applies to both revisions — the circuit itself is
> unchanged. The schematic title block gives the revision.

---

## Before you assemble — three findings *(rev 1.0 only — see warning above)*

### 1. CRITICAL — `LOGIC_0` is never connected to ground *(rev 1.0 only)*

`GND` and `LOGIC_0` are two separate nets on this board. In the 1976 schematic the "0" symbol
simply *is* 0 V, but here `LOGIC_0` was drawn as its own label and never tied down. It has
38 routed track segments and 12 pads, and **no source of any kind** — no connector pin, no
resistor, no link to the ground pour.

Pads on `LOGIC_0`: IC2 pins 3, 4, 5 (J1–J3) · IC3 pins 6, 7 · IC4 pins 6, 7 (set-to-nine) ·
S3a pin 3 · S3b pin 3 · S5a pin 3 · S6c pin 2 · S6d pin 2.

Floating TTL inputs read as logic 1, so as built:

- Both 7490s permanently forced to nine → frozen `99` on the display, never counts.
- IC2 J-inputs high → the bistable toggles instead of resetting; start/stop unreliable.
- Random and pools switch positions cannot release the counter resets.

**Fix:** one short insulated wire on the solder side, **IC3 pin 7 → IC3 pin 10**. That is one
`LOGIC_0` pad to one `GND` pad on the same package. Afterwards confirm continuity at a far
point such as IC2 pin 3. Correct the schematic before any future board order.

### 2. Diode silkscreen band is on the anode end

The project's cached `Device:D` symbol has **pin 1 = anode, pin 2 = cathode**. Stock KiCad has
it the other way round, and the stock THT diode footprints put the printed cathode band at the
**pad 1** end. So the silk band sits at the end the netlist treats as the anode — on all
sixteen 1N914s and on Z1.

The netlist intent is electrically correct (diode-AND gates with anode on the pulled-up node;
Z1 cathode to +5V). Only the printed marking misleads.

**Verify on the bare board:** the Z1 pad at the silkscreen band end should show continuity to
`GND`, the far pad to `+5V`. If so, fit every diode with its **body band pointing away from the
printed band**. Z1 backwards puts a forward diode across the 5 V rail.

### 3. No decoupling, and a tight fuse

- **No supply decoupling anywhere on the digital section.** C1–C4 all belong to the analogue
  block. Add 100 nF ceramic across pins 7/14 of each DIP (8/16 on the 7447s) and a 100 µF bulk
  cap at `PWR`. Much easier during assembly than after.
- **F1** is a 0.5 A PTC in series with the whole 5 V rail; worst-case draw is near 600 mA.
- **Z1** is a 1N4733A 5.1 V zener directly across a 5.0 V rail — close enough to its knee to
  conduct continuously. Consider a 1N4734A (5.6 V) or omit.

---

## Bench setup

`PWR` is a 3-pin JST PH at the right-hand edge. No on-board regulator — both rails come in
externally.

| Pin | Net | Set to | Current limit | Feeds |
|-----|-----|--------|---------------|-------|
| 1 | GND | 0 V common | — | Ground pour, all IC ground pins |
| 2 | +5V_IN | 5.00 V | start 100 mA | Through F1 to all logic and both displays |
| 3 | +17V | 17.0 V | 50 mA | Only R2, R4, R6, TR4 — analogue section |

**Sequencing:** bring 5 V up first, then *ramp* 17 V. Power 17 V down first, 5 V last. The
noise output couples through C3 (0.1 µF) straight onto IC1 pin 5, a TTL input with no positive
clamp; switching 17 V in abruptly couples that step into the input.

If the supply's outputs are independent, tie both negatives together at the board.

**Two changes worth making for bench work:**

- Link out F1 and let the bench current limit protect instead — faster, adjustable, and it
  removes the PTC volt-drop from rail measurements.
- Socket every IC. The whole plan depends on populating the board progressively.

---

## Stages

Ordered so each stage only powers what the previous one proved. The display end is proved first
and you work backwards toward the analogue front end.

### Stage 0 — Bare board, before soldering

| Check | Probe | Expect | Meaning |
|---|---|---|---|
| Ground integrity | PWR pin 1 → IC1 pin 7, IC5 pin 8 | < 1 Ω | Pour reaches every package |
| **LOGIC_0 defect** | IC3 pin 7 → PWR pin 1 | **OPEN** | Confirms the defect |
| LOGIC_1 rail | IC1 pin 1 → R25 pad 2 | < 1 Ω | Tie-high net routed |
| Rail short | +5V pad → GND | open | No fabrication short |
| **Diode marking** | Z1 band-end pad → GND | continuity | Band is on the anode — fit diodes reversed to print |
| 17 V rail | PWR pin 3 → R2, R4, R6 | < 1 Ω | Routed, and isolated from +5V |

### Stage 1 — Power section and the ground fix

Fit: PWR, F1 (or link), Z1, R25, the LOGIC_0 wire link, all IC sockets, decoupling caps.
Leave every socket **empty**. 5.00 V, limit 100 mA.

| Measure | Expect | If it differs |
|---|---|---|
| Total 5 V current | < 5 mA | Tens of mA → Z1 conducting; swap for 1N4734A or omit |
| Rail at each socket Vcc | 5.00 V | Sag points at F1 or a routing fault |
| LOGIC_1 at IC1 pin 1 | ≈ 5.0 V | 0 V → R25 open or misfitted |
| LOGIC_0 at IC2 pin 3 | 0.00 V | Link did not take, or net not fully routed |
| Z1 temperature | cold | Warm → continuous conduction |

**Stop if** 5 V current exceeds a few mA with no ICs fitted.

### Stage 2 — The 17 V rail

Sockets still empty. With 5 V already up, **ramp** the second output slowly to 17 V, limit
50 mA. Confirm 17 V at the top of R2, R4, R6, and **0 V at IC1 pin 5** (held down by R5).

### Stage 3 — Noise generator and clock

Fit: TR1–TR3, TR4, R1–R7, C1–C4. Logic sockets still empty.

Orientation notes:
- TR1–TR3 = 2N2926, TO-92, pad 1 = E, 2 = B, 3 = C.
- TR4 = 2N2646 UJT in **TO-18 metal can** (different footprint), pad 1 = E, 2 = B1, 3 = B2.
- C1, C2, C4 are correctly silk-marked (unlike the diodes): **+** is the pad 1 end.
- **TR1's collector is deliberately unconnected.** The noise comes from its reverse-biased
  base-emitter junction avalanching. Do not "fix" it.

| Test | Where | Pass criterion |
|---|---|---|
| Clock running | across R7 (220 Ω) | A pulse every ~3–4 s, visible on a DMM |
| Noise present | NOISE_OUT / IC1 pin 5 | Random noise you **cannot trigger the timebase on**. A stable trace means the noise source is *not* working. |
| DC sanity | NOISE_OUT to GND | Low DC level (R5 holds it down). Near 17 V → C3 leaky or misfitted. |

No noise usually means this particular 2N2926 will not avalanche at 17 V — breakdown scatters
between devices. Try another transistor.

**Stop if** NOISE_OUT sits at a high DC level — fitting IC1 with 17 V on a TTL input destroys it.

### Stage 4 — Displays and decoders, via the lamp test

Fit: IC5, IC6, R11–R24, both DL707s. **5 V only, 17 V off.** Raise the 5 V limit to ~700 mA.

The 7447 lamp-test input is on **pin 3** of both decoders. **Ground pin 3 and every segment
lights**, showing `8`. That verifies the decoder, all seven segment resistors, the socket wiring
and the display itself, per digit, with no counter or clock involved.

With both digits showing `8` you are at worst-case display current — **this is also your F1
margin measurement**. Note the total.

Blanking (pin 4 on both 7447s) comes from IC2, which is not fitted, so it floats high and
blanking is inactive. Tack a temporary 1 kΩ from that net to +5 V so it is held rather than
floating.

Displays are confirmed **DL707 / MB101 common anode**; the board correctly ties socket pins 3
and 14 to +5 V. Socket pin 9 (third common anode) is unconnected — harmless, but strapping it
to pin 3 shares segment current across three pins instead of two.

### Stage 5 — Counters, on an injected clock

Fit: IC3, IC4, IC11, R8–R10.

With no switch panel wired the reset lines float high and hold both counters cleared (`00`).
To emulate **random numbers** mode on the bench, ground **S3a pin 1** and **S3b pin 1**
(the `IC4_R0` and `IC3_R0` connector pins). Both counters can then run 0–99.

Inject **1–2 Hz** at **IC4 pin 14** (units counter clock). A debounced pushbutton works for
single-stepping.

**Expect a strange counting sequence — it is correct.** The tens digit is clocked from the
units **C** output inverted by IC11c, so the tens digit advances each time the units digit
passes 4. The article does this deliberately: in dice mode the units counter resets at 6, so D
never toggles and a conventional carry would never fire. No numbers are lost.

**Stop if** you see a frozen `99` — that is the signature of the `LOGIC_0` defect.

### Stage 6 — Gate and bistable

Fit: IC1, IC2.

Keep the UJT clock out of circuit so you can step by hand: **lift R7**, fit a temporary 1 kΩ
from IC1 pin 9 to +5 V, and a momentary pushbutton from pin 9 to ground. Press-and-release
generates the falling edge that clocks IC2. Keep injecting the slow clock at IC4 pin 14.

**The net names lie — the wiring is right.** The custom 7472 symbol has its labels swapped
against the real SN7472 (clear on pin 2, preset on pin 13, Q on pin 8, Q̄ on pin 6). Both pairs
are inverted, so they cancel and the board is wired exactly as the article specifies. But the
net named `IC2_Q` is physically Q̄, and `IC2_QBAR` is physically Q.

| Action | Expect | Because |
|---|---|---|
| At rest | Display lit and static | Q̄ high, blanking inactive; gate closed |
| Press Play (S1 pin 1 to ground) | Display **blanks**, counting starts | Preset asserted: Q high opens the gate, Q̄ low blanks the 7447s |
| Release Play, pulse manual clock | Counting stops, number appears | J=0, K=1 resets on the clock edge |

Then restore R7 and let the real 3–4 s UJT clock take over.

### Stage 7 — Game modes and zero detector

Fit: D5–D20, R26–R30, IC9, IC10, the switch loom. Keep the injected slow clock.

| Mode | Range | Expected behaviour |
|---|---|---|
| **No game selected** | — | Holds `00`, will not count. Designed behaviour; confirm first. |
| Random | 00–99 | Full cycle in the odd sequence; stoppable on any count including `00` |
| Roulette | 0–36 | Reaches 33 via `36 30 31 32`, resets to `04`, continues `05 06 07`… |
| Pools | 1–59 | `…58 59 50 51 52 53 04 05…`. Stops on a single zero, **ignores stop when both digits are zero** |
| Dice | 1–6 each | Neither digit exceeds 6; **cannot be stopped while either digit shows zero** |

The last two rows are the zero detector working: IC9 spots an all-zero digit, IC10 combines the
two digits as AND or OR depending on S6c, IC11b restarts the count.

Finally remove every temporary link — blanking pull-up, manual clock switch, grounded reset
pins, injected clock.

---

## Reference

### Current budget (approximate, typical standard TTL)

| Load | Typical | Note |
|---|---|---|
| Nine TTL packages | ≈ 290 mA | The two 7447s dominate at ~64 mA each |
| Both digits showing `88` | ≈ 155 mA | 14 segments × ~11 mA through 270 Ω |
| Typical running total | ≈ 400 mA | Average digit lights about five segments |
| **Worst case** | **≈ 580 mA** | Past F1's 0.5 A hold current |
| 17 V rail | < 15 mA | Only R2, R4, R6 |

### Symptom → cause

| Symptom | Most likely cause |
|---|---|
| Frozen `99`, never counts | `LOGIC_0` floating — set-to-nine inputs read high |
| Frozen `00`, never counts | No game selected, or a reset line stuck high — switch latching or loom |
| 5 V rail sags under load | F1 near its hold current — link it out for bench work |
| Rail collapses at power-on | Z1 fitted backwards |
| Noise trace triggers cleanly | Noise source dead; TR1 not avalanching. Try another 2N2926. |
| Erratic counts, random restarts | Missing decoupling |
| Tens digit advances "too early" | Correct behaviour — carry is the inverted units C output |
| A game mode never resets | Diodes fitted to the silkscreen band instead of against it |

### Open items for the switch loom

- The second red pushbutton on the original panel is still unidentified (annotated `?` in the
  Fig. 5 overlay on `IMG_5886.jpg`).
- S1's third contact has no confirmed use; the board only needs two.
- Dashed lines in the magazine drawing are **mechanical ganging only**, not electrical —
  S3a/S3b etc. are poles of a single interlocked pushbutton.
