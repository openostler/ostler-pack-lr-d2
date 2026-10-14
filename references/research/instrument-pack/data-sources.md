---
title: "D2 Td5 instrument data: K-line vs hardwired taps, and the law"
area: references
status: stable
version: 1.0
updated: 2026-10-14
summary: >
  Signal-by-signal choice between K-line (only the Td5 ECM answers while driving, about 4–8 Hz rpm with a dedicated poller; SLABS goes silent once moving) and high-impedance taps on the pack wiring, with fallbacks and safety criticality; tap interface circuits; UK/EU legal constraints (C&U 35/36, R39 speedometer accuracy, MOT 1.6, 4.9, 7.1.6, 7.8). Recommends keeping the original pack connected.
---

# Discovery 2 Td5: instrument-cluster data sources for Ostler

Research date: 2026-10-14. Vehicle: Land Rover Discovery 2 Td5 (1999–2004), K-line only (no CAN).
Scope: every signal the factory instrument pack (IP) shows, and how Ostler can get each one.
The sources are K-line diagnostics (A), hardwired taps (B) and other sensors (C). Legal and
safety constraints for the UK and EU follow at the end.

Repo context comes from this repo (`JamesWrightDavid/discovery2-diag`) (read-only): `docs/discovery-2-td5/*`,
`references/protocol_state_handoff.md`, `references/td5_fault_codes.md`, the SLABS and BCU
references, and the NanoCom menu transcriptions.

## Confidence marks

| Mark | Meaning |
|---|---|
| **[C] confirmed** | Proven on the car in the repo, or two or more independent sources agree. |
| **[S] single source** | One source only, for example one workshop-manual page or one forum post. |
| **[I] inferred** | My engineering inference from the facts above. Not documented anywhere. |

---

## 1. Key findings (read this first)

1. **Only the Td5 engine ECU is usable over K-line while driving.** [C]
   - The repo's ESP32 node holds a Td5 session at about 1 Hz on real drives, motorway included. Whole-LID read glitches run at about 1 %.
   - SLABS (ABS) **suspends diagnostics as soon as the car moves**, and stays silent until the next ignition cycle. The repo proved this on 2026-08-29, logging 0–71 km/h.
   - So wheel speeds and ABS, traction-control, HDC and SLS state **cannot** come over K-line while driving.
2. **The bus allows one diagnostic session at a time.** [C] Switching modules is slow:
   - Every module needs StopCommunication `82` first.
   - SLABS needs about 25 s of bus silence before it will accept an init.
   - The BCU needs a 5-baud init (about 2 s), fails roughly 1 attempt in 4, and wants an ignition transition first.
   - In practice, a driving display gets **Td5 only** from K-line.
3. **Achievable Td5 refresh is about 4–8 Hz for RPM and about 2–4 Hz for speed.** This assumes a dedicated poller (details in section 3). [I, from bus arithmetic plus [S] P2/P3 figures]
   - The hardwired tach and speed pulses carry the same information with under 100 ms latency, and are independent of any software session.
4. **Almost every warning lamp is a separate hardwired line into the IP.** [S: RAVE "Instruments, Description and Operation", page by page]
   - Most lines are **low-side**: the owning ECU or switch pulls the lamp line to earth. The IP supplies +12 V (fuse 27 or 13) through its LED.
   - The IP's own thresholds are **< 1.8 V = lamp on** and **> 7.7 V = lamp off**.
   - Three groups are different: main beam (+12 V high-side from the IDM), auto manual/sport (EAT drives the line high or low), and a few BCU **serial-link** items (transfer neutral, gear position, seat-belt chime).
5. **The ECUs run the 3-second bulb checks themselves, not the IP.** This covers ABS, SRS, HDC, TC, SLS, ACE, coolant, MIL and low fuel. [S]
   - A replacement display that faithfully mirrors the lamp lines therefore reproduces the bulb check, which the MOT relies on. That only works if it is awake within about 1 s of ignition-on.
6. **Removing the IP has side effects on the rest of the car.** [I, partly C]
   - The ECUs see open lamp loads. The Td5 has proven fault bits "MIL lamp drive open load" (`11.5`/`13.5`), "glowplug lamp drive open load" (`11.6`/`13.6`) and cruise-lamp drive faults. The SRS has fault `004` "warning lamp open circuit".
   - The alternator warning line may also be the field-excitation path.
   - The IP holds the odometer, the audible-warning speaker (seat belt, lights-on and SLABS chimes), and the BCU serial-link partner.
   - **Recommendation: keep the original IP connected and powered.** It can be relocated or hidden. Ostler should tap its lines at high impedance. If the IP must go, the replacement has to emulate the lamp loads and meet the fail-safe rules in section 7.
7. **UK law requires a working, illuminated speedometer, a main-beam tell-tale and an indicator tell-tale.** [C]
   - Sources: C&U regs 35/36, MOT manual 7.8 and 4.9.
   - The MOT also fails an ABS warning device that is "not working" (1.6) and an SRS MIL that shows a malfunction (7.1.6).
   - The diesel engine-MIL MOT check applies only to vehicles first used from 1 July 2008, so not to a D2. [C]

---

## 2. What drives the factory instrument pack

Source: RAVE Discovery II, section 88 "Instruments, Description and Operation", pages 1–37
(workshop-manuals.com mirror), plus the repo. Marked [S] unless noted.

