---
title: "D2 instrument pack replacement: how it fits Ostler"
area: references
status: stable
version: 1.0
updated: 2026-10-14
summary: >
  Survey of the Ostler platform, firmware, shell and this pack for a cluster replacement: the data path (ESP32-S3 node, gate, MQTT/VISS, VSS overlay), the existing warning and instrument rules (shell spec §21, ADR-0052), the planned read-only isolated tap nodes (ADR-0032 §14), the safety rules a replacement must keep, where each piece belongs, and 16 platform gaps.
---

# How a D2 Td5 instrument-cluster replacement fits Ostler

Research date: 2026-10-14. Read-only survey of four checkouts:

- **Platform docs and code** (`openostler/ostler`): `openostler/ostler` (cited below as `P/`).
- **Node firmware:** `openostler/ostler-firmware` (`F/`).
- **D2 pack** (`d2diag`): this repo (`JamesWrightDavid/discovery2-diag`) (`D/`).
- **Shell UI library** (`ostler-shell`): `openostler/ostler-shell` (`S/`).

Every claim cites a file. Where the docs say nothing, this file says so; it does not guess.

> **Headline.** Ostler has no plan for replacing the instrument cluster. It does have most of
> the parts for *augmenting* it:
>
> - instrument pages and warning rules in the shell;
> - read-only, isolated pulse, analogue and lamp taps on a sensor node;
> - VSS names for most cluster values.
>
> The plan's word "cluster view" is a false friend. It means the view of a *compute cluster*
> of Ostler devices, not the instrument cluster (§2.1).
>
> On the D2 the K-line can't feed a cluster today, for three reasons:
>
> - the node's gate refuses the Td5 security unlock (Tier 0 only);
> - SLABS goes silent once the car moves;
> - nothing on the bus reports the cluster's own warning lamps or fuel level.
>
> So a D2 cluster needs **wire taps** first.

---

## 1. Data path: car → node → gateway/Brain → VSS/VISS → shell

### 1.1 Components, hop by hop

1. **The car's buses.** On the D2 this is K-line, KWP2000 on OBD pin 7
   (`D/hardware/README.md`, "OBD-II pins used").
   - The car's buses stay at the edge and are never bridged onto Ostler's network
     (`P/decisions/adr-0027-ip-everywhere-ecosystem-architecture.md` §10).
2. **The node** (Ostler Diagnostics, ESP32-S3).
   - It owns K-line/CAN I/O, decodes to VSS, holds the transmit gate, and carries GPS
     and the parked broker (`P/decisions/adr-0032-one-node-optional-brain.md` §2;
     `P/decisions/adr-0039-product-family-diagnostics-guardian-hub.md` §1).
   - The firmware app is in `F/firmware/node/`. On core 1 it runs init → `21 <lid>`
     reads → decode with the portable C decoder → publish → release `82`
     (`F/firmware/README.md`, "What it does").
   - It reads the pack's `signals/*.json` as JSON at run time, with no generated header
     (`F/CLAUDE.md`, "Hard rules"; `P/CONSTITUTION.md`, "Layering and data").
3. **Node → Brain/gateway link.**
   - Product links: USB-NCM, or 10BASE-T1S. Ethernet is for prototypes; UART is
     dev-only (`P/decisions/adr-0032...md` amendment C2; `adr-0039...` §4).
   - Today the node publishes MQTT 5 on `ostler/v1/<vid>/<device>/vss/<VSS path>`.
     Each payload is `{value, unit, ts, t_us, source, name, c, raw}`
     (`F/firmware/README.md`, "What it publishes").
   - The Brain reads it through `NodeFeed`/`NodeSource`. Staleness is tracked per
     signal on the node's clock, and confidence is never raised
     (`P/docs/architecture.md`, NodeSource section).
