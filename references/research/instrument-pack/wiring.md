---
title: "D2 instrument pack: connectors, inputs and wiring (C0230/C0233)"
area: references
status: stable
version: 1.0
updated: 2026-10-14
summary: >
  Pin-by-pin C0230 (20-way) and C0233 (16-way) with wire colours, gauge inputs (tach 2 p/rev, speed 8000 p/mile, coolant PWM, fuel sender 19–245 Ω floating), every tell-tale and who drives it (mostly low-side sense, <1.8 V on / >7.7 V off; main beam high-side; indicators, trailer, seat belt, gear and neutral only on the 10 400 bit/s SK bus to the BCU/IDM), what the pack holds (odometer mirrored in the BCU, chime speaker, BCU road speed) and what breaks if it is removed. Every fact tagged confirmed / single source / inferred.
---

# Discovery 2 (L318, 1999–2004) instrument pack: connectors, inputs and wiring

Scope: Td5 first, with V8 differences noted. Covers pre-2003 (99MY–02MY) and 2003MY+ "D2a" packs.
Goal: enough detail to replace the pack with a custom display that reads the same signals.

## Confidence key

| Tag | Meaning |
|---|---|
| **[C2]** | Confirmed by two or more sources. Several of these are separate Land Rover publications: the Electrical Library, the Circuit Diagrams and the Workshop Manual. |
| **[S1]** | Single source. |
| **[INF]** | Inferred by me from the sources. It is not stated by any of them, so verify it on the car. |

Source IDs (S1…S17) are listed under **Sources** at the end. Wire colour codes (Land Rover):
B black, G green, K pink, L light, N brown, O orange, P purple, R red, S slate/grey, U blue,
W white, Y yellow. The main colour comes first and the tracer second, so "WN" is white with a brown tracer.

---

## 1. Connectors

The pack has **two connectors**. Both are on the back of the pack and both are black female housings:

| ID | Ways | Colour | Location | Source |
|---|---|---|---|---|
| **C0230** | 20 (cavity 2 unused) | Black, female | Behind instrument pack | [C2] S1 and S2 connector-details tables |
| **C0233** | 16 (cavity 9 unused before 03MY) | Black, female | Behind instrument pack | [C2] S1, S2 |

> C0229 is **not** the pack. It is the fascia/main-harness interface (the radio/ICE pins 17–25 and
> 39–40, mated with C0067). C0230-16's road-speed wire passes through C0229-5/C0067-5 on the way
> from header C0290 [S1 connector table, S3]. The pack wires travel through the in-line connector
> **C0047/C0223** (fascia ↔ main harness, under the steering column). The C0047 pin numbers below come from
> the 03MY LHD circuit diagram [S1-only: S3]. One owner reports the in-line connector colours/layout
> differ from the manual [S11].

Workshop manual pack removal: "Disconnect 2 multiplugs from instrument pack" [S5 repairs page].
That agrees with the two connectors above [C2].

### 1a. C0230 (20-way): pin by pin

Cavity colours come from the connector-details tables. They are identical in the 05/99 library (S2) and the 03MY library (S1).

| Pin | Wire | Function | Far end | Conf. |
|---|---|---|---|---|
| 1 | PW | Driver's door switch input (door-open state). It is also used with SLABS remote-control modes. | Header C0287 → driver's door lock/switch (M117 on LHD) and SLABS C0655-1; BCU C0660-17 shares the door switch line | [C2] S2 text, S3 diagram. Function within the pack: [S1] S2 |
| 2 | — | Not used (no cavity listed) | — | [C2] S1, S2 |
| 3 | US | **Main beam warning lamp** feed. The IDM high-side switches +12 V to the lamp and the lamp earths inside the pack. | IDM (passenger fusebox) | [C2] S1 text, S5 |
| 4 | YR | **Sport mode lamp** sense. The EAT pulls it low. | EAT ECU C0193-25 | [C2] S1, S3, S5; S11 measured about 10 V idle |
| 5 | YG | **Manual mode lamp** sense. The EAT pulls it low. | EAT ECU C0193-51 | [C2] S1, S3, S5, S11 |
| 6 | WN | **Oil pressure lamp** sense. The switch closes to earth. | Oil pressure switch S216 (Td5 via C0448-1/C0162-1) | [C2] S1, S3, S5 |
| 7 | BLG | **Transmission oil temp lamp** sense. A switch closes to earth. | Header C0293 → gearbox oil-cooler switch S230 (C0219-2) and transfer-box switch S296 (C0686-1) | [C2] S1, S3, S5 |
| 8 | BU | **Diff-lock lamp** sense. The switch closes to earth. | Header C0761 → diff-lock switch(es). The same switch also signals SLABS. | [C2] S1, S3, S5 |
| 9 | PK | **Off-road mode lamp** sense. SLABS pulls it to earth. | Splice A186 → off-road switch S308 and SLABS C0655-11 | [C2] S1, S3 |
| 10 | KO | **Brake / handbrake / fluid lamp** sense, pulled to earth. | Header C0293 LHD/C0285 RHD → handbrake switch S213, fluid-level switch S212 **and SLABS C0504-9** | [C2] S1, S3, S5 |
| 11 | NY | **Charge lamp**, which is also the **alternator excitation (D+/IND) feed** | Td5: header C0294 LHD/C0287 RHD → alternator; also Td5 FBH air-temp sensor | [C2] S1, S3, S5 |
| 12 | RN | **Panel illumination +** (side-lamp feed; dimmed feed on NAS/Japan) | Header C0759 (fuse 33 / dimmer rheostat) | [S1] S1 cavity colour plus S1 illumination text (RN from C0759). The pin function is my reading: [INF], strong |
| 13 | KB | **Fuel sender return.** The pack provides the sender's earth. | Fuel tank unit C0114-3 (via C0047-10, C0378-12/C0392-12) | [C2] S1, S3, S5 |
| 14 | GB | **Fuel gauge / sender feed** (pack → sender) | Fuel tank unit C0114-2 (via C0047-21, C0378-13/C0392-13) | [C2] S1, S3, S5 |
| 15 | WS | **Tachometer input** (engine-speed pulses) | Header C0291 → Td5 ECM C0658-19 (V8 ECM C0637-17). The same WS net also feeds SLABS C0655-10. | [C2] S1, S3. S1 text misprints it as "C0235-15". |
| 16 | KG | **Road speed input** (speedo, odometer, overspeed) | Header C0290 (plus splice A87 on ATC cars) → SLABS ECU C0504-3 | [C2] S1, S3, S5 |
| 17 | B | **Earth** | Earth header C0551 (C0047-11) | [C2] S1, S3, S12 |
| 18 | B | **Earth** | Earth header C0551 (C0047-13) | [C2] S1, S3, S12 |
| 19 | LG | **Ignition +12 V** (pos. II), fuse 27 (10 A, passenger box) via header C0760. It is the common feed for most lamps and the gauges. | Passenger fusebox F27 | [C2] S1, S3, S5, S12 |
| 20 | PO | **Permanent battery +12 V**: engine-bay F13 (10 A) → passenger box (PN) → PO. It powers the sounder, anti-theft LED and "fuse 13" lamps. | Passenger fusebox | [C2] S1 text (PO wire) and S1 fuse table (F13 "instrument pack"). The use for the sounder and LED is my reading: [INF] from S5 ("speaker can operate with ignition off", anti-theft LED "via fuse 13") |

