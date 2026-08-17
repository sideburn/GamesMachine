# Games Machine — required KiCad edits before rev 2.0 board order

Running list of board fixes discovered during rev 1.0 bring-up (2026-08-08 → 2026-08-16).
All are worked around on the rev 1.0 board; all need real fixes in the KiCad source.

## 1. Tie `LOGIC_0` to `GND`
`LOGIC_0` is a floating net on rev 1.0 — 38 routed segments, 12 pads (IC2 pins 3/4/5, IC3/IC4
pins 6/7, S3a/S3b/S5a pin 3, S6c/S6d pin 2), no connection to ground anywhere. Symptom on an
unfixed board: frozen `99`. Rev 1.0 workaround: bodge wire IC3 pin 7 → IC3 pin 10.
**Fix: join `LOGIC_0` to `GND` in the schematic (or just relabel the net GND) and reroute.**

## 2. Fix diode symbol/footprint polarity convention
The project's cached `Device:D` symbol has pin 1 = anode; stock KiCad footprints put the
silkscreen cathode band at pad 1. Result: the printed band marks the **anode** end for all of
D5–D20 and Z1, so every diode had to be fitted opposite the silkscreen. Netlist is electrically
correct — only the silk misleads.
**Fix: replace the cached symbol with the stock `Device:D` (pin 1 = cathode) and re-check every
diode's orientation in the schematic, or fix the footprint band. Verify band = cathode on the
new plot before ordering.**

## 3. Swap UNITS and TENS display positions
UNITS display sits on the left (x=149), TENS on the right (x=164) — reverse of reading order.
Wiring and silkscreen labels are internally consistent; the number just reads transposed
(board shows "41" for the value 14). This masqueraded as a Roulette range failure during
bring-up.
**Fix: swap the two display footprint placements so TENS is on the left, UNITS on the right.**

## 4. Fix transistor footprint pin order — TR1, TR2, TR3 (2N2926)
The schematic/footprint assumed E-B-C lead order, but the real 2N2926 is **E-C-B** (per
Central Semiconductor datasheet). On rev 1.0 all three had to be installed with leads 2 and 3
crossed.
**Fix: correct the symbol/footprint pin mapping so the part drops in straight.**

## 5. Fix TR4 (2N2646 UJT) footprint pin mapping
Same class of problem: the TO-18 footprint pad order vs. the real 2N2646 pinout required
working out orientation by measurement ("sharpie method") rather than trusting the silkscreen
notch. Rev 1.0 install ended up straight-in after much confusion, but the symbol/footprint
mapping was never trustworthy.
**Fix: verify the 2N2646 symbol pin numbers against the TO-18 footprint pads (E/B1/B2 vs pads
1/2/3 and the tab position) and correct so the notch/tab on silkscreen matches the real part.**

## Lower priority / consider for rev 2
- ~~No decoupling capacitors anywhere on the digital section~~ — **decided against adding
  these**; board runs fine without them, not worth the layout churn.
- F1 (0.5 A PTC) undersized against ~580 mA worst-case display current — size up.
- Z1 (1N4733A 5.1 V) sits close to its knee on a 5.0 V rail — consider 5.6 V part or omit.
- Custom 7472 symbol has PRE/CLR and Q/Q̄ label pairs swapped vs. the real SN7472 (cancels out
  electrically, but net names `IC2_Q`/`IC2_QBAR` are physically backwards — confusing to probe).
  Relabel symbol pins and net names to match reality.