4. **The target protocol is VISS v3 on every hop.**
   - Node→gateway, node→Brain and Brain/gateway→apps all speak VISS v3 over WebSocket,
     with gRPC/HTTP where they fit (`P/decisions/adr-0051-viss-v3-is-the-core-vehicle-data-protocol.md` §1–2).
   - MQTT becomes external only. Migration steps V0–V4 retire the internal broker and
     SSE `/events` (§3, §8).
   - The **gateway** (the Brain's Python service, or the Android Gateway app) owns the
     node link, the VISS server and the permissions controller (§5, §9).
5. **Apps and the shell.**
   - Every app bundles the `ostler-shell` library
     (`P/decisions/adr-0052-mfd-pages-sizes-and-the-shell-library.md` §2).
   - Pages read the gateway over VISS. The shell itself "never talks to hardware or holds
     car data" (`S/CLAUDE.md`, "Working rules").
   - The shell gets motion state, alerts, module freshness and layouts from the gateway
     through `ShellHost` (`S/src/contract/host.ts`).

### 1.2 Where signals get their VSS names

- **The `metric` field.** A pack signal record may carry `metric`, a COVESA VSS 6.1 path
  (`P/decisions/adr-0016-covesa-vss-canonical-signal-namespace.md`, "Decision";
  `D/docs/architecture.md`, "Signal store"). It is written through `upsert_field`, which
  refuses an unknown path (`P/docs/architecture.md`, "Metrics").
- **The one overlay.** `P/vss/ostler.vspec` is the *single* overlay. It adds
  `Vehicle.Ostler.*` for meanings VSS lacks.
  - **Packs do not ship overlays.** A new meaning gets a reviewed `Vehicle.Ostler.<Domain>.*`
    entry in that platform file (`adr-0016` "Decision").
  - `tools/build_metrics.py` generates `src/openostler/metrics.json` from it (`P/vss/README.md`).
- **Fields without a VSS path.** These publish as `<pack>.<module>.<field>`, for example
  `d2.td5.air_temp` (`F/firmware/README.md`).
- **D2 metrics today.** All are from `D/src/d2diag/signals/*.json`:

  | Field | VSS path | Confidence |
  |---|---|---|
  | `td5.rpm` | `Vehicle.Powertrain.CombustionEngine.Speed` | proven |
  | `td5.speed` | `Vehicle.Speed` | proven |
  | `coolant_temp` | `…EngineCoolant.Temperature` | proven |
  | `battery` | `Vehicle.LowVoltageBattery.CurrentVoltage` | proven |
  | `manifold_press` | MAP | proven |
  | `maf_sensor` | MAF | proven |
  | `accel_pedal_pct` | `…Accelerator.PedalPosition` | proven |
  | `ambient_press_1` | `Vehicle.Exterior.AirPressure` | proven |
  | `transfer_low` | `…Transmission.IsLowRangeEngaged` | proven |
  | ride heights | `Vehicle.Ostler.Chassis.RideHeight.*Raw` | proven |

  - **Gaps:**
    - there is no fuel level ("There is no fuel-level source",
      `P/specs/2026-10-06-u0-seams-design.md` §D);
    - there is no oil pressure value;
    - no warning lamp has a metric.
- **Upstream VSS 6.1 already has most cluster leaves** (`P/vss/upstream/vss.json`):
  - `Vehicle.Speed`, `CombustionEngine.Speed`, `EngineCoolant.Temperature`, `EOP`;
  - `FuelSystem.RelativeLevel`, `IsFuelLevelLow`;
  - `Vehicle.TraveledDistance`;
  - `Chassis.ParkingBrake.IsEngaged`;
  - `Body.Lights.DirectionIndicator.{Left,Right}.IsSignaling`, `Beam.High.IsOn`,
    `Fog.Rear.IsOn`;
  - `ADAS.ABS.IsError`;
  - `Cabin.Seat.Row1.DriverSide.IsBelted`;
  - `Exterior.AirTemperature`.
- **Not in VSS** (they would need overlay entries):
  - D2-style telltales: glow plug, charge/alternator, oil pressure *switch*, water in fuel
    filter, transmission temperature, SLS, HDC, ACE, traction control, diff lock;
  - MIL, beyond `Vehicle.Ostler.Diagnostics.MilOn`.

---

## 2. What the docs already say about the cluster

### 2.1 "Cluster view" does not mean the instrument cluster

- `P/references/research/cluster_view.md` is titled "Cluster view and role holders —
  Proxmox, Home Assistant, election patterns for a small in-car cluster".
- It feeds ADR-0037 (role holders) and the Network page. It is about showing the
  *devices* in the car, not instruments.
- `P/specs/2026-10-06-ui-architecture-design.md` §3.7 ("Network page, peer view and
  cluster view") and `GET /cluster` (`P/docs/architecture.md`) are the same concept.
- **No document proposes replacing the D2 instrument cluster.**

### 2.2 Instrument-cluster facts that are in the docs

- **The fast-signals spec.** `P/specs/2026-10-02-canbus-and-fast-signals-design.md`
  (draft, 2026-10-02) proposes "tapping the instrument-cluster tacho/speed pulse signals
  for fast RPM/road-speed without polling the K-line".
  - Road speed is "measured by the instrument cluster via the BCU, not the ECM".
  - Measurement uses "a hardware pulse counter — RP2040 PIO or ESP32 PCNT — not in Linux".
  - "Input protection (mandatory) … fast optocoupler (6N137)".
  - Plan B1: verify the taps "against the K-line `21 09` and the speedo".
  - Open question: "Exact cluster-connector pins for the tacho drive and the road-speed
    feed, and their voltage swing."
- **No instrument pack on the bus.** The pack's address sweep T-26 found that "no
  cruise, HEVAC, **instrument pack** or IDM answers a standard fast or 5-baud init on
  pin 7 on this car" (`D/references/test-plan-resolved.md`, 2026-10-04;
  `D/docs/discovery-2-td5/system-map.md` row "HEVAC, instrument pack, IDM: 🔴
  unconfirmed … else read via BCU inputs").
- **The BCU reports the cluster's state.** NanoCom's BCU → Instruments page shows 24
  discrete warning states: ABS, SRS, Brake, Oil pressure, Alternator, Check engine,
  Glow plug, Fuel filter, Transmission temp., Check ACE/HDC/SLS, Diff lock and more.
  - It also shows "Instr. milage" and a separate "BCU milage", plus "IP ignition pos. 2"
    (`D/references/menus/bcu-inputs.md`).
  - These are **not decoded** (`D/references/nanocom/feature_map.md` row "Read inputs —
    instrument … OPEN").
- **Td5 cluster outputs.** The Td5 ECM drives cluster outputs through service `30`:
  MIL `30 A2`, rev counter `30 B7`, temperature gauge `30 BA`, glow plugs `30 B3`
  (`D/references/input_mapper_design.md`; `D/docs/capability-inventory/td5.md`).
  - Its feature config lists "Temperature Gauge; Tachometer; … Road Speed … MIL Lamp"
    (`D/references/menus/td5.md`).
- **Ambient temperature** is "measured by the instrument cluster via the BCU, not the
  engine ECU" (`D/docs/discovery-2-td5/engine-td5.md`, ext_temp section).
- **The ECUs watch their warning-lamp circuits.** Faults already seen on the car:
  - airbag 004 "warning lamp open circuit" (`D/references/airbag_fault_codes.md`);
  - Td5 "GLOWPLUG LAMP DRIVE OPEN LOAD" (`D/references/fault_capture_runsheet.md`).

  So a replacement that removes the lamp loads would set fault codes. See §4.

### 2.3 Instrument pages and the gauge display

- **The gateway app ships the instrument pages.** It "ships the **instrument pages**
  (attitude, heading, transfer, speed, engine)". ADR-0052 is **proposed**, not yet
  accepted (`P/decisions/adr-0052...md` §1).
- **Instruments are ordinary pages.** There are no drive modes; "Off-road" is just a
  saved layout (§5).
- **The default instruments** are Speed, Attitude, Heading, Transfer and Engine, plus
  Route during guidance. "An app sets this through `instrument: true`"
  (`P/references/design/2026-10/shell-session/SPEC-motion-sound-text.md` §7.1).
- **The page contract has an instrument flag:** `instrument?: boolean | (() => boolean)`,
  "A warning never covers an instrument's values" (`S/src/contract/page.ts`, `PageDef`).
- **The mock instrument pages** (`S/src/mock/pages/instruments.tsx`, placeholder data):
  - `ENGINE_GAUGES` lists coolant, battery, boost, oil, fuel and intake;
  - fuel's module is `"instruments"`, a module that doesn't exist on the car;
  - coolant tone: caution from 105 °C, warning from 115 °C.
- **No locked primary region.** "Cockpits always keep the primary flight display.
  **Recommend no fixed tile,** but warnings always have a place … **Decided 2026-10-12
  (owner): no fixed tile.**" (ADR-0052 §9 item 10).
- **Add-on display ideas:**
  - "Gauge display: A small round or bar display for chosen signals | T1S | VSS paths
    via MQTT | next" (`P/references/research/addons_catalogue.md` §1);
  - backlog #29 "Gauge display module: an ESP32-S3 round/bar screen for always-visible
    coolant, boost, EAT temperature, flags and alarm state … Optional, alongside the
    main screen" (`P/references/research/features_backlog.md`).
- **Telltales while moving.** The UI spec allows a "telltale sheet (≤ 30 characters a
  line)" while Moving (`P/specs/2026-10-06-ui-architecture-design.md` §3.5).
  - The app model has a `telltale_list` template, "≤ 6 rows" (`P/specs/2026-10-06-app-model-design.md`).
  - "Safety surfaces are not removable. The fault telltale …" (same spec).
  - UK regulation 109 allows "Diagnose live data and telltales" on a driver-visible
    screen (`P/references/research/driver_distraction_rules.md` §4.2).

### 2.4 Warning rules (quoted)

- **ADR-0052 §8:**
  - "**Caution (amber):** rings the tile the problem is about …, lights the strip and
    chimes; it never takes over a tile."
  - "**Warning (red):** takes over one tile until ACK … never covers an instrument page."
- **Shell spec §21 ruling 1** (`P/specs/2026-10-11-ostler-shell-design.md`, owner
  2026-10-13):
  - "**A warning on its own instrument tile** … its main value always stays visible: a
    red warning ring …, a red ACK bar inside the bottom of the tile …, and the app
    turns its own value red."
  - Small tiles (under 200 dp) get ring and tag only. "An instrument's values are never
    covered."
  - Implemented in `S/src/alerts/metrics.ts` (`holdsBar`, `barHeight` 68/78 dp).
- **Ruling 2:** "The alert tab is 60 dp tall while moving … every touch target … at
  least 60 dp."
- **Ruling 3 (locked colours):**
  - "A contrast check for the locked warning and caution colours … runs in CI … Themes
    that fail are tuned … The locked alert colours themselves don't change."
  - ADR-0052 §8: themes never change "warning and caution colours or safety icons";
    imported CSS keeps them "locked".
- **Fail-safe defaults** (shell spec §21):
  - "unknown motion (missing, stale, an unreachable Gateway, a new host …) counts as
    moving";
  - "every warning shows; a stale value never looks fresh; and alerts stay above edit
    mode".
- **Alert routing.** Alerts are app → Gateway → shell; "the shell only routes the alert
  … and never recolours anything inside a page. There is no master caution"
  (`S/src/alerts/types.ts`).
- **Sound:**
  - the warning tone repeats "every 10 s, 6 dB quieter, until ACK";
  - "Warnings can't be muted below 50% of the alert volume"
    (`SPEC-motion-sound-text.md` §3).
- **Steering wheel.** "Select acknowledges a warning only while one is showing" (ADR-0052 §4a).

### 2.5 The moving rules and the 24 dp text floor

- **ADR-0052 §5:**
  - "While moving, the floor is about actions, not pages … **text minimum 24 px**";
  - §9 decision 1 recommends 24 px over the brief's 22 px.
- **Shell spec §21:** "All text is at least 24 dp while moving, labels included."
- **Code** (`S/src/motion/rules.ts`):
  - `MOVING_TEXT_MIN_DP` and `MOVING_TARGET_MIN_DP` come from tokens;
  - `toMotionState()` maps anything unknown (including `idling`) to `unknown`, which
    `isMoving()` treats as moving.
- **Motion reports** (`S/src/contract/host.ts`): the gateway sends motion at least every
  1 s, and a report older than 3 s counts as unknown.
- **Instrument tiles turn solid while moving**: "transparent instrument tiles turn solid"
  (`SPEC-motion-sound-text.md` §7.2).
- **Driving state** comes "from vehicle speed (any system in session), then GPS speed …,
  then proven gear/handbrake signals" (`P/specs/2026-10-06-ui-architecture-design.md` §3.5).
  - Moving means "> 5 km/h for > 2 s".
  - "in a SLABS session the Td5 speed is absent and GPS covers it."

---

## 3. Hardware: input nodes, MCUs and input boards

### 3.1 The MCU choice: ESP32-S3

- **Current decision.** The node and every variant run on **ESP32-S3 with PSRAM**,
  "one firmware, all variants" (`P/decisions/adr-0039...md` §2;
  `P/decisions/adr-0032...md` §6, Risks "PSRAM is required").
  - The K-line reference is an ESP32-S3 with a discrete transceiver (MC33290 or Si9241;
    the L9637D is obsolete) (`F/firmware/README.md`, "Bench wiring").
- **Earlier draft: Pi + RP2040.** `P/specs/2026-10-02-hardware-platform-design.md` (draft)
  chose "a Pi brain + RP2040 real-time co-processor … plus an always-on low-power tracker
  ESP32". It used RP2040 PIO for "K-line timing, CAN, I2C sniff, pulse count".
  - ADR-0032 (one node, no separate buddy; firmware in portable C on the ESP32)
    supersedes this in practice.
  - RP2040/can2040, Teensy and STM32 survive only as options in the CAN fast-signals spec
    table ("Step to Teensy/STM32 only if …") and in hardware research for Pi power
    boards (`P/references/research/hardware.md`).
- **Bus timing.** ESP32-S3 + LAN8651 is the T1S reference
  (`P/decisions/adr-0026-module-bus-10base-t1s.md`; `P/references/t1s_bench_plan.md`).

### 3.2 Sensor and input nodes

ADR-0032 §14 describes them:

- **What they read:** "a fast tacho (crank, coil or injector pulses), an EGT thermocouple,
  boost, oil pressure and oil temperature, wideband AFR …".
- **How they tap:** "Taps on car wiring are read-only and isolated (optocouplers or high
  impedance). They never load or alter an ECU signal."
- **Detected or declared** (ADR-0032 Amendments B1): what can't be detected is
  declared, including "tacho and speed pulses with their wheel pattern, analog senders,
  read-only taps".
- **Budgets checked at boot** (B2): ADC2 with Wi-Fi on, reserved pins, and "more pulse
  inputs than PCNT and MCPWM capture provide".

The add-ons catalogue lists the products (`P/references/research/addons_catalogue.md`;
`P/GOALS.md` §4, "Sensor nodes … read-only, isolated taps; readings are source-tagged VSS
signals"):

- Fast tacho sensor node;
- Boost and oil sensor node;
- **Read-only isolated taps:** "Reads car wiring (lamp, switch, sender lines) through
  optocouplers or high-impedance inputs; never loads or alters an ECU signal | T1S | VSS
  with source tags | next".

### 3.3 ESP32-S3 input budget

From `P/references/research/node_sensors.md` §4–§5.

| Resource | What is available |
|---|---|
| PCNT | **4 pulse inputs** (tacho, road speed, flow) |
| MCPWM capture | Up to 6 per-tooth inputs |
| RMT RX | Lab snapshots only |
| ADC | **ADC1 only** for senders, because ADC2 is unusable with Wi-Fi on; an external ADS1115 when more channels are needed |
| LEDC | 8 PWM channels: "Gauge outputs on add-ons only" |
| GPIOs | About 8–12 free on a custom node board |

- **K-line and pulse counting together.** "PCNT tacho alongside K-line safely";
  per-tooth crank timing belongs on an engine-bay node (§5 verdict).
- **Placement** (§7): the engine-bay node goes on the cabin side of the bulkhead
  (ESP32-S3 parts are rated to 65–105 °C ambient).

### 3.4 The firmware's sensor-detection spec (accepted, not implemented)

`F/docs/specs/sensor-detection.md` is the input-board design on paper.

- **Item kinds:**
  - `pulse.crank` (PCNT `count` or MCPWM `per_tooth`; `conditioner: vr|opto|hall|logic`;
    wheel pattern);
  - `analog.sender` (ADC1 pin, divider, named or inline curve, `open_v`/`short_v`);
  - `tap.digital` (pin, `input: opto`, active level);
  - `spi.max31855`;
  - `output.relay`, which needs category and tier and is refused on boards with
    `outputs_allowed: false`, including every diagnostic node.
- **Each item maps to a VSS `metric`** and a rate (§3.2).
- **Harness ID** (§4): a resistor on an ADC1 pin selects one of 15 × 15 harness
  profiles from a `harness-ids.json` registry *in the firmware repo*. "Adding a harness
  is a PR with its wiring drawing."
- **Manifest** (§7): items publish into the capability manifest with `origin` and
  `status` (`ok`/`no_signal`/`fault`/`refused`).
- **Config changes** need a local link, the Owner role and Parked (§3.3).
- **Not built yet.** `F/firmware/node/components` has `node_hal`, `kline`, `gate`,
  `poll`, `nodecfg` and `ojson`. There is no PCNT, ADC, harness or manifest code (grep
  finds none).

### 3.5 Display hardware

- **Displays are Android head units or phones** on Wi-Fi or USB
  (`P/decisions/adr-0027...md` §2, "Displays and phones"). The Dudu7 class is the bench
  target (`P/decisions/adr-0052...md` Confirmation).
- **Head units cold-start.** They "force-stop apps 20–30 s after ignition off …
  Every wake is a cold start" (`P/references/research/direction_standard_not_os.md`).
- **The Brain boots slowly.** It is off while parked and is woken by the node; the boot
  timeout is 60 s (`P/decisions/adr-0040-power-states-and-wake.md` §2, §4.4).
- **No dedicated cluster display exists.** The only small-display idea is the gauge
  display module (§2.3).

---

## 4. Safety constraints that bear on a cluster replacement

### 4.1 Read-only from the car and the transmit gate

- **The gate is the only path to the car.** "The node transmit gate is the only path to
  the car … No other device, link or service transmits to the car"
  (`P/CONSTITUTION.md`, "Safety").
- **The firmware gate allows Tier 0 only.** That means `81`, `82`, `3E`, `21 <lid>` and
  scrubbed `1A`. "Everything else is refused", including SecurityAccess `27` and session
  changes `10` (`F/firmware/README.md`, "The transmit gate").
- **So the Td5 can't be read from the node today.** "The Td5 engine ECU answers `21 xx`
  only after StartDiagnosticSession `10 A0` and SecurityAccess `27 01`/`27 02` … The
  node does not send them, so on the car the Td5 is expected to answer `7F 21 33`"
  (same file). The keygen plugin is a placeholder (`F/keygen/`).
- **The hard lines permit the fix.** They allow the SecurityAccess "a module needs for
  diagnostic reads (e.g. the Td5 seed-key unlock)" (`P/GOALS.md` §3). So the fix is a
  firmware keygen plugin plus a gate rule, not a policy change.
- **Displays must not transmit.** The cluster display and any emulated output must
  never transmit to a car bus.
  - "Never transmit with the source address of a module present in the car"
    (`P/decisions/adr-0024-body-bus-links-passive-by-default.md`, the I/K-Bus case,
    stated generally in "Decision drivers": "Never impersonate a module the car relies on").
  - Head-unit CAN emulation runs on "a dedicated bus to the head unit, not the car's"
    (`P/specs/2026-10-02-canbus-and-fast-signals-design.md`).

### 4.2 "The brain never touches the car"

- **The rule.** "Only the node talks to the car's buses; the brain (and the cloud,
  phone and Home Assistant) consume the node's VSS messages over IP"
  (`P/CONSTITUTION.md`, "Layering and data"; `P/decisions/adr-0032...md` §3).
- **One exception.** ADR-0044 allows adapters on a Brain *with no node*.
- **What this means for a cluster:**
  - taps and pulse counting live on a node, not the Pi ("not in Linux (Pi scheduler
    jitter …)", fast-signals spec);
  - cluster screens are consumers only.

### 4.3 Taps must be read-only and isolated

- **The rule.** "Taps … never load or alter an ECU signal" (`P/decisions/adr-0032...md`
  §14, B2).
- **Enforced in the schema.** "the config schema cannot turn a tap into an output"
  (`P/references/research/node_sensors.md` §8).
- **Bench check.** A crank tap needs "a conditioner with high input impedance; bench
  check on the scope that the ECU's waveform is unchanged" (same file).
- **Removing the cluster changes the loads.** This is the key point for a *replacement*.
  The ECUs monitor their lamp circuits: airbag fault 004 "warning lamp open circuit";
  Td5 "GLOWPLUG LAMP DRIVE OPEN LOAD" (§2.2).
  - A replacement that removes the cluster would need dummy loads or driven lamp
    circuits.
  - That is *altering* the car's wiring, which ADR-0032 §14 and §15 ("Anything that
    switches a car function needs its own ADR") don't allow without a new ADR.
  - **Augment, don't remove, is the only path the rules already cover.**
- **Outputs need their own ADR.** "Every output declares its category and tier … a
  board profile with outputs disallowed (… every diagnostic node) refuses a declared
  output" (ADR-0032 B2). Driving original gauges through LEDC PWM would be an output.

### 4.4 Fail-safe display of warnings

- **Stale data.** "a stale value never looks fresh"; a gateway loss shows a banner
  (shell spec §21; `S/src/contract/freshness.ts`).
- **Module freshness is coarse.**
  - The gateway re-sends module status every **10 s**; a module goes stale after
    **30 s** (`MODULE_HEARTBEAT_MS`, `MODULE_FRESH_MS` in `S/src/contract/freshness.ts`).
  - That is far too slow for a speedometer or a red telltale. Per-signal staleness exists
    at the Brain from the node's `t_us` (`P/docs/architecture.md`, NodeSource), but the
    shell contract has no per-signal age budget.
- **Data honesty.** "A missing value is never zero, an unscanned system is never OK,
  and stale data looks stale" (`P/GOALS.md` §2.4). The shell's "Unavailable … show '—'"
  rule matches (`SPEC-motion-sound-text.md` §5).
- **Wi-Fi isn't trusted for critical links.** "Never rely on Wi-Fi for alarm-critical
  links … Wi-Fi may carry convenience and telemetry only" (`P/decisions/adr-0026...md`).
  - Displays are on Wi-Fi or USB (ADR-0027 §2).
  - Nothing classifies *driver warnings* as alarm-critical, so whether a cluster's
    telltale link may be Wi-Fi is undecided.
- **Alarm paths are independent.** "Alarm paths never depend on the brain or the
  internet" (`P/CONSTITUTION.md`). There is no matching rule for warning lamps.

### 4.5 Motion rules

- **Unknown counts as moving** (§2.5). Safety follows actions, not pages (ADR-0052 §5).
- **Steering-wheel controls** "never change the car beyond acknowledging a warning"
  (ADR-0052 §4a).
- **Moving rules are enforced on the server.** "The server re-checks the state from the
  node on every action" (UI spec §3.5).
  - A cluster's own speed source (a tap) would also feed the driving state.
  - ADR-0032 A5: "A gate decision never trusts a speed that arrived from another
    device's role; it uses its own bus or its own receiver."

### 4.6 Data honesty for new signals

- **Every field is proven or candidate.** "Nothing is promoted to `proven` without a car
  result recorded in the pack's car-test plan" (`P/CONSTITUTION.md`;
  `D/CONSTITUTION.md`).