### 1b. C0233 (16-way): pin by pin

| Pin | Wire | Function | Far end | Conf. |
|---|---|---|---|---|
| 1 | BO | **ACE lamp** sense (pulled low) | ACE ECU C0647-36 | [C2] S1, S3, S5 |
| 2 | WO | **SLS lamp** sense (pulled low) | SLABS C0655-8 | [C2] S1, S3, S5 |
| 3 | WR | **ABS lamp** sense (pulled low) | SLABS C0504-18 | [C2] S1, S3, S5 |
| 4 | WU | **Traction control lamp** sense (pulled low) | SLABS C0504-13 | [C2] S1, S3, S5 |
| 5 | WY | **HDC active (green) lamp** sense | SLABS C0504-16 | [C2] S1, S3 |
| 6 | G | **Ignition +12 V, fuse 35** (10 A, shared with SRS DCU). It supplies the SRS lamp. | Passenger fusebox F35 | [C2] S1 table and text, S3. The 05/99 text wrongly says "C0233-3". |
| 7 | YR | **SRS lamp** sense. The airbag DCU sinks it, and the DCU also monitors the lamp circuit. | Airbag DCU C0256-13 | [C2] S1, S3, S5 |
| 8 | BS | **Anti-theft LED** sense. The BCU sinks it. | BCU C0661-20 | [C2] S1, S3, S5 |
| 9 | K | **03MY+ only. Road-speed output to the satellite-nav head unit** (nav and auto volume control) | Splice A198 → head unit C0098-1 and accessory-power connector C0942-5 | Pin present only in the 03MY table: [C2] S1 vs S2. Connection: [S1] S3 (nav page). Function: [S1] S1 nav text, which says "road speed … input to the head unit on an O wire from the instrument pack". The colour conflict (O vs K) is unresolved. |
| 10 | WK | **HDC fault (amber) lamp** sense | SLABS C0504-17 | [C2] S1, S3 |
| 11 | OG | **Water-in-fuel lamp** sense (Td5) | Water-in-filter sensor (powered from the fuel-pump relay, WP) | [C2] S1, S3, S5 |
| 12 | BY | **Glow plug lamp** sense (Td5). The ECM sinks it. | Td5 ECM C0658-30 | [C2] S1, S3, S8 |
| 13 | RS | **MIL** sense. The ECM sinks it. | Td5 ECM C0658-6 / V8 ECM C0637-20 | [C2] S1, S3, S7, S8 |
| 14 | GS | **Low-fuel-level output, pack → V8 ECM** (misfire-diagnosis plausibility). Not connected on Td5. | V8 ECM C0637-8 ("Low fuel level, input, active high") | Wiring: [S1] S3. Function: [C2] S5 fuel page and S7 (ECM pin 8 "low fuel level"). That this is the pin is my reading: [INF], strong |
| 15 | GU | **Coolant temperature gauge input** (PWM from the ECM) | Td5 ECM C0658-7 / V8 ECM C0636-44 | [C2] S1, S3, S5, S7, S8 |
| 16 | SK | **Serial data bus** (bidirectional, single wire) to the BCU and IDM | Header C0293 LHD/C0292 RHD → BCU C0661-1 and IDM (passenger fusebox) | [C2] S1, S3, S5, S6 |

### 1c. Pre-2003 vs 2003MY+ differences

