# 30 W Offline Flyback Charger

A 30 W isolated AC-DC flyback converter designed from first principles and
validated in LTspice. Mains input, 20 V at 1.5 A out, with optocoupler
feedback, an RCD clamp and output overvoltage protection.

**Status:** Simulation stage. Schematic capture started, no board built.
**Tools:** LTspice 26.0.1, Altium Designer, Excel

## Specification

| Parameter | Value |
|---|---|
| Input | 220 V, 50 Hz sine source |
| Output | 20 V at 1.5 A, 13.33 Ω load |
| Output power | 30 W |
| Topology | Isolated flyback, discontinuous conduction |
| Primary inductance | 300 µH |
| Secondary inductance | 12 µH |
| Coupling coefficient | 0.999 across primary, secondary and auxiliary |
| Turns ratio | 5:1 |
| Isolation | Optocoupler feedback, no galvanic path across the barrier |

## What is in this repository

| Path | Contents |
|---|---|
| `simulation/flyback-30w.asc` | Full converter, LTspice |
| `simulation/ovp.asc` | Output overvoltage protection, simulated separately |
| `simulation/window-voltage-protection.asc` | Input window detector, abandoned, see below |
| `docs/transformer-calculator.xlsx` | Transformer sizing spreadsheet |
| `hardware/` | Altium project and schematic document |

## Design approach

The converter is a single-switch isolated flyback running in discontinuous
conduction. Flyback was chosen over forward or resonant topologies because
at 30 W it needs one magnetic component and one switch, and isolation comes
free with the transformer rather than requiring a separate barrier.

### Switch selection and voltage stress

The switch sees the rectified bus plus the reflected output plus the leakage
spike. With a 5:1 turns ratio the reflected secondary voltage is about 100 V
on top of the rectified bus, before any clamp overshoot. An STW11NM80 rated
at 800 V was chosen to leave headroom for the leakage spike and for high-line
operation rather than sizing to the nominal case.

Gate drive is through a 10 Ω series resistor, slowing the edge enough to
reduce ringing without pushing switching loss up unnecessarily.

### Leakage energy

Transformer leakage cannot couple to the secondary, so it has to go
somewhere at turn-off or it appears across the switch. An RCD clamp of
2.2 nF and 47 kΩ through a VS-E5PH3006 ultrafast diode catches that energy
and bleeds it through the resistor. The capacitor is rated 450 V because it
sits at the drain node.

### Feedback loop

Regulation is closed around a TL431 shunt reference driving a PC817
optocoupler into an LT1242 current-mode controller. The output divider is
75 kΩ over 10.135 kΩ, which with the TL431 2.5 V reference sets the output
in the 20 V region. Compensation sits on the TL431 cathode, a 10 kΩ and
1.8 nF series network with 0.1 nF across it for high-frequency rolloff.

The optocoupler is what keeps the control loop isolated. Nothing crosses the
barrier except light.

### Output stage

An MBR20100CT Schottky rectifies the secondary. Bulk output capacitance is
470 µF and 200 µF with a 2.2 µH inductor between them, giving a second-order
post-filter to pull down the switching ripple that the bulk capacitor ESR
leaves behind.

### Overvoltage protection

If the feedback loop opens, the output runs away. A 22 V zener
(BZX84B22VLY) into the base of a 2N2907 pulls the output down once it passes
threshold, giving a hard limit that does not depend on the loop being
healthy. Simulated separately in `ovp.asc`.

## Design decisions and dead ends

**Window voltage detector, abandoned.** An input-range detector built from
two shunt references was intended to inhibit the converter outside a valid
input window. It is in `window-voltage-protection.asc`. The problem is that
the reference threshold and the drive current into it are coupled when the
network sits on a high-voltage rail, so the trip point moves with the thing
it is supposed to be measuring. It was dropped rather than forced, and the
overvoltage protection on the output covers the failure mode that actually
matters.

**Transformer calculator inputs do not match the final design.** The
spreadsheet in `docs/` was the sizing tool, and its stored inputs
(11.5:1 ratio, 147 µH primary) are from an earlier design point, not the
300 µH and 5:1 that the simulation ended up using. It is included because
the method is the useful part: primary inductance from power and duty,
secondary from the turns ratio, turns from core area and peak flux density,
then air gap from the AL value needed to land on that inductance. The
switching frequency cell is also stale.

## Key components

| Reference | Part | Why |
|---|---|---|
| M1 | STW11NM80 | 800 V, sized for reflected voltage plus leakage spike rather than nominal bus |
| D1, D3, D4, D5 | RFNL5BGE6S | Input bridge rectifier |
| D6 | VS-E5PH3006 | Ultrafast clamp diode, has to recover faster than the leakage spike |
| D2 | MBR20100CT | Schottky output rectifier, low forward drop at 1.5 A |
| U1 | LT1242 | Current-mode PWM controller |
| U2 | PC817 | Optocoupler, isolates the feedback path |
| U3 | TL431 | Shunt reference and error amplifier |
| D7 | BZX84B22VLY | 22 V zener, output overvoltage trip |

## Outstanding work, in order:

1. Export the simulation results. The transient run in this repository has
   no saved waveforms, so the regulation and ripple figures are not yet
   documented here.
2. Reconcile the transformer calculator with the final design point.
3. Finish the Altium schematic and take it to layout.

## Licence

MIT. See [LICENSE](LICENSE).