- Tap-derived speed, RPM and fuel start as `candidate`. The fast-signals spec's B1
  (verify against K-line `21 09` and the speedo) is the natural test-plan item.

### 4.7 D2 protocol facts that rule out K-line for driving data

- **SLABS is standstill-only.** "Diagnostics are STANDSTILL-ONLY … went silent the
  moment the car moved … stays dead until the next ignition cycle … live ABS-sensor data
  while driving is unreachable over K-line — only an analog tap on the sensor wires can
  get that" (`D/references/slabs/overview.md`).
- **One module at a time.** "K-line is a shared bus: one module at a time"
  (`D/CONSTITUTION.md`), with request spacing P3 from 55 ms
  (`P/references/research/node_sensors.md` §5).
- **The consequence.** The Td5 is the only K-line source of speed, RPM and coolant
  while driving, at a few Hz shared across all its fields.
- **Nothing on the bus covers the cluster's own lamps or the fuel gauge** (§1.2, §2.2).

---

## 5. Where a "D2 instrument pack replacement" naturally sits

The repo rule: a part gets its own repo only when its "toolchain, licence, release
cadence or contributors differ" (`P/decisions/adr-0034-repo-boundaries.md`). Packs are
"JSON data for the C decoder … optional lab-only Python … optional C keygen plugin
source".