| Item | Driver | Signal into the IP |
|---|---|---|
| Speedometer and odometer | SLABS ECU | Pulse train at **8000 pulses/mile** (pin C0504-3 to IP C0230-16; the ECM also gets it at C0658-13). Overspeed lamp table: 165.75 Hz = 77 mph, which works out to about 7750 p/mile. The speedo shows nothing below 1.5 mph. [C: RAVE plus forum repost] |
| Tachometer | Td5 ECM | Digital 12 V square wave, **2 pulses/rev**. At key-on with the engine off, the ECM outputs a 228 rpm-equivalent pulse train (test); the IP ignores anything below 228 rpm. [C: RAVE plus Defender2 "24 Hz at tickover"; one forum post says 4 p/rev, which conflicts]. The repo's Td5 output test `30 B7` "rev-counter" confirms the ECM drives it. [C] |
| Coolant gauge | Td5 ECM | **PWM** duty cycle. Out of range (> 94 % or < 8 %) makes the gauge read cold and lights the hot lamp. The repo's Td5 output test `30 BA` "temp gauge" corroborates. [C] |
| Fuel gauge | Tank sender, read by the IP | Variable resistor, **19 Ω full / 245 Ω empty**. The IP supplies it, and the sender earth **returns to the IP**, not chassis. The gauge is damped, with fast settle within 2.5 s of ignition-on. [S] |
| Low-fuel lamp | IP and ECM | On above 158 ± 8 Ω, off below 113 ± 6 Ω. The IP also sends a low-fuel signal to the ECM. RAVE text is inconsistent about which unit earths the LED. [S] |
| Gear-position LCD (P R N D 3 2 1) | BCU | Over the serial link. [S] The BCU reads the selector W/X/Y/Z switches (BCU input menu, repo). [C] |
| Serial link (BCU / IDM / IP) | BCU, IDM | "Low speed bus … 10 400 bit/s". Used for market configuration, TestBook control, gear display, transfer-neutral lamp and chimes. If comms fail for more than 3 s after ignition-on, the gear display flashes. [S] Whether this is the OBD pin 7 K-line or a separate wire is **unknown**; the repo's pin-7 sniffs never reported unsolicited BCU-to-IP traffic. [I] |

---

## 3. Option A: K-line diagnostics

### 3.1 What each ECU exposes (from the repo plus public projects)

| ECU (address, init) | Useful cluster data | Status |
|---|---|---|
| **Td5 ECM** `0x13`, fast init, `10 A0`, seed/key | Signals listed below. | [C] proven on RDL 016; open-source projects agree (Ekaitza, SimonRafferty, BinOwl, pyTD5Tester) |
| **SLABS** `0x29`, fast init, no session | Wheel speeds `21 43` (u16 ×4; wheel order unconfirmed; scale not km/h). Battery `21 44@12`. Heights `21 54`. Faults `21 11` (logged) and `21 47` (current). **Standstill only. Poll at 1 Hz or slower, or the session dies.** | [C] |
| **BCU** `0x40`, 5-baud | The NanoCom "BCU Instruments" page shows a lamp mirror (listed below), plus "Instr. mileage" and "BCU mileage" and the DCU inputs (lights, indicators, fogs, W/X/Y/Z, transfer neutral). **LID and bit mapping for all of these is still open.** | [C] menu exists; [I] probably reflects IP state via the serial link |
| **EAT** (`72`-framed; possibly generic `0x18`) | Inputs page: gear switches W/X/Y/Z, program switch, high/low range, engine, turbine and output speed. Frame format not decoded. | [S] menu only |
| **SRS** `0x5B`, addressed | Fault records only (`21 02`). Fault `004` = warning-lamp circuit open. No lamp-state LID. | [C] read; [S] meaning |
| **ACE** (address unresolved) | Bulk block at about 1 Hz includes the warning lamp and road speed; unmapped. | [S] |

The Td5 signals proven in the repo:

- `21 09` rpm
- `21 0D` road speed (u8, km/h, 1 km/h resolution)
- `21 1A@0` coolant °C
- `21 1A@12` fuel temp °C
- `21 1A@4` intake air °C
- `21 10` battery V
- `21 1C@0` boost
- `21 1D@6` injection quantity, which feeds a fuel-used computer
- `21 1E` bit 0 = brake pedal
- `21 3B` fault bit block, 211 bits mapped

Still open on the Td5:

- Cruise, clutch and transfer-ratio switch bits in `21 1E`/`21 36`.
- There is **no** fuel-level LID.
- There is **no** lamp-state LID for MIL or glow.

The BCU "Instruments" items are: LH/RH DI, LH/RH trailer DI, seat belt, diff lock, transfer neutral, autobox manual and sport, off-road level, ABS, traction control, SRS, HDC select, glow plug, brake, oil pressure, alternator, check engine, fuel filter (water), transmission temperature, check ACE, check HDC and check SLS.

The BCU route is the only K-line path to most tell-tales, but it has four problems: it is undecoded, it is fragile to init, it cannot share the bus with a Td5 session, and it probably goes blank if the IP is removed.

### 3.2 Realistic Td5 refresh rate