| Item | Pre-03MY | 03MY+ | Conf. |
|---|---|---|---|
| C0233 cavity 9 | Absent | K wire: road speed to the nav head unit | [C2] S2 vs S1 tables; S3 |
| Diff-lock lamp | Red LED; the lock is for rolling-road testing only | Amber LED, top-left; the pack sounds **3 chimes** on engage and disengage; diff-lock switch**es** | [S1] S5 (diff-lock page) |
| All other C0230/C0233 cavities and colours | Same | Same | [C2] S1, S2 |
| Plug-in swap between eras | — | Forum reports say it is a straight physical swap. The odometer then flashes until it is synced with the BCU (TestBook/NanoCom); mileage reportedly syncs upward only. | [S1] S14. These are forum search snippets only; the pages returned 403. |
| Market variants (dial faces/units) | UK, NAS, Canada, Australia, ROW/Gulf/Japan | Same | [S1] S5 |

The physical 03MY face or dial-artwork differences were not confirmed (see Gaps).

---

## 2. Gauge inputs

| Gauge | Pin | Source | Signal | Scaling / levels | Conf. |
|---|---|---|---|---|---|
| **Speedometer** and odometer/trip | C0230-16 (KG) | SLABS ECU C0504-3. The same KG net also feeds the ECM (Td5 C0658-13 / V8 C0637-22), ACE C0647-5, ICE amp C0491-20, ATC C0792-2 and the cruise ECU (V8) | Square-wave "PWM", switched 0–12 V | **8000 pulses per mile** (≈4971 pulses/km; 2.22 Hz per mph). **2 Hz reference at standstill.** The speedo shows nothing below 1.5 mph (2.5 km/h). | 8000 ppm: [C2] S5, S7. 0–12 V and 2 Hz: [S1] S7. Fan-out: [S1] S1 (SLABS section) |
| **Tachometer** | C0230-15 (WS) | Td5 ECM C0658-19 (V8 C0637-17). Shared with SLABS C0655-10. | Digital pulse train | **2 pulses per engine revolution**. With ignition on and engine stopped, the ECM emits pulses equal to 228 rpm (diagnostic); the gauge ignores readings below 228 rpm. Full scale is 6000 rpm, and the same gauge is used on V8 and Td5. Level: V8 pin table says "PWM 0–5 V"; one Td5 owner reports a 12 V pulse. | 2 ppr: [C2] S5, S9, S10. 228 rpm: [S1] S5. V8 0–5 V: [S1] S7. Td5 12 V: [S1] S15. One forum claims early Td5 ECUs give 1 ppr: [S1] S16, low confidence |
| **Fuel gauge** | C0230-14 (GB, feed out) and C0230-13 (KB, return) | Tank unit (sender combined with the pump), D179 / J111 | Variable resistor. **The sender floats: its earth is the pack (KB), not the chassis.** The pack feeds the gauge from F27 ignition. | **Empty 245 Ω, Full 19 Ω.** Low-fuel lamp ON above 158 ± 8 Ω (pointer 8°), OFF below 113 ± 6 Ω (pointer 20°). Gauge is damped, reaches its reading within 2.5 s of ignition-on and returns to 0 with ignition off. | Wiring: [C2] S1, S3, S5. 245/19 Ω and lamp thresholds: [S1] S5. The 158 Ω value is also in S7: [C2]. A forum says ≈15 Ω full [S1] S13, and a D1 V8 is quoted at 19–275 Ω (not D2) |
| **Coolant temperature** | C0233-15 (GU) | **ECM output** (Td5 C0658-7, V8 C0636-44). The ECM reads its own ECT sensor; there is no separate gauge sender on D2. | PWM 0–12 V | V8: pulse width **768 µs at −40 °C … 4848 µs at 140 °C**. Out of range (> 94 % duty when hot, < 8 % when cold) makes the gauge read cold and turns on the high-temp lamp. High-temp lamp duty thresholds: **Diesel ON 78.8 ± 2 %, OFF 76.8 ± 2 %**; V8 77.8/75.7 % (Gulf 79.8/77.8 %). The lamp corresponds to 121 °C on and 118 °C off. | ECM-driven PWM: [C2] S5, S7, S8. µs values: [S1] S7 (V8). Duty thresholds: [S1] S5. **Td5 PWM frequency and scaling: not found** |