### 5.1 The D2 pack (`discovery2-diag`): vehicle facts as data

- **Signal store additions** (`D/src/d2diag/signals/`):
  - BCU "Instruments" decoding of the 24 warning states, instrument and BCU mileage, and
    IP ignition position, as candidate fields with `metric`s;
  - the remaining Td5 fields;
  - the NanoCom capture path is already planned (`D/references/bcu_capture_runsheet.md`,
    "INSTRUMENT states (×24)").
- **A D2 cluster-harness description as data.** For each line:
  - the cluster connector pin;
  - the signal: tacho drive from the ECM, road-speed feed, fuel sender, temperature
    gauge drive, lamp lines;
  - voltage swing, conditioner, pulses per km or rev, and the fuel sender curve.
  - All `candidate` until measured. This answers the fast-signals spec's open question
    ("Exact cluster-connector pins …").
  - **Tension:** the firmware spec puts harness profiles in `F/harness-ids.json`, but
    ADR-0034 says vehicle data lives in packs. The pack should own the D2 profile, with
    the firmware registry pointing at it (gap G7).
- **Warning thresholds and plain-words alert text** (coolant caution and warning, low
  fuel, oil pressure and so on).
  - The store already has `limits` and `normal` per field (for example coolant).
  - The mock's 105/115 °C thresholds live in shell mock code today
    (`S/src/mock/pages/instruments.tsx`) and belong in pack data.