Bus arithmetic at 10 400 baud, 8N1, is about 0.96 ms per byte. Request `02 21 09 2C` is 4 bytes, about 3.8 ms; the echo arrives at the same time. Reply `04 61 09 hh ll cs` is 6 bytes, about 5.8 ms.

| Term | Value | Source |
|---|---|---|
| P2 (ECU response delay) | 25–50 ms (ISO 14230 default). SimonRafferty observes 50–150 ms for simple LIDs and 150–300 ms for composite blocks (`1A`, `1B`, `1C`, `40`). | [S] |
| P3 (inter-request gap) | 25 ms minimum on the Td5, 100 ms recommended (SimonRafferty); ISO default is 55 ms. The repo runs with `p3_min = 0`. | [S] |
| One small transaction | about 60–120 ms | [I] |
| Throughput | about 8–15 transactions/s | [I] |

A dedicated poller schedule could look like this:

- rpm on every other slot
- speed on 1 slot in 3–4
- `21 1A` (temperatures) every 1–2 s
- `21 10` (battery) every 2 s
- `21 3B` (faults) every 5–10 s
- no `3E` keepalive needed, because the stream itself keeps the session alive

That gives **RPM about 4–8 Hz and speed about 2–4 Hz**, with one-sample latency of up to 250 ms or more [I]. The repo's existing node reads the whole LID set at about 1 Hz [C].

By comparison, the tach line runs at 25 Hz at idle and 160 Hz at 4800 rpm. The speed line runs at 11 Hz at 5 mph and 222 Hz at 100 mph. Period capture on these gives a fresh value on every pulse.

### 3.3 Is continuous ECM polling safe while driving?

- **Session survival: yes.** [C] The repo logged real drives, including a motorway run and deliberate overrun lift-offs, at about 1 Hz without session loss.
  - About 1 % of reads lose a whole LID to a comms glitch (TODO "comms_glitch"), so the decoder must tag these and never show them as a sensor fault.
  - The session times out after about 2 s with no traffic [C, repo plus SimonRafferty]. Expect to re-establish after cranking, because of the voltage dip. [I]
- **The ECM does not appear to throttle or drop live data while running.** No source reports it doing so; NanoCom and Hawkeye users log while driving. [S/I]
  - Contrast SLABS, which deliberately suspends diagnostics once the car moves [C].
  - Some composite LIDs return NRCs or fragments if polled too fast [S: SimonRafferty]. Respect P3.
- **Operational safety rules.** [I]
  - Allow-list the service IDs on the driving path to `10`, `27`, `21`, `3E`, `20` and `82` only. Hard-block `30` (IO control: fuel pump, MIL, glow), `31` (routines, including `31 C2` injector cut), `14` and `3B`.
  - A software bug that sends `31 C2 0n` at speed would cut an injector.
  - Ostler must give up the bus on request (release with `82`), so a workshop tool or NanoCom can connect.
  - Ostler must never poll SLABS or the BCU while moving.
- **Conclusion:** K-line is a good *secondary* source for engine values the cluster lacks (°C, boost, battery, faults). It should **not** be the only source for the speedometer or for any safety tell-tale.

---

## 4. Option B: hardwired taps

### 4.1 Design rules

- **Two modes: keep the IP fitted, or replace it.**
  - *IP kept* (recommended): tap every line with high-impedance inputs (≥ 100 kΩ, or a comparator), so the IP keeps working and the ECUs still see their real loads.
  - *IP removed*: Ostler must **provide** what the IP provided.
    - A pull-up and lamp-equivalent load on every low-side line, so ECU lamp drivers don't log open-load faults.
    - The fuel-sender supply.
    - The main-beam bulb load.
    - Possibly the alternator excitation current.
    - The audible warnings.
- **Input protection on every line** (ISO 7637 pulses, load dump up to about 40 V on an old vehicle, reverse battery). [I]
  - A series resistor of 10–47 kΩ.
  - A TVS of about 18–24 V standoff, or clamp diodes to the rails after the series resistor.
  - An RC low-pass filter, with time constant set per signal: about 10 µs for pulse lines, 1–10 ms for lamp lines.
  - Then a Schmitt input or comparator into the 3.3 V MCU domain.
- **Opto-isolation.** It is useful where ground offsets or noise are a concern, and it neatly doubles as the lamp load. [I]
  - **Low-side lines** (ECU sinks to earth): +12 V (ignition) → resistor (about 1–2.2 kΩ, sized to the original LED current) → opto LED → lamp line. When the ECU pulls the line low, the opto conducts. That one part both senses the lamp and gives the ECU its load.
  - **High-side lines** (main beam): line → resistor → opto LED → ground, in parallel with a bulb-equivalent load if the IP is removed.
  - Suitable optos are PC817 or TLP291 for lamp lines, and 6N137 or H11L1 (Schmitt output) for pulse lines.
- **Match the IP's lamp thresholds** in a comparator with hysteresis: on below 1.8 V, off above 7.7 V. Treat 1.8–7.7 V as a **fault** (a short or a pulled lamp), not a state. [S thresholds; I design]
- **Frequency and period capture.** [I]
  - Use MCU timer input capture: STM32 TIMx, ESP32 MCPWM capture or PCNT, or RP2040 PIO.
  - Measure period for low-rate accuracy and average over N pulses at high rate.
  - Set a timeout to show zero: about 0.5 s for speed and about 0.3 s for rpm.
  - Run the front end on an always-on MCU separate from the UI computer, so it is live before Linux boots.