The Td5 ECM checks its gauge and lamp outputs for faults. Logged faults include "tachometer open
load", "temperature gauge open load / short", "MIL lamp drive open load" and "glow plug lamp drive
open load" [S17: fault-dictionary-td5.md, proven on cars]. **A replacement display must present a
load or pull-up on these lines, or the Td5 will log faults** [INF from S17 and S8 ("enabling the
tachometer option without a tachometer fitted will show an open load fault")].

---

## 3. Warning lamps and tell-tales: who drives them

**General electrical model.** Almost every lamp is an LED powered *inside* the pack. The
pack's microprocessor reads a **sense line**. The external module or switch **pulls the line to earth**,
so the external side is low-side. The thresholds are **< 1.8 V = lamp ON, > 7.7 V = lamp OFF**.
For the charge lamp they are < 2.0 V and > 7.6 V. [C2] S5 (stated on most lamp pages) and S11 (owner measured about 10 V
on the idle sport and manual lines, which dropped to 0 V when active). This means the pack provides a pull-up on each
sense pin [INF]. Lamps are rated **1.2 W** on the circuit diagram; main beam is a **14 V 2 W bulb** [S1] S3, S5.
There are 28 lamps in four "warning lamp packs"; legends are invisible until lit [S1] S5.

"Self-check" below means the ignition-on bulb check and who performs it.

| Lamp (colour) | Pin / path | Driver (who switches) | Side | Pack-side supply | Self-check | Conf. |
|---|---|---|---|---|---|---|
| Oil pressure (red) | C0230-6 WN | Oil pressure switch to earth (Td5 opens at 0.25–0.42 bar; V8 0.22–0.59 bar) | Low | F27 | None performed; the lamp is naturally lit with the engine stopped | [C2] S1, S5 |
| Charge / ignition (red) | C0230-11 NY | Alternator. Its regulator and brushes pull the line low when not charging, and alternator output above 7.6 V turns the lamp off. **This line also excites the alternator.** | Low (sense) | F27 | Lit until the engine runs | [C2] S1 (starting/charging), S5 |
| Glow plug (amber, Td5) | C0233-12 BY | Td5 ECM C0658-30 | Low | F27 | ECM-controlled (on during preheat) | [C2] S1, S5, S8 |
| MIL / "Service Engine Soon" (amber) | C0233-13 RS | ECM (Td5 C0658-6, V8 C0637-20). The EAT faults also light it via the ECM. | Low | F27 | **ECM**: on 3 s, then off. If a fault is present it goes off for 1 s, then on. | [C2] S5, S7, S8 |
| High coolant temp (red) | Internal (from the C0233-15 PWM) | The pack decodes the PWM duty (see §2). The S5 text says the ECM controls the earth, but the diagram shows no separate pin. | — | F27 | S5 says the ECM performs a 3 s check | Lamp internal to the pack: [S1] S3 diagram. Thresholds: [S1] S5. The two descriptions conflict |
| Low fuel (amber) | Internal (from sender resistance) | Pack logic (158/113 Ω) | — | F27 | 3 s | Internal pack logic: [C2] S1, S3. The S5 lamp page says "ECM illuminates", which contradicts S5's own fuel page and S1; I treat it as an error |
| Water in fuel (amber, Td5) | C0233-11 OG | Water-in-filter sensor output | Low (sensor output below threshold) | F27 | **Pack**, 3 s | [C2] S1, S5 |
| ABS (amber) | C0233-3 WR | SLABS C0504-18 | Low | F27 | **SLABS**, 3 s | [C2] S1, S3, S5 |
| Brake / handbrake / low fluid (red) | C0230-10 KO | Handbrake switch, fluid-level switch, **or SLABS C0504-9** (brake system fault) | Low | F27 | **Pack**, 3 s | [C2] S1, S5 |
| Traction control (red per S5) | C0233-4 WU | SLABS C0504-13 | Low | F27 | **SLABS**, 3 s | [C2] S1, S5 |
| HDC active (green) | C0233-5 WY | SLABS C0504-16 | Low | F27 | **SLABS**, 3 s | [C2] S1, S5 |
| HDC fault (amber) | C0233-10 WK | SLABS C0504-17 | Low | F27 | **SLABS**, 3 s | [C2] S1, S5 |
| SLS (amber) | C0233-2 WO | SLABS C0655-8. Flashes at 2 Hz with remote raise/lower or transport mode; lit when the supply is under 10 V. | Low | F27 | **SLABS**, 3 s (coil-spring cars go out after 850 ms) | [C2] S1, S5 |
| Off-road mode (amber) | C0230-9 PK | SLABS C0655-11. The same line is the off-road switch input: SLABS earths it and pulses it to re-read the switch. Flashes at 2 Hz during height transitions. | Low | F27 | **SLABS**, 3 s | [C2] S1, S5 |
| ACE (amber) | C0233-1 BO | ACE ECU C0647-36 | Low | F13 (permanent) | **ACE ECU**, 3 s | [C2] S1, S5 |
| Diff lock (red before 03MY, amber 03MY+) | C0230-8 BU | Diff-lock switch to earth | Low | F13 (S5) | None stated | [C2] S1, S5 |
| Transmission oil temp (red, auto) | C0230-7 BLG | Oil-temp switches to earth | Low | F13 | **Pack**, 3 s | [C2] S1, S5 |
| Sport / Manual (green, auto) | C0230-4 YR / C0230-5 YG | EAT ECU. Truth table: both high = off; sport low = Sport; manual low = Manual; **both low = both flash (EAT fault)** | Low | Pack | None | [C2] S1, S5 |
| SRS / airbag (red) | C0233-7 YR (supply C0233-6 G, F35) | Airbag DCU C0256-13. **The pack also forces the lamp on if the DCU supply is below half the ignition voltage.** The DCU detects 0 V on YR as a lamp failure and stores a fault. | Low | F35 | **DCU**: 3 s (S5) / 3–5 s (S1) | [C2] S1, S5 |
| Anti-theft LED (red) | C0233-8 BS | BCU C0661-20 (< 1 V when armed) | Low | F13 (permanent) | S5 says the "ECM" self-checks it until engine start, probably meaning the BCU | [C2] S1, S5 |
| Main beam (blue, 2 W bulb) | C0230-3 US | **IDM high-side** +12 V feed; the bulb earths in the pack | **High** | — | None | [C2] S1, S5 |
| Direction indicators L/R (green) | **Serial bus** C0233-16 | BCU/IDM messages. The fast flash on bulb failure is sent over the bus. | Bus | F27 | None | [C2] S1, S5, S6; S10 (DIY users needed board hacks for these) |
| Trailer (green) | **Serial bus** | IDM/BCU (current sense on indicator circuit) | Bus | F13 | **BCU**, 3 s every ignition-on | [C2] S1, S5, S6 |
| Seat belt (red) | **Serial bus** | BCU (buckle switch on BCU C0662-16); 6 s timer options. **The pack self-times the lamp and chime if the link fails.** | Bus | F13 | BCU, 6 s | [C2] S1, S5, S6 |
| Transfer box neutral (amber; NAS/Canada/Japan only) | **Serial bus** | BCU (transfer neutral switch) plus chime | Bus | Pack | None | [C2] S1, S5, S6 |
| Overspeed (amber; **Gulf only**) | Internal / bus | **Conflict.** S1 says the pack software sets it from the C0230-16 speed at 78/74 mph. S5 says it is BCU-illuminated at 77/74 mph (165.75/158.83 Hz). | — | Pack | None | [S1] each; unresolved |
| Gear selected PRND321 (LCD) | **Serial bus** | BCU (from the gear-selector switch). The LCD flashes if diagnostic/serial comms fail for more than 3 s. | Bus | — | — | [C2] S5, S6 |
| Low range / high range | **No tell-tale in the pack** | — | — | — | — | [S1] S5 lamp list (32 items, none for range). Absence is [INF] |
| Fog lamps (front/rear) | **No tell-tale in the pack.** The indicators are in the fascia switches. | Switch indicator lamp | — | — | — | [C2] S1 fog lamp sections, S10 ("no fog light warning in the cluster") |
| Cruise control | **No tell-tale in the pack.** The lamp is in the cruise switch. | Switch indicator lamp | — | — | — | [S1] S1 (cruise section); the S5 lamp list has no cruise lamp. Note the Td5 ECM does have a "cruise lamp drive" output with fault codes [S17]; its D2 use is unknown |
| Audible warning (sounder in pack, 87 dB) | Bus requests (BCU, SLABS via BCU, transfer neutral) | Requests: 0 off, 1 seat belt, 2 key-in, 3 SLABS/HDC, 4 transfer neutral, 5 lights-on. Works with ignition off. | Bus | Pack | — | [C2] S1 (lights-on alarm), S5, S6 |

---

## 4. Pack internals and dependencies

| Question | Finding | Conf. |
|---|---|---|
| Does the pack have logic? | Yes. A microprocessor drives the 4 air-cored/damped dials and an LCD (odometer 0–999 999, trip 0.0–999.9, PRND321). A photocell controls LCD contrast, and the 28 lamps are mostly LEDs. Inputs are "digital, analogue, pulse train, PWM". | [S1] S5 |
| Bus link | **Single-wire bidirectional serial bus** on C0233-16 (SK) shared by the **BCU (C0661-1)**, **IDM** and pack, at **10 400 bit/s**. This is not the pin-7 diagnostic K-line. | [C2] S1, S5, S6 (10 400 bit/s is in S6; also quoted by a forum, S18) |
| Odometer storage | Stored **in the pack** and **mirrored in BCU EEPROM**. They are compared at ignition I→II. The BCU follows the pack upward when they differ by up to 10 km. With the "odometer error warning" option set, the BCU sends an **LCD-flash request** if they disagree by more than ±10 km. TestBook must read the old pack's mileage to program a new one. Odometer reset is allowed once only, before 150 miles. | [C2] S5, S6, S10/S14 (forum: flashing odometer after a swap) |
| Diagnostics / configuration | TestBook configures the pack (auto/manual, diesel/V8, ACE, SLS, Gulf/Japan/ROW, HDC, TC, service reset, odometer reset) and runs a dial/lamp sweep test. The BCU has a matching **"Settings → Instrument pack"** block (Transmission, Engine, ACE, SLS, Gulf, Police, HDC, TRC). | [C2] S5, S17 (bcu-settings.md) |
| Is the pack on the pin-7 K-line? | **No evidence it is.** It is absent from the diagnostic-socket wiring list. The T-26 sweep on this project's car (fast and 5-baud, 0x01–0xEF) found no pack. TestBook reaches it through the bus (BCU/IDM side) [INF]. | [C2] S1 (diag socket section), S17 (test-plan-resolved T-26) |
| Pack status reported to BCU | NanoCom "BCU → Instruments" shows lamp states (DI L/R, trailer DI, seat belt, diff lock, transfer neutral, autobox manual/sport, off-road, ABS, TC, SRS, HDC, glow plug, brake, oil, alternator, check engine, fuel filter, trans temp, check ACE/HDC/SLS), **"Instr. mileage" vs "BCU mileage"**, **"IP trip switch"** and **"IP ignition pos. 2"**. This strongly suggests the pack reports these over the bus. | Menu content: [S1] S17. Bus-reporting interpretation: [INF] |
| Does the BCU rely on the pack? | **Yes, for road speed.** "The BCU receives the road speed signal from the instrument pack via the serial data bus" and uses it for speed-related locking (above 7 km/h / 4 mph). The pack also provides speed to the IDM. | [C2] S1, S6 |
| Trip computer / message centre | **None on D2.** The LCD is odometer/trip/gear only. | [S1] S5 |
| Chime / buzzer | Sounder is inside the pack (see §3) | [C2] S5, S6 |
| Pairing | The BCU-IDM link needs re-learning after replacement (ignition on 5 min or TestBook). No such pairing is stated for the pack. A forum claims the pack is "paired" with the BCU and that nothing works and the odometer blinks without it. | Official: [S1] S6. Forum: [S1] S10 |

### What breaks if the pack is removed (or replaced by a passive display)

| Effect | Why | Conf. |
|---|---|---|
| BCU speed-related locking stops working; any other BCU or IDM speed-dependent logic loses speed | Speed reaches the BCU/IDM only via the pack's bus messages | [C2] S1, S6 |
| All audible warnings are lost: seat belt, key-in, lights-on, transfer neutral, SLABS/HDC chimes | The sounder is in the pack | [C2] S5, S6 |
| The BCU/IDM keep sending, but with no pack the indicator, trailer, seat-belt, PRND and neutral tell-tales disappear | They exist only as bus messages | [C2] S1, S5 |
| Td5 ECM logs **tachometer / temp-gauge / MIL / glow-lamp open-load** faults | The ECM monitors its output drivers | [S1] S17 (proven on car). Mechanism: [INF] |
| SRS DCU logs **lamp-circuit fault** (code 004, "Airbag warning lamp circuit open") and may keep the system in fault | The DCU monitors the YR line for 0 V/open | [C2] S1, S17 (airbag dictionary) |
| EAT may log **Sport/Manual lamp circuit** faults (P1810-12/13) | The EAT monitors its lamp outputs | [S1] S17. Mechanism: [INF] |
| **Alternator may not self-excite / charge** | The charge-lamp line (C0230-11 NY) is the excitation feed through the pack. Without it, the alternator relies on residual magnetism. | Circuit: [C2] S1, S5. Consequence: [INF], high importance. **Keep an excitation path, e.g. a resistor or lamp from ignition to NY.** |
| Fuel sender has no supply or earth, so fuel level is unreadable | The sender is fed and earthed only through the pack | [C2] S1, S3, S5 |
| V8 only: ECM loses the low-fuel signal (misfire plausibility only) | C0233-14 output | [S1] S3, S7 |
| 03MY nav cars: the head unit loses road speed (nav dead-reckoning, auto volume) | C0233-9 output | [S1] S1, S3 |
| BCU odometer copy stops updating, and any new pack shows a mismatch (flashing odo) | Odometer sync logic | [C2] S6, S14 |
| Unaffected: ECM, ACE, cruise, ICE and ATC road speed | They take KG directly from SLABS, not from the pack | [S1] S1 (SLABS road-speed fan-out) |
| Unaffected: SLABS engine speed | SLABS has its own WS tap (C0655-10) | [C2] S1, S3 |

---

## 5. Signal levels and electrical characteristics

| Signal | Pin | Level / characteristic | Conf. |
|---|---|---|---|
| Ignition supply (gauges, most lamps) | C0230-19 LG | +12 V ign (pos. II), F27 10 A | [C2] S1, S3 |
| Ignition supply (SRS lamp) | C0233-6 G | +12 V ign, F35 10 A (shared with SRS DCU) | [C2] S1 |
| Permanent supply | C0230-20 PO | +12 V battery via engine-bay F13 (10 A, "Inertia switch, BCU, IDM, instrument pack") | [C2] S1 text and fuse table |
| Earths | C0230-17, C0230-18 B | Earth header C0551 (footwell, near brake pedal) | [C2] S1, S3, S12 |
| Illumination | C0230-12 RN | Side-lamp + feed (fuse 33 → header C0759). NAS/Japan: rheostat-dimmed RN. S5 says some markets use a **PWM dimmer**. 3 dial bulbs; the pack provides their earth. | [S1] S1 plus S5. Pin mapping: [INF] |
| Lamp sense inputs | see §3 | Pulled up in the pack (about 10 V measured). External sink: < 1.8 V on, > 7.7 V off | [C2] S5, S11 |
| Charge lamp sense | C0230-11 | < 2.0 V on, > 7.6 V off | [S1] S5 |
| Main beam | C0230-3 | +12 V high-side from IDM into a 14 V 2 W bulb (about 0.15 A) | [S1] S5. Current: [INF] |
| Lamp current (LED circuits) | — | Rated 1.2 W per lamp on the diagram (nominal, about 0.1 A). Actual LED current is lower. | [S1] S3. Current: [INF] |
| Road speed | C0230-16 | 0–12 V square wave, 8000 pulses/mile, 2 Hz at standstill | [C2]/[S1] as §2 |
| Engine speed | C0230-15 | 2 ppr; V8 0–5 V (S7), Td5 reportedly 12 V (S15); 228 rpm-equivalent at key-on | as §2 |
| Coolant gauge | C0233-15 | PWM 0–12 V (V8 768–4848 µs pulse width; Td5 unknown) | as §2 |
| Fuel sender | C0230-14 / -13 | 19 Ω full … 245 Ω empty; floating (pack-earthed) | as §2 |
| Serial bus | C0233-16 | Single-wire bidirectional, 10 400 bit/s. Voltage levels are not documented; K-line-like (ISO 9141-style) levels are likely. | Speed: [S1] S6. Levels: [INF] |
| Water-in-fuel sensor | C0233-11 | Sensor output voltage, low (< 1.8 V) = water | [C2] S1, S5 |

---

## 6. Practical notes for a replacement display (all [INF], derived from the above)

1. **Hard-wired inputs** are simple to read with opto or comparator front-ends: speed (C0230-16), rpm (C0230-15),
   coolant PWM (C0233-15), fuel resistance (C0230-14/-13), and about 17 active-low lamp lines. Each lamp line needs a **pull-up to
   about 10–12 V through a resistor**, because the pack normally supplies it and the external ECUs only sink. Without it, ECUs
   that check their outputs (Td5 ECM, SRS DCU, EAT) log open-load faults.
2. Measure the fuel sender **ratiometrically between C0230-14 and C0230-13**. Do not reference it to chassis.
3. **Keep alternator excitation** on C0230-11 NY, using a resistor or bulb from ignition.
4. Indicators, trailer, seat belt, transfer neutral, gear position, chimes and odometer sync exist **only on the SK bus**.
   To keep them you must either keep the original pack in circuit (hidden) or decode the 10 400 bit/s bus.
   Keeping the pack in place but hidden is the low-risk option. It preserves the BCU speed feed, chimes, odometer and the
   SRS/ECM lamp loads, and a custom display can tap the same lines in parallel at high impedance.
5. The Td5 ECM's live data already carries rpm, road speed (`21 0D`), coolant and battery over pin-7 K-line in this repo
   (`references/protocol_state_handoff.md`). A display could use that instead of tapping the gauge wires, but lamp
   states still need the hard-wired lines or the bus.

---

## Confidence / gaps

- **Not found / open:**
  - Td5 coolant-gauge PWM frequency and its duty-to-°C scaling. Only V8 pulse widths were found (S7).
  - The SK serial bus protocol: frame format, addresses, message IDs and voltage levels. Only the 10 400 bit/s rate is documented.
  - Td5 tacho output voltage. 12 V is single-source forum (S15); 0–5 V is the V8 figure (S7).
  - Pack illumination current and bulb wattage. Dimmer type per market (rheostat per S1 vs PWM per S5).
  - Physical and electronic differences of the 2003MY pack beyond C0233-9 and the diff-lock behaviour, for example
    whether 03MY packs need different BCU option coding. The forum swap threads (aulro, landroverforums) returned
    HTTP 403, so only search-result snippets were seen (S14).
  - C0233-9 wire colour conflict: the S1 text says O, while the S3 diagram and S1 table say K.
  - RHD in-line connector numbering. The C0047/C0223 pins quoted are from the LHD 03MY diagram.
- **Conflicts in the sources:**
  - Overspeed lamp: pack logic at 78/74 mph (S1) vs BCU at 77/74 mph (S5). The S5 Hz table implies about 7750 pulses/mile, not 8000.
  - Low-fuel and high-coolant lamps: S5 lamp pages say "ECM illuminates/earths". S1 text, the S3 diagram and S5's own fuel page
    show them as pack-internal. I treat them as pack-internal.
  - Seat-belt lamp supply: "fuse 13" (S5) vs ignition (S1).
  - The 05/99 library text has errata: SRS on "C0233-3", "C0239-19" and "C0235-15" are misprints for C0233-6, C0230-19 and C0230-15.
- **Source independence:** S1, S2 and S3/S4 are all Land Rover publications. They are separate documents (library text,
  connector tables, circuit diagrams, workshop manual) but not fully independent. S3 and S4 are the same drawing
  (VWR102090), so S4 is not counted as a second source. S2 (05/99) and S1 (03MY) agree on every C0230 cavity and on
  C0233 apart from pin 9.
- **Market and variant caveats:** Many lamps are market or option dependent: overspeed (Gulf), transfer neutral
  (NAS/Can/Japan), SLS/ACE/HDC/TC/auto, and the NAS "Service Engine Soon" legend. The workshop manual seen (S5–S7) is the
  **NAS 6th edition**, which has no Td5 content. Td5 behaviour comes from S1, S3, S8, S9 and the repo (S17).

---

## Sources

- **S1**: Land Rover *Discovery Series II Electrical Library*, LRL 0529 (3rd ed., 99MY–03MY), §4 "Instruments" pp. 4.134–4.141,
  plus BCU, SLABS, SRS, charging, lights-on alarm, indicators, interior illumination, ICE/nav sections, fuse table, and
  connector details C0230/C0233. https://discoii.wordpress.com/wp-content/uploads/2012/02/discovery-ii-electrical-library.pdf
  (identical copy: http://www.discovery2.co.uk/pdf_files/D2_electrical_library.pdf)
- **S2**: Land Rover *New Discovery Electrical Library*, VDL 100000 2nd ed. (05/99): instruments text and connector
  details C0230/C0233. https://www.fuddymuckers.co.uk/rtfm/Discovery/D2/Discovery_2_VDL-100000-2-1999-05_Electrical_Library.pdf
- **S3**: Land Rover *Discovery Series II 2003MY Electrical Circuit Diagrams, LHD*, §47 "Instruments" pp. 119–120
  (VWR102090-A-02/-03), §55 Navigation p. 136, §20 EMS V8 p. 59. http://www.discovery2.co.uk/pdf_files/D2_circuit_diagrams_03MY.pdf
- **S4**: Land Rover *Electrical Circuit Diagrams LRL 0553 NAS* (2002), §47 p. 112. This is the same drawing as S3.
  https://gershon.ucoz.com/landrover/TD5/Electrical.pdf
- **S5**: *Discovery Series II Workshop Manual* (RAVE, 1999MY on, 6th ed. NAS), section 88 Instruments, Description
  and Operation, 37 pages, starting at
  https://workshop-manuals.com/landrover/discovery-ii/instruments/description_and_operation/description/
  Key pages:
  - speedo 8000 ppm: …/description/page_1754/
  - odometer: …/page_1755/
  - tacho: …/page_1756/
  - fuel: …/page_1757/ and …/page_1758/
  - coolant: …/page_1759/
  - MIL: …/page_1760/
  - overspeed: …/page_1762/
  - illumination: …/page_1763/
  - high coolant temp: …/page_1767/
  - low fuel: …/page_1768/
  - diff lock: …/page_1771/
  - SRS: …/page_1773/
  - sounder: …/page_1788/
  - diagnostics and config: …/page_1789/
  - block diagram: https://workshop-manuals.com/landrover/discovery-ii/instruments/description_and_operation/instrument_block_diagram/
  - lamp list: https://workshop-manuals.com/landrover/discovery-ii/instruments/description_and_operation/instrument_pack/
  - removal: https://workshop-manuals.com/landrover/discovery-ii/instruments/repairs/instrument_pack/
- **S6**: Same manual, section 86 Body Control Unit:
  - 10 400 bit/s and IDM link: https://workshop-manuals.com/landrover/discovery-ii/body_control_unit/description_and_operation/description/
  - C0661-1 pinout: …/description/page_1527/
  - pack provides speed and odometer stored in BCU: …/description/page_1537/
  - speed locking: https://workshop-manuals.com/landrover/discovery-ii/body_control_unit/description_and_operation/page_1543/
  - odometer update logic: …/description_and_operation/page_1552/
  - audible warnings: …/description_and_operation/page_1555/
- **S7**: Same manual, Engine Management V8:
  - C0637 pin table (8 = low fuel input, 17 = engine speed PWM 0–5 V, 20 = MIL, 22 = VSS PWM 0–12 V):
    https://workshop-manuals.com/landrover/discovery-ii/engine_management_system_v8/description_and_operation/description_engine_management/page_374/
  - VSS 8000 ppm, 0–12 V, 2 Hz: https://workshop-manuals.com/landrover/discovery-ii/engine_management_system_v8/description_and_operation/page_417/
  - low fuel and coolant PWM 768–4848 µs: …/description_and_operation/page_419/
- **S8**: Blackbox Solutions, *Lucas Td5 Diagnostic (SM010)* (C0658 B6 MIL, B7 temp gauge, B13 vehicle speed, B19 tacho,
  B30 glow lamp; tacho open-load note). https://blackbox-solutions.com/help/SM010.html
- **S9**: Rimmer Bros, *Rev Counter Kit Defender Td5 (LR1200)*: "ECU is programmed to output two pulses per engine revolution", pin C0658-19.
  https://rimmerbros.com/ItemFiles/Manuals/LR1200.pdf
- **S10**: LandyZone, "Td5 instrument cluster": D2 pack in a Defender. Indicator and trailer LEDs need modification; no fog
  tell-tale; pack "paired" with the BCU over the serial link. https://www.landyzone.co.uk/land-rover/td5-instrument-cluster.386820/
  The LandyZone "Tachometer" thread (https://www.landyzone.co.uk/land-rover/tachometer.345338/) also gives 2 ppr on pin 19 (search summary).
- **S11**: LandyZone, "instrument lights": C0230-4/-5 measured about 10 V idle, 0 V when active; in-line connector layout differs.
  https://www.landyzone.co.uk/land-rover/instrument-lights.197104/
- **S12**: LandyZone, "Instrument panel not working on D2": earths C0230-17/-18 to header C0551 via C0047/C0223 pins 11/13; feed on C0230-19 (F27).
  https://www.landyzone.co.uk/land-rover/instrument-panel-not-working-on-d2.323991/
- **S13**: AULRO, "fuel gauge D2" (about 15 Ω full; search snippet only, page 403):
  https://www.aulro.com/afvb/discovery-2-a/213251-fuel-gauge-d2-2.html
- **S14**: Cluster swap and odometer threads (search snippets only; pages 403):
  - https://www.aulro.com/afvb/discovery-2-a/184388-instrument-cluster-d2-d2a.html
  - https://landroverforums.com/forum/discovery-ii-18/99-02-03-04-gauge-cluster-swap-75994/
  - https://www.aulro.com/afvb/discovery-2-a/253505-flashing-odometer.html
- **S15**: LR4x4, "Wiring the tach to a Compushift on a Defender Td5" (12 V pulse; cannot drive two loads; search snippet).
  https://lr4x4.com/topic/36510-wiring-the-tach-to-a-compushift-on-a-defender-td5/
- **S16**: 4x4Community, "TD5 Rev counter issue" (claims 1 ppr on an early ECU; search snippet).
  https://www.4x4community.co.za/forum/showthread.php/154730-TD5-Rev-counter-issue
- **S17**: Local repo this repo (`JamesWrightDavid/discovery2-diag`):
  - `references/menus/bcu-inputs.md` (BCU → Instruments / Power distribution)
  - `references/menus/bcu-settings.md` (Instrument Pack config)
  - `references/test-plan-resolved.md` (T-26 address sweep, 2026-10-04)
  - `docs/discovery-2-td5/fault-dictionary-td5.md` (gauge and lamp open-load faults)
  - `docs/discovery-2-td5/fault-dictionary-airbag.md` (004 lamp circuit)
  - `docs/discovery-2-td5/fault-dictionary-autobox.md` (P1810)
- **S18**: Land Rover Forums thread quoting the RAVE 10 400 bit/s datalink text (search snippet only):
  https://landroverforums.com/forum/discovery-ii-18/