- **Test-plan items** (`D/references/test_plan.md`):
  - tap vs `21 09` RPM;
  - tap speed vs `21 0D` and GPS (ADR-0032 A5's speed-ratio check);
  - lamp taps vs the BCU instrument states;
  - each with a procedure and a decision rule written first.
- **Overlay requests to the platform** for D2 telltales with no VSS node (§1.2). The
  pack proposes them; it can't ship them (ADR-0016).
- **`layout.json` / a saved "Cluster" layout** for the D2 (right-hand drive is already
  declared, `D/docs/architecture.md`).

### 5.2 The platform (`openostler/ostler` and its app and shell repos)

- **VSS overlay.** `Vehicle.Ostler.<Domain>.*` entries for telltales VSS lacks, with
  reuse of upstream leaves where they fit: `ParkingBrake.IsEngaged`, `ABS.IsError`,
  `IsBelted`, `DirectionIndicator.*`, `FuelSystem.RelativeLevel`, `TraveledDistance`.
- **A generic analogue/pulse/lamp input node** as a *product and contract*:
  - the module manifest entries for `pulse.*`, `analog.*` and `tap.*` signals;
  - source tags (ADR-0032 §13);
  - the "Read-only isolated taps" and "Fast tacho" add-ons (`addons_catalogue.md`).
  - The firmware implements it (§5.3). The platform owns the manifest schema and the
    module-contract conformance kit (ADR-0034).
- **Cluster screen layouts and instrument pages.** These ship in the gateway app
  (ADR-0052 §1):
  - Speed;
  - Engine (tacho, coolant, fuel, oil, battery);
  - a **telltale strip or page** with `instrument: true`.
  - A saved "Cluster" layout, possibly auto-selected when the car starts (ADR-0052 §9
    item 5).
- **A telltale-to-alert service in the gateway.** Today alerts are raised by apps
  (`S/src/alerts/types.ts`). A cluster needs a gateway-side rule set (from pack data) that
  raises caution or warning from signals even when no app page is open.
- **Driving state.** Accept a tap-derived speed as "vehicle speed" in the UI spec §3.5
  order, so the D2 has a moving signal without GPS while SLABS or the Td5 are offline.

### 5.3 The firmware (`ostler-firmware`)

- **Implement `docs/specs/sensor-detection.md`:** board profiles, PCNT pulse counting,
  ADC1 senders with curves, opto digital taps, harness ID, the capability manifest, and
  VSS publishing with source tags.
- **The Td5 keygen C plugin and a gate rule** for the diagnostic-read unlock (`10 A0`,
  `27 01/02`), with shared vectors against the pack's Python keygen (ADR-0032 §8,
  ADR-0034). Without this the node can't read speed, RPM or coolant.
