---
title: "Instrument pack: getting its data into Ostler, then a screen in its place — research plan"
area: references
status: stable
version: 1.0
updated: 2026-10-14
depends_on: [references/research/instrument-pack/wiring.md, references/research/instrument-pack/data-sources.md, references/research/instrument-pack/ostler-fit.md, references/protocol_state_handoff.md]
summary: >
  Research only, no decisions. The owner's aim (2026-10-14): first get the D2 Td5 instrument pack's data
  into Ostler, and eventually replace the pack with a screen. Shows the route the research points to: a
  read-only Y-harness tap on C0230/C0233 into an ESP32-S3 input node (pulse, PWM, differential fuel
  sender, lamp opto senses, receive-only SK bus), the Td5 over K-line for extra values, and the
  original pack kept connected but hidden. It sets out stages, the legal and platform constraints, where
  each piece would live, and the car tests that would close the gaps. Points for later are listed as
  open questions, not decisions.
---

# Instrument pack: getting its data into Ostler, then a screen in its place

**Research only; nothing here is decided.** The owner's aim (2026-10-14): first get the pack's data into
Ostler, and eventually replace the pack with a screen. A design spec comes later, when that work starts.

**Evidence:** the three research notes in
[this folder](.):

- [wiring.md](wiring.md): pinouts and lamp drivers;
- [data-sources.md](data-sources.md): K-line vs taps, and the law;
- [ostler-fit.md](ostler-fit.md): the platform's pieces and gaps.

Facts below carry those notes' confidence marks. Anything marked *inferred* is a car test, not a design input.

## 1. What the pack is today

- **Plugs.** Two plugs behind the pack:
  - **C0230**, 20-way: supplies and earths, main beam, auto sport/manual, oil, gearbox oil temp,
    diff lock, off-road, brake, charge, illumination, fuel sender, tach, road speed.
  - **C0233**, 16-way: ACE, SLS, ABS, TC, HDC ×2, SRS (with its own feed), anti-theft, water in
    fuel, glow plugs, MIL, coolant PWM, the 03MY satnav speed output, and the **SK serial bus**.
- **The pack is a computer.** It has a microprocessor, the dials and an LCD, and it decides the
  low-fuel and hot-coolant lamps itself. It holds the odometer, which the BCU mirrors. It drives
  the chime speaker. It passes road speed to the BCU and IDM over the SK bus.
- **The pack is not on the pin-7 K-line.** The T-26 sweep confirms this.
- **Gauge inputs:**
  - tach: Td5 ECM, 2 pulses per rev;
  - speed: SLABS, 0–12 V square wave, 8000 pulses per mile;
  - coolant: PWM from the ECM;
  - fuel: tank sender, 19 Ω full to 245 Ω empty, floating (the pack supplies its earth).
- **Lamps.** Almost all are LEDs inside the pack, switched by a **low-side sense line**. The pack
  pulls the line up; the owning ECU or switch pulls it to earth (< 1.8 V on, > 7.7 V off). The exceptions:
  - main beam is a +12 V high-side feed from the IDM;
  - indicators, trailer, seat belt, transfer neutral and the PRND321 display **exist only as SK bus
    messages** from the BCU and IDM (10 400 bit/s, single wire).
- **Bulb check.** The ECUs run the 3 s ignition-on bulb check on their own lines; the pack checks only a few.

## 2. Constraints that shape the plan

1. **Read-only from the car** (pack and platform constitutions; ADR-0032 §14). Taps never load or
   alter an ECU signal. Any output to the car needs its own ADR.
2. **Removing the pack breaks things:**
   - The Td5 ECM logs MIL and glow-lamp open-load faults (`11.5`/`13.5`, `11.6`/`13.6`).
   - The SRS DCU logs fault `004`, warning lamp open circuit.
   - The EAT may log its lamp faults.
   - The alternator may not excite, because C0230-11 is its D+ feed (*inferred*, must be checked).
   - The BCU loses road speed, so speed-related locking stops.
   - The chimes and odometer go.