- **Fuel sender, resistance to voltage.** [I]
  - *IP kept*: measure the voltage across the sender **differentially**, because its earth returns to the IP and not to chassis. Use an INA or a differential ADC at high impedance. Calibrate against the 19/245 Ω end points, or compute R by also measuring the IP's feed.
  - *IP removed*: drive the sender ratiometrically, from a 5 V reference through about 1 kΩ (≤ 5 mA, low dissipation in the tank) into the ADC. Or use a constant current of about 2–5 mA.
  - Apply software damping (a long time constant while driving, fast settle for 2.5 s after ignition-on, as the IP does).
  - Low-fuel hysteresis: on above 158 Ω, off below 113 Ω.
- **PWM coolant gauge line**: capture the duty cycle. The duty-to-°C curve isn't published, so calibrate it against K-line `21 1A@0`. Out of range (> 94 % or < 8 %) means a fault. [S/I]

### 4.2 Line types

The table below lists each line's type and the tap circuit that suits it. [S] RAVE, unless marked.

| Line | Type | Tap circuit |
|---|---|---|
| Tach (ECM) | 12 V push-pull square, 2 p/rev | Divider or opto plus Schmitt, then input capture. Ignore below 228 rpm-equivalent. |
| Road speed (SLABS) | Pulse, 8000 p/mile. Output stage (open-collector or push-pull) not documented. | High-Z comparator. If the IP is removed, add a pull-up option so the ECM still gets road speed for cruise. [I] |
| Coolant gauge (ECM) | PWM | Duty-cycle capture |
| Fuel sender | Resistive, 19–245 Ω | As in section 4.1 |
| ABS, TC, HDC info, HDC fault, SLS, off-road (SLABS); ACE (ACE ECU); SRS (DCU); glow, coolant-hot (ECM); oil switch; WIF sensor; trans-temp sensor; diff-lock switch; brake (handbrake switch, fluid switch and SLABS, wire-ORed); trailer, seat belt, indicators, anti-theft (BCU) | **Low-side**: earth path. IP supplies +12 V via fuse 27 or 13. | Pull-up opto, or a high-Z comparator |
| MIL (ECM; also EAT on autos) | "Voltage to IP CPU", same 1.8/7.7 V thresholds | High-Z comparator |
| Alternator charge | Alternator line: on below 2.0 V, off above 7.6 V | High-Z comparator. **If the IP is removed, keep an excitation load** (bulb-equivalent resistor). Verify on the car's alternator. [I] |
| Main beam (IDM) | **High-side** +12 V into a 2 W bulb, earthed through the IP | Divider or opto to ground, plus a bulb load if the IP is removed |
| Manual/sport (EAT) | EAT drives each line high or low. Both low = both lamps flash = EAT fault. | Two comparators |
| Transfer neutral, gear LCD, chimes | **BCU serial link** (no discrete line) | Tap the selector W/X/Y/Z and transfer-neutral switch lines instead, or sniff the serial link with a logic analyser (protocol undocumented). |

---

## 5. Option C: other sources

| Source | Gives | Strengths and limits |
|---|---|---|
| **GPS** (u-blox M8/M9/M10, as in repo test T-31) | Speed, distance, heading | 1–10 Hz with about 0.1–1 s latency and good steady-state accuracy (target ±2 km/h in T-31). Drops out in tunnels, under trees and at start-up (time to first fix). **Good cross-check and fallback for speed and odometer; never the only legal speedometer source.** [I] |
| **IMU / accelerometer** (LSM6DSOX, phone; test T-32) | Longitudinal and lateral g | Can bridge speed through short GPS or pulse gaps (seconds) and detect "moving or stopped". Integrated speed drifts quickly. [I] |
| **Extra fuel-level sender** | Fuel level | Not needed: the existing sender is readable (section 4.1). An extra in-tank sender adds a fuel-system breach. A fuel-used integral from Td5 `21 1D@6` (repo fuel computer, about 7.6 L/100 km validated) makes a good **cross-check** and "range" estimator. [C repo / I] |
| **Battery voltage** (K-line Td5 `21 10` or SLABS `21 44@12`; or a direct ADC) | Charging state | Backs up the charge lamp: engine running and voltage below about 12.8 V means a charge fault. [I] |

---

## 6. Signal-by-signal table

Key:

- **K-line** = yes/no, ECU, ID and the rate achievable while driving.
- **HW tap** = hardwired method.
- **Crit** = safety criticality: **H** must not be missed (drive-safety or legal), **M** important, **L** convenience.