- **A board profile for a D2 "cluster tap" node** (`outputs_allowed: false`). This is
  either the Diagnostics node behind the dash (PCNT tacho and speed are allowed beside
  K-line) or a separate sensor node.
- **VISS v3 serving (migration step V3)**, so a display can subscribe straight to the
  node with no Brain in the path (ADR-0051 §8).
- **Optional later:** gauge-display firmware (ESP32-S3 round or bar screen, backlog #29)
  that subscribes to the node directly. This is the "always-visible", Brain-independent
  telltale path.

---

## 6. Gaps: platform pieces that don't exist yet

| # | Gap | Evidence |
|---|---|---|
| G1 | **No ADR or spec for cluster replacement or augmentation**, and no regulatory research. Nothing covers UN/ECE telltale or speedometer requirements, MOT, or odometer continuity. The BCU and the instrument pack each hold a mileage (`D/references/menus/bcu-inputs.md`) | grep finds no such doc; the distraction research only covers regulation 109/110 screen use (`P/references/research/driver_distraction_rules.md`) |
| G2 | **Removing the cluster's lamp loads** would set ECU faults and needs dummy loads or outputs, which the rules forbid without an ADR | §4.3 |
| G3 | **The node can't read the Td5:** no keygen plugin, and the gate is Tier 0 only | `F/firmware/README.md`; `F/keygen/` placeholder |
| G4 | **Sensor detection and input items aren't implemented** (no PCNT, ADC, taps, harness ID or manifest in the firmware) | `F/firmware/node/components/` |
| G5 | **No fuel-level, oil-pressure or telltale signals** in the D2 store, and no overlay paths for D2 telltales | §1.2; `P/specs/2026-10-06-u0-seams-design.md` §D |
| G6 | **BCU "Instruments" page not decoded** (24 states, two mileages) | `D/references/nanocom/feature_map.md` "OPEN" |
| G7 | **Harness profiles live in the firmware repo** while vehicle wiring facts belong in packs; no rule for who owns a D2 cluster harness profile | `F/docs/specs/sensor-detection.md` §4.3 vs `P/decisions/adr-0034...md` "Packs are data first" |
| G8 | **No gateway-side telltale→alert rules.** Alerts are app-raised; nothing raises a warning when no app page is open | `S/src/alerts/types.ts`; `S/src/contract/host.ts` |
| G9 | **Freshness is too coarse for instruments:** module-level, 10 s heartbeat and 30 s stale. Instruments need per-signal age budgets in the page contract | `S/src/contract/freshness.ts` |
| G10 | **No cluster display class.** Head units cold-start and the Brain may take up to 60 s to boot; no requirement for "telltales visible within N s of ignition" | ADR-0040 §4.4; `direction_standard_not_os.md` |
| G11 | **Link criticality is undecided:** whether driver warnings are "alarm-critical" (wire-only) or may use Wi-Fi | ADR-0026; ADR-0027 §2 |
| G12 | **VISS isn't served yet.** V0–V4 aren't done; the node still speaks MQTT, and failover from the Brain to the Android gateway mid-drive is "Open" | ADR-0051 §8, §9 |
| G13 | **ADR-0052 and ADR-0053 are still "proposed"** (the MFD, instrument pages in the gateway, hardware detection). The shell spec is approved | `P/decisions/adr-0052...md`, `adr-0053...md` headers |
| G14 | **No "locked primary region".** The owner chose "no fixed tile", so a cluster layout can be edited away while parked. A cluster product may need a protected layout | ADR-0052 §9 item 10 |
| G15 | **The gauge display module is an idea only** (catalogue "next", backlog #29), with no spec or hardware | `addons_catalogue.md`; `features_backlog.md` |
| G16 | **Minor doc drift:** the pack constitution still says "The ESP32 decode header is generated from it", against the platform rule of no generated header | `D/CONSTITUTION.md` vs `P/CONSTITUTION.md`, `P/decisions/adr-0034...md` |

## 7. Suggested framing (from the rules above, not a decision)

1. **Augment first.** Keep the stock cluster wired. Add read-only, opto-isolated taps
   (tacho, road speed, fuel sender, the key lamp lines) on an ESP32-S3 node. Show them
   in a gateway "Cluster" layout with instrument pages and pack-defined warnings. Every
   step is inside existing rules (ADR-0032 §14, shell spec §21).
2. **Replace later, only with ADRs.** A full replacement needs ADRs for:
   - lamp-circuit loads and outputs (G2);
   - a protected cluster display path with a boot-time budget and wired link (G10, G11);
   - legal and odometer requirements (G1).
