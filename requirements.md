# LumenPnP Requirements

System-level expectations for the LumenPnP build, hand-written for compiler v1
(spec 11's interview agent replaces hand-writing in phase 3). Each `req` block
is machine-parsed by the skanda expectation compiler — edit values, keep the fields.

> DRAFT under review — blocks whose `# source:` says "estimate" need confirmation.
> LumenPnP has no battery, so REQ-004 is an idle current-draw check, not a
> battery-life budget (per spec decision 2026-08-14).

## Electrical

```req
# source: bom.csv PSU line, verbatim: "24v-power-supply ... A power supply
# that can output at least 6A @ 24v DC is recommended." Cross-checked against
# pnp/pcb/mobo/power.kicad_sch: input jack is a COMBO_POWER_JACK, protected by
# an SMF28A TVS diode (28V standoff) — sized for a 24V nominal rail with margin
# to the 35V/50V-rated bulk caps on the same sheet. Grounded, not an estimate.
id: REQ-001
title: 24V input rail within PSU regulation
quantity: voltage
subject: "mobo/+24V"
target: 24.0
unit: V
tolerance: {lo: 22.8, hi: 25.2}
verification: dmm
```

```req
# source: pnp/pcb/mobo/power.kicad_sch — TPS54560BDDA buck converter feeds the
# "+5V" net directly (component and net both present on that sheet). No
# datasheets/ dir checked into this repo, so the exact regulation spec isn't
# verified here; band is a generic ±4% pending the TPS54560 datasheet (plan C
# can tighten this once it reads the part datasheet).
id: REQ-002
title: 5V logic rail within regulation
quantity: voltage
subject: "mobo/+5V"
target: 5.0
unit: V
tolerance: {lo: 4.8, hi: 5.2}
verification: dmm
```

```req
# source: pnp/pcb/mobo/microcontroller.kicad_sch shows STM32F407VETx powered
# from the "+3.3V" net; pnp/pcb/mobo/power.kicad_sch shows that net generated
# by an AP1117-33 (a fixed 3.3V LDO — real part in the schematic, not
# invented). VDD abs range for the STM32F407 is 1.8-3.6V; band held to ±3%
# consistent with a fixed-output LDO. Grounded.
id: REQ-003
title: 3V3 MCU rail within regulation
quantity: voltage
subject: "mobo/+3.3V"
target: 3.3
unit: V
tolerance: {lo: 3.2, hi: 3.4}
verification: dmm
```

```req
# source: ESTIMATE - no basis found in this repo. pnp/pcb/mobo/motor_driver.kicad_sch
# confirms multiple TMC2226-SA stepper driver ICs on the motherboard (real parts,
# real quantity not counted here) and microcontroller.kicad_sch confirms an
# STM32F407VETx MCU, but there is no datasheets/ directory and no published
# current-consumption figures anywhere in this checkout. Target/band below are a
# rough order-of-magnitude placeholder (driver quiescent current + MCU + logic
# overhead, reflected through the 24V input after buck/LDO conversion losses).
# MUST be replaced with a bench PSU ammeter reading (this REQ's own
# verification method) before treating this number as real.
id: REQ-004
title: motherboard idle current draw on 24V input
quantity: current_idle
subject: "mobo/+24V"
target: 0.2
unit: A
tolerance: {lo: 0.05, hi: 0.5}
verification: psu
```

## Mechanical

```req
# source: ESTIMATE - no mass or weight figure exists anywhere in this repo
# (bom.csv has no mass column; CAD masses would require opening binary
# STEP/STL files, which is out of scope here per policy). Band is
# deliberately wide (roughly +-40%) to signal genuine uncertainty rather than
# false precision. MUST be replaced with an actual bench-scale weighing
# (this REQ's own verification method) before treating this as real.
id: REQ-005
title: total machine mass
quantity: mass
subject: "stage:step:machine"
target: 14.0
unit: kg
tolerance: {lo: 8.0, hi: 20.0}
verification: scale
```

```req
# source: bom.csv EXT line, verbatim: "vslot-extrusion-20mmx20mmx600mm", qty 7
# - the frame is built from 600mm v-slot members (grounded fact). The 700mm
# overall-depth target itself is NOT independently confirmed in this repo (no
# assembly-level envelope dimension found); it is carried over from a
# secondary, unverified citation (docs.opulo.io set-up guide, "a desk at
# least 28 inches / 71cm deep will perfectly support a LumenPnP") that this
# session did not re-fetch. Treat the 700mm figure as needing confirmation
# even though the 600mm-extrusion basis is solid.
id: REQ-006
title: machine depth fits a 71cm bench
quantity: dimension
subject: machine
target: 700.0
unit: mm
tolerance: {lo: 0.0, hi: 710.0}
verification: caliper
```

## Harnesses

```req
# source: pnp/cad/CHA/y1-motor/y1-motor.yml (real WireViz harness definition,
# not invented) - JST PH 4-pin (motherboard side) to JST PH 6-pin (motor
# side), 4x 22 AWG stranded conductors, 0.78m cable length. 22 AWG copper is
# ~53 mOhm/m, so intrinsic one-way conductor resistance is ~41 mOhm; adding
# two JST PH crimp/contact interfaces keeps a healthy connection well under
# 100 mOhm. 0.2 Ohm nominal / 1.0 Ohm ceiling is a conservative catch-all that
# flags a bad or missing crimp without false-triggering on nominal wire
# resistance. Grounded (wire/connector facts real; threshold is derived, not
# a hard spec).
id: REQ-007
title: harness conductor continuity end-to-end
quantity: resistance
subject: cha/y1-motor
target: 0.2
unit: ohm
tolerance: {lo: 0.0, hi: 1.0}
verification: dmm
```

## Fasteners

```req
# source: bom.csv FST line, verbatim: "M5-tslot-nut" (qty 60) - the subject
# below is a real part stem in this repo, not invented. The torque VALUE
# itself is still an ESTIMATE: no torque spec was found in this repo (no
# fastener spec sheet or datasheets/ dir). 3.0 N*m is standard-practice
# guidance for M5 into a t-slot nut in aluminum v-slot (derated well below
# the ISO 8.8 steel-steel table value of 5.9 N*m, because t-nut-in-aluminum
# strips at lower torque than steel-on-steel). Needs confirmation against
# the actual t-slot nut's rated torque before treating as ground truth.
id: REQ-008
title: frame v-slot joint fastener torque
quantity: torque
subject: FST/M5-tslot-nut
target: 3.0
unit: N*m
tolerance: {lo: 2.5, hi: 3.5}
verification: torque_driver
```
