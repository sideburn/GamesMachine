# Games Machine — rev 2.0 board fixes

Defects found during rev 1.0 bring-up (2026-08-08 → 2026-08-16) and how each was resolved in
the KiCad source (2026-08-16 → 2026-08-17).

**Status: all fixes applied. DRC clean — 0 violations, 0 unconnected. Gerbers and drill files
regenerated. Board is ready to order.**

---

## 1. `LOGIC_0` never tied to ground — FIXED

`LOGIC_0` was its own net with no connection to `GND` anywhere: 38 routed segments across 12
pads (IC2 pins 3/4/5, IC3 and IC4 pins 6/7, S3a/S3b/S5a pin 3, S6c/S6d pin 2) and nothing
driving it. Floating TTL inputs read high, so both 7490s sat permanently forced to nine — the
symptom on an unfixed board is a frozen `99` that never counts. Rev 1.0 workaround was a bodge
wire from IC3 pin 7 to IC3 pin 10.

**Fix:** renamed all 12 `LOGIC_0` labels to `GND`, merging the nets, then updated and re-routed
the PCB. The bodge wire is not needed on rev 2 — those pads are genuinely part of the ground
net now, so the pour reaches them.

## 2. Diode silkscreen band marked the anode — FIXED

The schematic's *cached copy* of `Device:D` had pin 1 = anode, while the stock footprint
(`Diode_THT:D_DO-35_SOD27_P7.62mm_Horizontal`) correctly places the silkscreen cathode band at
pad 1. The footprint was never wrong — only the cached symbol had drifted from the stock
library. Result: the printed band marked the **anode** end on all of D5–D20 and Z1, so every
diode had to be fitted opposite the silkscreen.

**Fix:** swapped the pin *numbers* in the cached symbol (the "A" position is now 2, the "K"
position now 1), leaving names, graphics and wiring untouched. KiCad binds wires by coordinate
rather than pin number, so this corrected all 17 diodes in one edit. Each diode footprint was
then rotated 180° so pad 1 sits where pad 2 was — this both restores the original routing
geometry and carries the band to the cathode end.

Verified by netlist: Z1 now reads pin 1 → `/+5V`, pin 2 → `/GND`; D11 pin 1 → `/IC3_QA`,
pin 2 → `/S4A_ROULETTE37` — both reversed from rev 1.0, electrical intent unchanged. Confirmed
visually in the 3D view.

## 3. Display digits in reverse reading order — FIXED

UNITS sat on the left (x=149) and TENS on the right (x=164), so every result read with its
digits transposed — the board showed `41` for the value 14. Wiring and silkscreen labels were
self-consistent, so this was purely placement, but it convincingly masqueraded as a Roulette
range failure during bring-up.

**Fix:** exchanged the two placements — TENS1 now at x=149 (left), UNITS1 at x=164 (right).
Identical DIP-14 footprints at the same Y with no rotation, so a straight swap; each display
keeps its own segment nets. Display area re-routed.

Segment resistor banks deliberately left alone: R11–R17 (tens) and R18–R24 (units) are
horizontal rows already spanning the full width above both displays, so only the routing
between them changed.

## 4. 2N2926 pin order — FIXED (TR1, TR2, TR3)

The project-local symbol declared E-B-C, but the real 2N2926 is **E-C-B** (per the Central
Semiconductor datasheet). All three had to be fitted with leads 2 and 3 crossed on rev 1.0.

**Fix:** swapped the B and C pin numbers in both `GamesMachine.kicad_sym` and the schematic's
cached copy. Netlist now reads pad 1 = E, pad 2 = C, pad 3 = B, so the part drops straight in.
PCB updated and re-routed.

## 5. TR4 (2N2646 UJT) footprint — NO CHANGE NEEDED

Checked against the 3D view on 2026-08-17: the pin mapping is correct as drawn and TR4 drops in
fine. The part just wants rotating roughly 10° clockwise from the silkscreen tab mark when
fitting, because the TO-18 leads sit on a circle and don't line up dead-on with the pad angles.
A fitting nuance, not a defect.

Identify a UJT's leads by **measurement**, never from a pinout diagram — ohms-mode resistive
pair = the two bases, leftover lead = emitter, then the can-to-lead short distinguishes B1 from
B2. Two of the three 2N2646s sourced during rev 1.0 bring-up were counterfeit or dead, and one
passed a static meter test while being unable to fire under real bias.

---

## Housekeeping done alongside

- Two stitching vias added at IC3 pad 7 and S5a1 pad 3, each with a short stub track, clearing
  the last two `starved_thermal` errors. Both are former `LOGIC_0` pads that reached only an
  isolated island of the top-layer pour.
- Gerbers and drill files regenerated from the current board (the folder previously held rev 1.0
  output).

## Considered and deliberately not changed

- **Decoupling capacitors** on the digital section — the board runs fine without them; not worth
  the layout churn.
- **F1** (0.5 A PTC) — the paper worst case is around 580 mA, but the built board runs correctly
  on it, so it stays as is.
- **Z1** (1N4733A, 5.1 V) sits close to its knee on a 5.0 V rail. Measured 4.7–4.8 V in practice
  with Z1 fitted, so it isn't conducting — fine as built. A 5.6 V part would give more margin if
  ever revisited.
- The custom **7472** symbol has its PRE/CLR and Q/Q̄ label pairs swapped versus the real SN7472.
  The two inversions cancel so the board is wired correctly, but net `IC2_Q` is physically Q̄ and
  `IC2_QBAR` is physically Q — confusing when probing. Worth relabelling if the symbol is ever
  touched again.