3. **The law (UK).** C&U regs 35/36 require a working speedometer, lit, showing mph and km/h.
   R39 says it must never read low and may read at most 10 % + 4 km/h high. The MOT fails a dead
   speedo (7.8), a missing main-beam tell-tale (4.9), a non-working ABS lamp (1.6) and an SRS lamp
   showing a fault (7.1.6).
4. **K-line can't carry a cluster on its own:**
   - Only the Td5 ECM answers while driving; SLABS goes silent once the car moves.
   - The bus runs one session at a time.
   - The realistic rate is about 4–8 Hz for rpm and 2–4 Hz for speed. Those figures are *inferred*;
     the node does about 1 Hz today.
   - There is no fuel level and no lamp state over K-line.
5. **Platform gaps** (ostler-fit G3 and G9–G14):
   - the node can't yet unlock the Td5 (keygen, gate tier);
   - freshness is 30 s and module-wide, where instruments need per-signal budgets in under a second;
   - there is no boot-time budget: the Brain can take up to 60 s;
   - there is no protected cluster layout;
   - VISS isn't served yet.

The conclusion: **tap the pack's wiring for everything it shows, keep the original pack connected,
and use K-line for engine values the pack never had.**

## 3. Where each signal comes from

Primary sources are high-impedance taps on the pack side of C0230 and C0233. Criticality: **H** = drive-safety or legal.

| Signal | Primary (tap) | Pin | Fallback | Crit |
|---|---|---|---|---|
| Road speed | Pulse period capture, 8000 p/mile | C0230-16 | Td5 `21 0D`, then GPS | **H** |
| RPM | Pulse period capture, 2 p/rev; ignore the 228 rpm key-on signal | C0230-15 | Td5 `21 09` | M |
| Coolant temp (gauge) | Td5 `21 1A@0` °C over K-line | — | PWM duty capture (Td5 scaling *unknown*) | M |
| Hot coolant (lamp) | Derived inside the pack, so no line to tap. Use the PWM or K-line value with the pack's thresholds | C0233-15 | — | **H** |
| Fuel level | Differential voltage across the sender, with software damping | C0230-14 / -13 | Td5 fuel-used integral | M |
| Low fuel | Derived from the sender: on above 158 Ω, off below 113 Ω | as above | — | M |
| Oil, charge, brake, ABS, TC, HDC ×2, SLS, off-road, ACE, SRS, glow, MIL, water in fuel, gearbox oil temp, diff lock, anti-theft | Opto lamp sense on each line (< 1.8 V on) | see wiring.md §3 | K-line fault reads at standstill | **H** for oil, charge, brake, ABS, SRS; M for the rest |
| Sport / Manual (both low = EAT fault) | Two comparators | C0230-4 / -5 | — | M |
| Main beam | High-side divider or opto | C0230-3 | — | **H** |
| Indicators, trailer, seat belt, transfer neutral, PRND321, chime requests | **Receive-only tap of the SK bus** (decode to be captured) | C0233-16 | Taps on the indicator and selector switch lines (W/X/Y/Z) | **H** for indicators |
| Odometer | Integrate speed pulses into non-volatile memory, seeded from the pack's reading. The pack stays the legal record | — | GPS distance | M |
| Illumination / dimmer | Voltage sense, to dim Ostler with the panel lights | C0230-12 | Ambient sensor | L |
| Extras (boost, fuel temp, battery, IAT, faults) | Td5 over K-line | — | — | L |

## 4. Hardware: the cluster tap node

The cluster tap node is an **ESP32-S3 node** (ADR-0032 / ADR-0039, "one firmware, all variants")
with an input board.

- **Harness:** a **pass-through Y harness**, male and female C0230 and C0233, fitted between the
  pack and its plugs. Nothing is cut, and it is reversible. Every tap is high-impedance and isolated.