| Signal | Current source / driver | K-line availability | Hardwired tap method | Primary | Fallback | Crit |
|---|---|---|---|---|---|---|
| **RPM** | Td5 ECM → IP, 12 V square, 2 p/rev [C] | **Yes**: Td5 `21 09` u16 rpm [C]; about 4–8 Hz dedicated, 1 Hz today | Tach line: opto or Schmitt, then period capture; ignore below 228 rpm | **HW tach** | Td5 `21 09` | M |
| **Road speed** | SLABS → IP (and ECM), 8000 p/mile [C] | Td5 `21 0D` u8 km/h [C]; about 2–4 Hz. SLABS wheel speeds `21 43` **standstill only** [C] | Speed line: high-Z comparator and period capture; never load the line; never tap the ABS VR sensors | **HW speed pulse** | Td5 `21 0D`, then GPS | **H** (legal speedo, R39 / C&U 35) |
| **Odometer / trip** | IP integrates SLABS distance; stored in the IP, with a copy in the BCU [S] | BCU "Instr. mileage" / "BCU mileage" exist; mapping open [C menu] | Integrate speed pulses (8000/mile) into non-volatile memory, seeded from the IP reading at fit-up | **HW pulse integration** (IP kept, so the IP stays the legal record) | GPS distance | M (legal or sale record) |
| **Fuel level** | Tank sender 19–245 Ω, read by the IP [S] | **No** (no Td5 fuel-level LID) | Differential voltage across the sender (IP kept), or ratiometric drive (IP removed); software damping | **HW sender** | Td5 fuel-used integral (`21 1D@6`) | M |
| **Low fuel** | IP / ECM at > 158 ± 8 Ω [S] | No (the ECM gets a discrete; no LID known) | Derive from sender R with hysteresis 158/113 Ω, or tap the lamp line | **HW sender (derived)** | Lamp line or fuel-used integral | M |
| **Coolant temp** (gauge and hot lamp) | ECM PWM gauge; ECM earths the hot lamp at 121 °C on / 118 °C off [S] | **Yes**: Td5 `21 1A@0` °C [C]; 0.5–1 Hz is plenty | PWM duty capture; hot-lamp low-side tap | **K-line °C** for the gauge; **HW hot-lamp line** for the warning | The other one | **H** (hot lamp) |
| **Oil pressure warning** | Engine oil switch, earth to IP (0.25–0.42 bar diesel) [S] | BCU "Oil pressure" (open) | Low-side tap on the switch line | **HW switch line** | none (BCU mapping open) | **H** |
| **Charge warning** | Alternator line: on < 2.0 V / off > 7.6 V [S] | BCU "Alternator" (open); Td5 `21 10` battery V [C] | High-Z comparator; keep the excitation load if the IP is removed | **HW alternator line** | Battery V while the engine runs | **H** |
| **Glow plugs** | ECM earths the lamp [S] | No lamp LID. Td5 fault `11.6`/`13.6` = glow lamp open load [C/S]. BCU "Glow plug" (open) | Low-side tap | **HW lamp line** | — | L (start aid) |
| **MIL** | ECM (and EAT) voltage to the IP; ECM-driven 3 s self-test [S] | Td5 faults `21 3B` (no MIL bit); BCU "Check engine" (open) | High-Z comparator, 1.8/7.7 V | **HW line** | Td5 fault list (show "faults stored") | M (no MOT check for a pre-2008 diesel) |
| **ABS** | SLABS earths the lamp; 3 s self-check [S] | SLABS faults `21 47`, **standstill only** [C]; BCU "ABS" (open) | Low-side tap | **HW line** | SLABS fault read at standstill | **H** (MOT 1.6 major) |
| **Brake** (handbrake, fluid level, EBD) | Wire-OR earth from the handbrake switch, fluid switch and SLABS [S] | BCU "Brake" (open). Td5 `21 1E` bit 0 = brake **pedal** only [C] | Low-side tap on the shared line | **HW line** | — | **H** |
| **ETC / TC** | SLABS earth [S] | BCU "Traction control" (open) | Low-side tap | **HW** | — | M |
| **HDC info / HDC fault** | SLABS earth (two lamps) [S] | BCU "HDC select" / "Check HDC" (open) | Low-side taps | **HW** | — | M |
| **SLS / off-road** | SLABS earth; 2 Hz flash codes [S] | BCU "Check SLS" / "Offroad level" (open); heights `21 54` standstill only [C] | Low-side taps; decode the 2 Hz flash | **HW** | — | M |
| **ACE** | ACE ECU earth [S] | ACE bulk block "Warning Lamp" (unmapped) [S] | Low-side tap | **HW** | — | M (roll control fault) |
| **Water in fuel** | WIF sensor earth (fed from the fuel-pump relay) [S] | BCU "Fuel filter" (open) | Low-side tap | **HW** | — | M (engine damage) |
| **SRS** | DCU earth; the IP also forces it on if DCU supply < ½ ignition V [S] | SRS `0x5B` fault read (fault `004` = lamp open) [C]; BCU "SRS" (open) | Low-side tap. If the IP is removed, also replicate the supply-monitor rule | **HW line** | SRS fault read (stationary) | **H** (MOT 7.1.6 major) |
| **Indicators L/R** (and trailer) | BCU earth path; trailer lamp by BCU current sensing [S] | BCU "LH/RH DI", "LH/RH Tailor DI" (open) | Low-side taps (two lines plus trailer) | **HW** | — | **H** (C&U / MOT 4.9 tell-tale) |
| **Main beam** | IDM sources +12 V to the bulb, earthed via the IP [S] | BCU DCU "Main beam" (open) | High-side divider or opto; bulb load if the IP is removed | **HW** | — | **H** (MOT 4.9 major) |
| **Front / rear fogs** | BCU-controlled; **not an IP lamp** (tell-tale is on the switch) [I] | BCU DCU "Front/Rear fog" (open) | Optional high-side tap on the lamp feed | **Switch tell-tale stays** | HW feed tap | H for rear fog (MOT 4.9) — covered by the switch [I] |
| **Seat belt** | BCU earth path; 6 s timed with chime [S] | BCU "Seat belt" (open) | Low-side tap; the chime needs the IP speaker | **HW** | — | M |
| **Auto gear position** (P R N D 3 2 1) | BCU → IP **serial link** (from the W/X/Y/Z selector switches) [S] | EAT inputs W/X/Y/Z; BCU DCU W/X/Y/Z (both open) | Tap the four selector switch lines (4-bit code; table to capture) | **HW W/X/Y/Z** | Sniff the serial link | M |
| **Auto mode** (manual / sport) | EAT high/low lines; both flashing = EAT fault [S] | EAT "Program switch" (open) | Two comparators | **HW** | — | M |
| **High / low range** | Transfer high/low switch to EAT / ECM / SLABS; **no IP lamp** [S/I] | Td5 "Transfer Ratio" switch in `21 1E`/`36` (open); EAT "High/Low range"; SLABS `21 42` (open) | Tap the transfer switch line | **HW switch** | Td5 bit once mapped | L |
| **Transfer neutral** | Switch → BCU (analogue) → IP via serial [S] | BCU "Transfer neutral" (open) | Tap the neutral switch line | **HW** | — | M (the car can roll) |
| **Diff lock** | Diff-lock switch earths the lamp [S] | BCU "Diff lock" (open) | Low-side tap | **HW** | — | L |
| **Cruise** | Td5 ECM (no separate module on Td5 [C]); Td5 has a cruise-lamp driver (fault bits `6.0`, `8.1`, `10.0`, `12.0`) [C], but no cruise lamp in the RAVE IP list [S] | Switch bits in `21 1E`/`36` (open); active state not decoded | Tap the cruise lamp output (if wired) or the switches | **K-line once decoded** | HW switch taps | L |
| **Trailer** | BCU earth on extra indicator current [S] | BCU "Tailor DI" (open) | Low-side tap | **HW** | — | M |
| **Anti-theft LED** | BCU earth [S] | BCU immobiliser status [C]; Td5 `31 C0/33 C0` → `03` = not immobilised [C] | Low-side tap | **HW** | K-line status | L |
| *Bonus*: fuel temp, boost, battery, IAT | Td5 | Yes: `21 1A@12`, `21 1C@0`, `21 10`, `21 1A@4` [C] | — | K-line | — | L |

