# Games Machine — required KiCad edits before rev 2.0 board order

Running list of board fixes discovered during rev 1.0 bring-up (2026-08-08 → 2026-08-16).
All are worked around on the rev 1.0 board; all need real fixes in the KiCad source.

## 1. Tie `LOGIC_0` to `GND` — DONE in schematic, PCB pending
`LOGIC_0` is a floating net on rev 1.0 — 38 routed segments, 12 pads (IC2 pins 3/4/5, IC3/IC4
pins 6/7, S3a/S3b/S5a pin 3, S6c/S6d pin 2), no connection to ground anywhere. Symptom on an
unfixed board: frozen `99`. Rev 1.0 workaround: bodge wire IC3 pin 7 → IC3 pin 10.

**Done (commit d471d4e):** all 12 `LOGIC_0` labels renamed to `GND`, merging the nets. ERC
shows no new violations.
**Still to do in Pcbnew:** Tools → Update PCB from Schematic, then route / zone-fill the
former `LOGIC_0` pads. Not fabrication-ready until that's done and DRC passes.

## 2. Fix diode symbol/footprint polarity convention — DONE in schematic, PCB pending
The project's cached `Device:D` symbol had pin 1 = anode; the stock footprint
(`Diode_THT:D_DO-35_SOD27_P7.62mm_Horizontal`) correctly puts the silkscreen cathode band at
pad 1. The footprint was never wrong — only the schematic's *cached copy* of the symbol had
drifted from the stock library. Result on rev 1.0: printed band marks the **anode** end for
all of D5–D20 and Z1, so every diode had to be fitted opposite the silkscreen.

**Done:** swapped the pin *numbers* in the cached `Device:D` symbol (pin at the "A" position
is now 2, pin at the "K" position is now 1), leaving pin names, graphics, and every wire
untouched. Because KiCad binds wires by coordinate rather than pin number, this fixes all 17
diodes (D5–D20 + Z1, all of which share this one symbol) in a single edit with no per-instance
changes. Verified by netlist export: Z1 now reads pin 1 → `/+5V`, pin 2 → `/GND` (was the
reverse), and D11 pin 1 → `/IC3_QA`, pin 2 → `/S4A_ROULETTE37` (was the reverse) — electrical
intent unchanged, pin numbering now matches the footprint. ERC unchanged at 348 pre-existing
violations, same category breakdown.
**Still to do in Pcbnew:** Update PCB from Schematic, confirm the diode pads swapped as
expected, re-route if needed, and check a silkscreen plot shows band = cathode before ordering.

## 3. Swap UNITS and TENS display positions — DONE, display area needs re-route
UNITS display sits on the left (x=149), TENS on the right (x=164) — reverse of reading order.
Wiring and silkscreen labels are internally consistent; the number just reads transposed
(board shows "41" for the value 14). This masqueraded as a Roulette range failure during
bring-up.
**Done:** exchanged the two footprint placements — TENS1 now sits at x=149 (left), UNITS1 at
x=164 (right). Both are identical DIP-14 footprints at the same Y with no rotation, so this was
a straight swap of their `(at ...)` positions; each display keeps its own segment nets. The
number now reads in natural order.

The segment resistor banks were deliberately left alone: R11–R17 (tens) and R18–R24 (units) are
horizontal rows that already span the full width above both displays, so they don't need to move
— only the routing between them changes.

**Still to do in Pcbnew:** re-route the display area. Immediately after the swap DRC shows 14
shorts and 14 unconnected, because the old copper stayed put while the footprints traded places.

## 4. Fix transistor footprint pin order — TR1, TR2, TR3 (2N2926) — DONE in schematic, PCB pending
The schematic/footprint assumed E-B-C lead order, but the real 2N2926 is **E-C-B** (per
Central Semiconductor datasheet). On rev 1.0 all three had to be installed with leads 2 and 3
crossed.
**Done:** swapped the B and C pin numbers on the project-local `2N2926` symbol, in both
`GamesMachine.kicad_sym` and the schematic's cached copy. Pin functions and graphics are
unchanged; only the pad each one maps to moved. Netlist now reads pad 1 = E, pad 2 = C,
pad 3 = B for TR1/TR2/TR3, matching the real device, so the part drops straight in with no
crossed leads. TR4 (2N2646, TO-18) deliberately left alone — it is correct as built.
**Still to do in Pcbnew:** Update PCB from Schematic, then re-route TR1-TR3 (pads 2 and 3
swap nets).

## 5. TR4 (2N2646 UJT) footprint — NO CHANGE NEEDED
Checked against the 3D view on 2026-08-17: the pin mapping is correct as drawn and TR4 drops
in fine. The part just needs rotating roughly 10° clockwise from the silkscreen tab mark when
fitting, because the TO-18 leads sit on a circle and don't line up dead-on with the pad angles.
That is a fitting nuance, not a defect — deliberately left alone.

Worth remembering that identifying a UJT's leads should always be done by measurement (the
"sharpie method": ohms-mode bar pair = the two bases, leftover lead = emitter, then the
can-to-lead short splits B1 from B2) rather than by trusting any pinout diagram — two of the
three 2N2646s obtained during rev 1.0 bring-up were either counterfeit or dead.

## Lower priority / consider for rev 2
- ~~No decoupling capacitors anywhere on the digital section~~ — **decided against adding
  these**; board runs fine without them, not worth the layout churn.
- F1 (0.5 A PTC) undersized against ~580 mA worst-case display current — size up.
- Z1 (1N4733A 5.1 V) sits close to its knee on a 5.0 V rail — consider 5.6 V part or omit.
- Custom 7472 symbol has PRE/CLR and Q/Q̄ label pairs swapped vs. the real SN7472 (cancels out
  electrically, but net names `IC2_Q`/`IC2_QBAR` are physically backwards — confusing to probe).
  Relabel symbol pins and net names to match reality.