- **Pulse inputs ×2 (speed, tach):** Schmitt or comparator front-end, input impedance ≥ 100 kΩ,
  clamped, then opto. Captured with PCNT or MCPWM period capture on the ESP32-S3.
- **PWM input ×1 (coolant):** same front-end, duty capture.
- **Differential analogue ×1 (fuel):** an instrumentation amplifier across C0230-14/-13, read on ADC1.
  - With the pack kept, it measures the voltage the pack's own drive puts across the sender.
  - Calibrate full, ¼, ½, ¾ and empty on the car.
- **Lamp senses ×20:** an opto per line, fed from the line through ≥ 47 kΩ, so the load is tiny next
  to the pack's pull-up. The thresholds follow the pack's (< 1.8 V on, > 7.7 V off).
  - Read through an I²C expander (e.g. MCP23017 ×2).
  - The main beam's high-side tap uses its own divider and opto.
- **SK bus ×1:** a K-line-type transceiver wired **receive only**, with the TX pin not fitted, at 10 400 bit/s.
  The firmware never transmits on it.
- **Supplies:** fed from C0230-19 (ignition) and -20 (permanent), with its own fuse. A sense on
  C0230-12 for illumination.
- **Link to the Brain:** a wire (10BASE-T1S or USB-NCM), not Wi-Fi, for the tell-tales. This is open question Q3.
- **Boot:** the node must publish tell-tales **within 1 s of ignition-on**, so the ECUs' 3 s bulb
  checks are visible on screen.

## 5. Stages

### Stage A: augment (no change to the car's behaviour)

- **Fit and read:** fit the Y harness and tap node. Publish every signal in §3 as VSS through the
  node, with source tags and per-signal age.
- **K-line on the node:** add a dedicated Td5 poller for the extras. This needs the Td5 keygen and
  gate tier on the node.
- **Display:** Ostler shows the data in an instrument layout. The stock pack keeps doing its job, so
  this stage is legal by construction.
- **Done when:**
  - rpm and speed match the pack within 1 % on a drive;
  - every tap lamp, and the SK-bus tell-tales from Stage B, follow the pack, bulb check included;
  - no new ECU faults appear;
  - the pack's odometer is unchanged.

### Stage B: the Ostler cluster layout

- **Instrument pages** in the gateway (ADR-0052) for speed, rpm, fuel and coolant. A tell-tale strip
  shows R121 symbols and colours, with the locked caution and warning colours (shell spec §21).
- **Tell-tales become alerts in the gateway,** not in the apps. A warning shows even with no page open.
- **Per-signal freshness:** a stale speed or rpm reading **blanks** with "—", and never freezes.
- **A protected cluster region** that edit mode can't remove while driving. This revisits the
  "no fixed tile" ruling for this layout only.
- **SK bus:** decode it from captures to get indicators, seat belt, gear and neutral (car test T-IP-6).

### Stage C: replace the display, with the pack hidden but still connected

- **The swap:** an Ostler screen sits in the pack's aperture. The original pack moves behind the
  dash, still plugged into the Y harness. It keeps the odometer (the legal record), the chimes, the
  BCU's road speed, the alternator excitation and the lamp loads.
  - So no ECU faults are set and nothing is emulated.
- **Before Stage C is road-legal,** the Ostler cluster display needs:
  1. **An independent safety display path.** The node drives a small always-on speedometer and
     tell-tale renderer directly, without the Brain. It is lit within 1 s of ignition-on. If the
     Brain or the main screen fails, that path still shows speed and the H tell-tales. It could be a
     strip of LEDs or a small TFT on the node.
  2. **Speed accuracy to R39:** never under-read; over-read by at most 10 % + 4 km/h. Show mph with
     km/h. The 8000 p/mile pulse can be checked against GPS on the bench and in the car.
  3. **Bulb check on screen:** mirror the lines, so the ECUs' own checks appear.
  4. **MOT access:** the tester must see the ABS, SRS and main-beam tell-tales and a lit speedo with
     the ignition on.
  5. **A failure mode that is obvious:** no data means the speed blanks with a fault message. It never
     shows the last value.