---

## 7. Legal and safety constraints (UK, with EU notes)

### 7.1 Requirements

| Requirement | What it says | Implication for Ostler | Conf. |
|---|---|---|---|
| **C&U Regs 1986, reg 35** | Every motor vehicle must have a speedometer. Vehicles first used from 1 April 1984 must indicate **both mph and km/h** (simultaneously or switchable). Compliance with Directive 75/443 or **UNECE R39** is an alternative. | A D2 must always have a working speedometer. Ostler can be it only if it meets these. | [C] |
| **C&U reg 36** | The speedometer must be kept free of obstruction and "maintained in good working order". | Freezing, lagging or blanking is a defect. | [C] |
| **UNECE R39 accuracy** | Indicated speed must never be below true speed, and not above 110 % of true + 4 km/h: 0 ≤ V1 − V2 ≤ 0.1·V2 + 4 km/h. Annex 4 of recent series covers odometer accuracy. | Calibrate with a small positive bias (about +2–4 %) against GPS. Tyre size changes matter. | [S] secondary source for the formula; [I] for applying it to a retrofit |
| **MOT 7.8 Speedometer** | Not fitted, or not working: **Major**. Not illuminated: **Major**. Impaired or dim: Minor. | The display must be lit and working at the test. | [C] |
| **MOT 4.9 Tell-tales** | Main-beam tell-tale (first used from 1 April 1986) missing or inoperative: **Major**. Direction-indicator tell-tale (visual **or** audible): Minor. Hazard tell-tale must flash. Rear-fog tell-tale: **Major**. | Main beam and indicators must always show. Rear fog stays on the switch. | [C] |
| **MOT 1.6 ABS** | Warning device **not working**, or showing a malfunction: **Major**. | The ABS lamp must come on in the SLABS 3 s bulb check and go off. | [C] |
| **MOT 1.1.10 Brake fluid lamp** | Lit or defective: Minor. | Mirror the brake line. | [C] |
| **MOT 7.1.6 SRS MIL** | Fails only if it **indicates** a malfunction. A lamp that never lights "cannot be failed", per the DVSA blog, though hiding it defeats the safety purpose. | Show it faithfully, including the bulb check. | [C] |
| **MOT 8.2.2.2 Engine MIL (diesel)** | Applies only to vehicles first used **on or after 1 July 2008**. | Not an MOT item for a D2, but still worth showing. | [C] |
| **UNECE R121** (tell-tale symbols and colours) | Type-approval rule for new M/N vehicles, colour code per ISO 2575: red = danger, amber = caution, green = on, blue = main beam. | Not retroactively binding on an in-service 1999–2004 car, but it is the reference design for symbols and colours. Follow it. | [I] |
| **Road Vehicles Lighting Regs 1989** | Require tell-tales for main beam, indicators (may be audible) and rear fog. | Same as MOT 4.9. | [I] (from MOT cross-references; schedules not re-read) |
| **Odometer** | The IP is the record; MOT certificates log mileage; a mileage discrepancy is flagged. Selling with a misleading mileage is an unfair commercial practice (CPRs 2008). | Keep the IP's odometer running, or document the reading at the changeover and keep Ostler's count authoritative and non-resettable. | [I] |
| **Insurance / EU** | A modification must be declared to the insurer. In Germany and similar regimes, replacing a speedometer generally needs inspection approval (StVZO §57 / Einzelabnahme). | Check locally before relying on Ostler as the only speedometer. | [I] |

### 7.2 Recommended architecture: keep the IP

- **Keep the original IP connected and powered**, wherever it physically sits.
  - It stays the legal, type-approved speedometer and odometer.
  - It keeps the lamp loads, so there are no new Td5/SRS/EAT open-load faults.
  - It keeps the BCU serial-link partner, so there is no gear-display or link-loss behaviour.
  - It keeps the chimes, the fuel-sender supply and the alternator excitation.
- **Ostler becomes an additional display.** Feed it from a high-impedance tap harness on the IP connectors (C0229/C0230/C0231 region; pinout to be verified against the RAVE ETM) plus the Td5 over K-line.
- **This is the lowest-risk route, and it is legal without argument.** [I]

### 7.3 If the IP must be removed: fail-safe requirements

All [I].

1. **A hardware tell-tale path independent of the UI software.** An MCU front end, or ideally discrete LED drivers, fed directly from the lamp lines.
   - At minimum: ABS, brake, SRS, oil, charge, coolant-hot, indicators, main beam and MIL.
   - These lamps should light even if the Ostler computer is hung or still booting.
2. **Fast start.** The tell-tale and speed layer must be live within under 1 s of ignition-on, so the ECU-driven 3 s bulb checks (ABS, SRS, SLABS lamps, coolant, MIL) are actually visible. That is what the MOT tester and driver check.
   - A Linux UI that boots in 10–30 s cannot carry this on its own.
3. **The speedometer fails loudly.**
   - A watchdog blanks the speed and shows "SPEEDO FAULT" if pulses and the K-line disagree, or the source goes stale for more than 0.5–1 s. It must never freeze a last value.
   - Pulses are primary, with Td5 `21 0D` and GPS as plausibility checks.
   - Never under-read (R39). Show mph and km/h. The display must be illuminated.
4. **Load emulation.** Every low-side lamp line needs a pull-up and an LED-equivalent load, which a pull-up opto provides.
   - Main beam needs a bulb-equivalent load.
   - The charge line keeps its excitation path.
   - The road-speed line keeps any pull-up the IP provided, because the ECM uses that speed for cruise.
   - Then re-read the Td5, SRS and EAT faults to confirm that no lamp-open faults appear.
5. **Odometer continuity.**
   - Record the IP reading (photo, and BCU "Instr. mileage" once decoded) at removal.
   - Store distance in wear-levelled non-volatile memory, with no user reset of the total.
   - Expect the BCU "odometer error warn" behaviour (BCU setting; "NOT FITTED" on RDL 016) to need checking.
6. **Replicate IP-internal logic.**
   - SRS lamp on if the DCU supply is below ½ of the ignition voltage.
   - Low-fuel hysteresis.
   - The coolant gauge reads cold and the hot lamp lights on out-of-range PWM.
   - Audible indicator, seat-belt and lights-on warnings: the D2 uses the IP speaker for these.
7. **Audit the K-line driving path.** Allow-list read-only SIDs, touch no SLABS or BCU while moving, and release the bus for workshop tools.

---

## 8. Open items worth a car test

- Confirm the tach is 2 p/rev on the scope: 25 Hz at 750 rpm.
- Confirm the speed line's output-stage type and voltage levels.
- Probe the BCU / IP serial link: is it pin 7 or a separate wire, and what is its frame format? It would carry gear and transfer-neutral state.
- Decode the BCU "Instruments" LIDs with the NanoCom capture (repo T-25). That gives a stationary cross-check of every tell-tale over one session.
- Capture the W/X/Y/Z code table for each selector position.
- Measure the IP lamp-LED current per line, to size the emulation loads.
- Measure the fuel sender at full and empty on the car, to confirm 19/245 Ω.
- Check whether the Td5 cruise-lamp output is wired to anything on this car.

---

## Sources

Repository (this repo (`JamesWrightDavid/discovery2-diag`), read-only):

- `docs/discovery-2-td5/system-map.md`, `engine-td5.md`, `slabs.md`, `bcu.md`, `cruise-control.md`, `kline/session-lifecycle.md`, `kline/physical-and-init.md`
- `references/protocol_state_handoff.md`, `references/td5_fault_codes.md`, `docs/discovery-2-td5/fault-dictionary-td5.md`, `references/airbag_fault_codes.md`
- `references/slabs/overview.md`, `references/wabco_slabs_capabilities.md`, `references/valeo_bcu_capabilities.md`, `references/test-plan-resolved.md` (SLABS standstill-only, 2026-08-29; T-26 address sweep)
- `references/menus/bcu-inputs.md` (BCU Instruments page), `references/menus/autobox.md`, `references/menus/td5.md`, `references/d2-tool-cross-reference.md`, `references/td5-d2-ecosystem.md`, `TODO.md` (comms_glitch), `esp32/kline_node/kline_node.ino`, `hardware/README.md`