- **Platform ADRs.** This stage needs ADRs in openostler/ostler for:
  - a cluster display class with a boot budget and a wired-only rule;
  - the protected layout;
  - the safety display path.

### Stage D: remove the pack entirely (not recommended)

Removing the pack means:

- emulating the lamp loads (dummy loads on the sense lines);
- keeping alternator excitation (a resistor or bulb on C0230-11);
- **transmitting** on the SK bus to give the BCU road speed and take over the chimes;
- carrying the odometer, which can't be reset and has to stay in step with the BCU's copy.

All of these are outputs to the car, which the constitution forbids without ADRs. The odometer is a
legal record. The research stops at Stage C.

## 6. What belongs where

| Piece | Repo |
|---|---|
| **Signal definitions** | **This pack.** The signal-store entries for each tap (pin, kind, scaling, thresholds, fuel calibration, lamp → VSS path, criticality); requests for overlay paths for D2 tell-tales; the SK bus decode as it is found; car tests in `references/test_plan.md` |
| **The cluster harness profile** | **This pack, as data.** Pins and kinds for C0230 and C0233. Ownership is open (G7); the proposal is that wiring facts are pack data and the firmware reads them |
| **Firmware** | **openostler/ostler-firmware:** the input board profile, PCNT/MCPWM capture, the ADC differential read, the lamp expander, the SK receive-only decoder, the Td5 keygen and gate rule, and the safety display path |
| **Platform docs** | **openostler/ostler:** ADRs for the cluster display class and protected layout (Stage C), the tell-tale overlay paths, per-signal freshness, and tell-tale → alert rules in the gateway |
| **UI** | **openostler/ostler-shell:** instrument pages, the tell-tale strip, blanking on stale data, and the R121 icon set |

## 7. Car tests

Candidates for `references/test_plan.md` when the work starts.

| ID | Test |
|---|---|
| T-IP-1 | Scope C0230-15 (tach): voltage level, at idle and at 3000 rpm, and the 228 rpm key-on signal. Confirms Td5 12 V vs 5 V |
| T-IP-2 | Scope C0230-16 (speed): level and frequency at 30 and 60 mph, checked against GPS |
| T-IP-3 | Scope C0233-15 (coolant PWM): frequency and duty against Td5 `21 1A` °C, to get the Td5 scaling |
| T-IP-4 | Fuel: differential voltage across C0230-14/-13 at known fills (full, then after measured litres) |
| T-IP-5 | Lamp lines: on and off voltages for each sense pin, and the bulb-check timing at ignition-on |
| T-IP-6 | SK bus: receive-only capture at 10 400 bit/s while operating the indicators, trailer, seat belt, gear selector and neutral. Frame format and IDs |
| T-IP-7 | Alternator excitation: does C0230-11 carry the field current? Measure the current with the pack in place |
| T-IP-8 | Y-harness fitted: drive cycle, then read Td5, SRS, EAT and SLABS faults to confirm no new faults |

## 8. Open questions for later (not decisions)

The aim is set: data first (Stages A and B), then a screen in the pack's place (Stage C). These are
for when the design work starts.

- **Q1 — backup display.** What form the independent speed and tell-tale path takes in Stage C: LEDs, a small TFT on the node, or a second small screen.
- **Q2 — where the pack goes.** Where the hidden original pack and the Y harness live: behind the dash, or in the glovebox on an extension.
- **Q3 — tell-tale link.** Whether tell-tales travel only over a wire, never Wi-Fi. The research points to wired only.
- **Q4 — SK bus.** Whether decoding the SK body bus, receive-only, is in this pack's scope.
- **Q5 — symbols.** R121 symbols and colours for the tell-tales, or the D2's own.

## Changelog

- 2026-10-14 — v1.0: research plan from three research notes (wiring, data sources and law, Ostler fit). Moved from `specs/` as research, not a design for approval (owner, 2026-10-14).