RAVE Discovery II "Instruments, Description and Operation", workshop-manuals.com mirror:

- https://workshop-manuals.com/landrover/discovery-ii/instruments/description_and_operation/description/ (dials, lamp packs 1–2)
- https://workshop-manuals.com/landrover/discovery-ii/instruments/description_and_operation/description/page_1754/ (lamp packs 3–4, speedometer, 8000 p/mile, SLABS source)
- https://workshop-manuals.com/landrover/discovery-ii/instruments/description_and_operation/description/page_1755/ (odometer, gear display from the BCU)
- https://workshop-manuals.com/landrover/discovery-ii/instruments/description_and_operation/description/page_1756/ (tachometer: ECM, 2 p/rev, 228 rpm test)
- https://workshop-manuals.com/landrover/discovery-ii/instruments/description_and_operation/description/page_1757/ and https://workshop-manuals.com/landrover/discovery-ii/instruments/description_and_operation/description/page_1758/ (fuel gauge and sender: 19/245 Ω, 158 Ω low-fuel threshold)
- pages 1759–1787 in the same path (per-lamp sources, the 1.8/7.7 V thresholds, bulb checks, overspeed 165.75 Hz, oil switch, alternator 2.0/7.6 V, SRS supply rule, SLABS lamps, EAT manual/sport, anti-theft)
- https://workshop-manuals.com/landrover/discovery-ii/instruments/description_and_operation/description/page_1788/ (audible warning speaker) and https://workshop-manuals.com/landrover/discovery-ii/instruments/description_and_operation/description/page_1789/ (gear display flashes on comms loss)
- https://workshop-manuals.com/landrover/discovery-ii/instruments/description_and_operation/instrument_block_diagram/
- https://workshop-manuals.com/landrover/discovery-ii/body_control_unit/description_and_operation/description/ (BCU / IDM / IP datalink, 10 400 bit/s)

Forums and community:

- https://www.aulro.com/afvb/electronic-diagnostic-systems/177007-td5-conversion-slabs-speedo-signal.html (SLABS speed signal, pins)
- https://landroverforums.com/forum/discovery-ii-18/p0500-code-damn-44486/ (8000 p/mile, SLABS speed)
- https://landroverforums.com/forum/discovery-ii-18/disco-2-no-tach-100437/ (tach from the ECM; a 4 p/rev claim that conflicts)
- https://landroverforums.com/forum/discovery-ii-18/temp-fuel-gauge-issue-46956/
- https://www.defender2.net/forum/post721794.html (Td5 tach 12 V square wave, 24 Hz at idle, 2 p/rev)
- https://www.defender2.net/forum/post751171.html (BCU / IP serial link: no public spec)
- https://www.aulro.com/afvb/discovery-2-a/235517-dash-warning-lights-query.html
- https://www.blackbox-solutions.com/site/nanocom (Td5 cruise run by the ECM)
- https://www.nanocom-diagnostics.com/uploads/downloads/Discovery%20II%20Valeo%20BCU%20ECU%20Guide.pdf

Open-source Td5 projects:

- https://github.com/SimonRafferty/Land-Rover-Td5-Arduino-Diagnostics (TD5_PROTOCOL_REFERENCE.md: P2/P3, keepalive, LIDs)
- https://github.com/SimonRafferty/Td5-Diagnostic-App
- https://github.com/EA2EGA/Ekaitza_Itzali
- https://github.com/k0sci3j/BinOwl_Td5Gauge
- https://github.com/BennehBoy/LRDuinoTD5
- https://github.com/hairyone/pyTD5Tester
- https://github.com/pajacobson/td5keygen

Law and standards:

- https://www.legislation.gov.uk/uksi/1986/1078/regulation/35/made (C&U reg 35, speedometer)
- https://www.legislation.gov.uk/uksi/1986/1078/regulation/36 (C&U reg 36, maintenance)
- https://www.gov.uk/guidance/mot-inspection-manual-for-private-passenger-and-light-commercial-vehicles/1-brakes (1.1.10, 1.6 ABS)
- https://www.gov.uk/guidance/mot-inspection-manual-for-private-passenger-and-light-commercial-vehicles/4-lamps-reflectors-and-electrical-equipment (4.9 tell-tales)
- https://www.gov.uk/guidance/mot-inspection-manual-for-private-passenger-and-light-commercial-vehicles/7-other-equipment (7.1.6 SRS, 7.8 speedometer)
- https://www.gov.uk/guidance/mot-inspection-manual-for-private-passenger-and-light-commercial-vehicles/8-nuisance (8.2.2.2 diesel MIL, from 1 July 2008)
- https://mattersoftesting.blog.gov.uk/common-mistakes-made-by-mot-testers/ (SRS MIL guidance)
- https://eur-lex.europa.eu/eli/reg/2018/1857/oj/eng (UNECE R39 text, EU OJ)
- https://attachments.priuschat.com/attachment-files/2020/10/195739_R039r2e.pdf (R39 copy)
- https://www.wapcar.my/news/your-speedometer-is-lying-to-you-here’s-why-26765 (R39 tolerance explained, secondary)
- https://eur-lex.europa.eu/eli/reg/2016/18/oj (UNECE R121, tell-tales)
- https://globalautoregs.com/documents/14168 (R121 amendment history)
