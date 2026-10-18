# AI-Now Local ESP32 Chatbots PCB

**An expandable family of ESP32-based AI voice terminal PCBs, engineered for the independent AI-Now Local AI Chatbot firmware.**

[![Hardware](https://img.shields.io/badge/Hardware-ESP32%20Family-blue)](https://www.espressif.com/)
[![EDA](https://img.shields.io/badge/EDA-KiCad-green)](https://www.kicad.org/)
[![Status](https://img.shields.io/badge/Status-Planning%20%2F%20Design-yellow)](#development-status)

> **Project status:** This README defines the proposed hardware family, naming system, design requirements, and release process. It does **not** claim that schematics, PCB layouts, Gerbers, firmware ports, or production tests have been completed.
>
> **Firmware independence:** This is hardware for **[AI-Now ESP32 Local AI Chatbot](https://github.com/johntsaiainow/ESP32-Local-Ai-Chatbot)**, a separately developed firmware project. **No XiaoZhi firmware is used, forked, or required.** External breadboard guides may be used only as electrical reference material after verification.

## Contents

1. [Vision and Scope](#vision-and-scope)
2. [Five Main Hardware Families](#five-main-hardware-families)
3. [Expandable Subvariant and Revision System](#expandable-subvariant-and-revision-system)
4. [System Architecture](#system-architecture)
5. [Hardware Family Specifications](#hardware-family-specifications)
6. [Common Functional Blocks](#common-functional-blocks)
7. [Firmware and Host Integration](#firmware-and-host-integration)
8. [PCB and Manufacturing Standards](#pcb-and-manufacturing-standards)
9. [Repository Organization](#repository-organization)
10. [Design and Release Workflow](#design-and-release-workflow)
11. [Verification and Acceptance](#verification-and-acceptance)
12. [Contribution and Documentation Rules](#contribution-and-documentation-rules)
13. [Roadmap](#roadmap)
14. [References, License, and Maintainers](#references-license-and-maintainers)

## Vision and Scope

The project transforms experimental ESP32 voice-chat hardware into a **maintainable, extensible PCB platform**. It separates physical board engineering from application firmware while giving each board a documented firmware integration contract.

### Design principles

- **Own the stack:** AI-Now hardware and AI-Now custom firmware, without a dependency on third-party chatbot firmware.
- **Local-first:** Support Linux-hosted speech recognition, synthesis, and language-model services when supported by the board and firmware transport.
- **Five permanent top-level families:** V1 through V5 describe *product classes*, not five one-off PCB files.
- **Unlimited subvariants:** Each family may evolve into multiple display, audio, connector, power, enclosure, and connectivity configurations.
- **Reproducible fabrication:** Publish the exact schematic, PCB revision, manufacturing outputs, BOM, and validation record for every released board.
- **Explicit capabilities:** Never assume that USB, I2S, PSRAM, cellular networking, or on-device inference works identically across chips.
- **Design for test:** Provide accessible test points, power measurements, recovery access, and clear hardware identifiers.

### In scope

KiCad schematics and layouts, board-specific hardware specifications, reusable validated circuits, BOMs, 3D/mechanical files, assembly guides, manufacturing outputs, bring-up procedures, test results, and board-to-firmware integration documentation.

### Out of scope

A replacement chatbot firmware, a fork of XiaoZhi, undocumented promises of cloud-provider support, and a claim that all ESP32 chips can run the same binary. Firmware development remains in the companion repository.

## Five Main Hardware Families

| Family | Official name | MCU target | PCB stackup | Primary mission |
|---|---|---|---|---|
| **V1** | **ESP32-S3 Standard** | ESP32-S3 | **2-layer** | Accessible reference board and rapid prototyping |
| **V2** | **ESP32-C3 Compact** | ESP32-C3 | **2-layer** | Minimal, lower-cost voice endpoint |
| **V3** | **ESP32 Classic** | Original ESP32 | **2-layer** | Legacy-friendly voice terminal |
| **V4** | **ESP32-S3 Advanced AI Voice Node** | ESP32-S3 | **4-layer** | Higher-integration audio and compact system design; optional compatibility-mode subvariants |
| **V5** | **ESP32 Mobile Cellular Node** | ESP32-family MCU, initially ESP32-S3 candidate | **4-layer** | Cellular-connected mobile voice terminal |

**Naming invariant:** V1 is the **2-layer ESP32-S3** board family. V4 is the **4-layer advanced AI Voice Node** family. A compatibility pinout is a *subvariant or firmware profile* of V4, not the definition of the whole V4 family.

## Expandable Subvariant and Revision System

The five main versions remain stable while new hardware designs are added below them.

### Identifier format

```text
V<family>.<subvariant>-R<pcb-revision>

Examples:
V1.0-R1.0    ESP32-S3 standard reference board, first PCB release
V1.1-R1.0    ESP32-S3 with alternative display connector
V2.0-R1.0    ESP32-C3 minimal voice board
V4.2-R1.1    Advanced S3 node with compatibility-oriented pinout, revised PCB
V5.1-R1.0    Cellular node with a specific modem option
```

- **V1–V5:** permanent product families, indicating target architecture and baseline layer count.
- **`.0`, `.1`, `.2`, ...:** independent *subvariants* with different feature sets, footprints, interfaces, or board assemblies. The example allocations are **proposals**, not completed boards.
- **`R1.0`, `R1.1`, `R2.0`, ...:** physical PCB design revisions **within one subvariant**. A change affecting fabrication or electrical behavior must be tracked.
- **Firmware version:** separate from PCB revision; record tested firmware commit and board profile in each release manifest.
- **Assembly option:** optional suffix or BOM variant (for example, display fitted/not fitted) when the PCB copper is unchanged.

### Proposed subvariant catalog

| Family | Example subvariant | Concept | Status |
|---|---|---|---|
| V1 | V1.0 | Basic ESP32-S3, microphone, amplifier, USB | Proposed |
| V1 | V1.1 | Standard S3 with SSD1306 OLED header | Proposed |
| V1 | V1.2 | S3 with extended GPIO / development headers | Proposed |
| V2 | V2.0 | ESP32-C3 headless voice endpoint | Proposed |
| V2 | V2.1 | ESP32-C3 with compact display | Proposed |
| V3 | V3.0 | Original ESP32 reference audio board | Proposed |
| V3 | V3.1 | Original ESP32 with USB-UART programming interface | Proposed |
| V4 | V4.0 | Advanced S3 voice node, optimized audio and power | Proposed |
| V4 | V4.1 | Advanced S3 voice node with integrated display | Proposed |
| V4 | V4.2 | Advanced S3 voice node with compatibility-oriented connectors / GPIO mapping | Proposed |
| V5 | V5.0 | Cellular voice node with modem candidate A | Proposed |
| V5 | V5.1 | Cellular voice node with modem candidate B | Proposed |
| V5 | V5.2 | Battery-powered cellular configuration | Proposed |

> The table is a **roadmap**, not an assertion of existing files or validated hardware. New subvariants may be added without renumbering or renaming other families.

### What triggers a new subvariant?

Create a new subvariant when changing the MCU/module package, modem type, audio topology, major connector scheme, power source, form factor, display architecture, or materially different GPIO/firmware board profile. Use a new **PCB revision** for corrections and improvements to the same subvariant. Record both in a changelog.

### Minimum per-subvariant metadata

```yaml
board_id: V1.0
pcb_revision: R1.0
family: V1
status: proposed
mcu: ESP32-S3
layers: 2
firmware_repository: https://github.com/johntsaiainow/ESP32-Local-Ai-Chatbot
firmware_board_profile: TBD
firmware_tested_commit: null
schematic_review: pending
drc: pending
bringup: pending
fabrication_release: false
```

This example is a proposed `board.yaml` schema, not a current firmware configuration file.

## System Architecture

```text
                          AI-Now AI Services
             +------------------------------------------+
             | Linux host / local STT / local TTS / LLM |
             | Optional cloud integrations via backend  |
             +---------------------+--------------------+
                                   |
                 USB / Wi-Fi / Cellular* (as supported)
                                   |
             +---------------------v--------------------+
             | AI-Now Custom ESP32 Local AI Chatbot      |
             | Firmware + chip target + board profile   |
             +---------------------+--------------------+
                                   |
             +---------------------v--------------------+
             | PCB subvariant Vx.y-Rz                   |
             |                                        |
             | I2S Mic ---- MCU ---- I2S Amp -> Speaker|
             |                |                        |
             |          I2C Display / GPIO UI          |
             |                |                        |
             |       USB / UART / Wi-Fi / LTE*          |
             +-----------------------------------------+

* Cellular is a V5 capability only after modem and firmware integration.
```

The ESP32 is primarily a **voice and control endpoint**; demanding speech processing and LLM inference can run on a separate host. On-device capabilities depend on the actual MCU, memory, and firmware build.

## Hardware Family Specifications

### V1 — ESP32-S3 Standard · 2-Layer

**Goal:** A transparent, affordable baseline that is easy to inspect, assemble, debug, and reproduce.

**Proposed features:** ESP32-S3 module (N16R8 is a candidate), digital I2S microphone, I2S speaker amplifier, USB data/power where supported, accessible GPIO and test points, buttons, and optional OLED header.

**Design priorities:** Short audio routes; robust 3.3 V supply; ESP32-S3 boot/USB/strapping compliance; antenna keepout; adequate ground copper on a two-layer stackup; optional module/header population.

**Validation gates:** Correct boot mode, flash/PSRAM settings, reliable USB transport, microphone capture, speaker playback, simultaneous or alternating audio according to the driver, Wi-Fi stability, and brownout testing.

### V2 — ESP32-C3 Compact · 2-Layer

**Goal:** Small, cost-sensitive terminal that delegates expensive AI processing to a host.

**Proposed features:** ESP32-C3 module, simplified audio path, minimal buttons and LEDs, optional compact display, Wi-Fi and supported serial/debug interface.

**Engineering constraints:** Fewer pins and less memory than S3; a single I2S controller; native USB Serial/JTAG is **not equivalent** to the S3's general USB OTG peripheral. Audio full-duplex assumptions, memory usage, and USB protocols require a dedicated design.

**Validation gates:** GPIO allocation, available RAM/flash, I2S clocking, host transport, startup latency, watchdog behavior, and thermal/current budget.

### V3 — Original ESP32 Classic · 2-Layer

**Goal:** Support widely available original ESP32 modules and preserve a useful legacy hardware path.

**Proposed features:** Original ESP32-WROOM-32-class module, I2S microphone and amplifier, Wi-Fi, buttons, optional I2C display, UART debug/programming.

**Engineering constraints:** No S3-style native USB device peripheral; provide an external USB-UART bridge when wired USB serial is required. Respect strapping pins, ADC limitations, module antenna geometry, and the exact flash/PSRAM configuration.

**Validation gates:** UART flashing/recovery, audio drivers, network connection, reset and bootstrapping, and board-specific feature reductions.

### V4 — ESP32-S3 Advanced AI Voice Node · 4-Layer

**Goal:** A more integrated voice endpoint with improved power/ground architecture and audio performance.

**Proposed features:** ESP32-S3 with appropriate flash/PSRAM, I2S digital audio, improved decoupling, optimized speaker drive, optional integrated display, control inputs, expansion/debug, and compatibility-oriented *subvariants* where needed.

**Layer intent:** Signal / continuous ground reference / power or mixed plane / signal. Final stackup and controlled-impedance needs must be confirmed with the selected PCB fabricator; do not assume a generic four-layer stackup is electrically interchangeable.

**Compatibility mode:** A V4 subvariant may reproduce a specified legacy header or GPIO layout **for the AI-Now firmware port**. It does not imply XiaoZhi firmware compatibility.

**Validation gates:** Noise floor, playback distortion, microphone interference, power transients, ESD/EMI considerations, enclosure acoustics, and display/USB coexistence.

### V5 — ESP32 Mobile Cellular Node · 4-Layer

**Goal:** A self-contained voice endpoint able to connect over cellular data after a dedicated modem firmware integration.

**Proposed features:** ESP32-family controller (initial S3 candidate), LTE Cat.1 modem, SIM interface where needed, RF antenna connector or validated antenna, I2S audio, status UI, service/debug port, and optional battery architecture.

**Modem candidates:** ML307R and NT26K are *evaluation candidates*, not pin-compatible or electrically interchangeable parts. Each modem requires a separate subvariant unless a fully verified shared design is feasible.

**Power and RF requirements:** Size the modem supply for documented transmit current peaks and voltage droop; follow module-specific VBAT, enable, logic-level, SIM, and antenna requirements. Never tap an unidentified regulator pin as a power supply. Follow RF keepouts and maintain adequate separation from microphone/audio traces.

**Networking requirements:** Explicit UART direction and flow-control mapping; AT/control firmware or other selected modem interface; connection state machine, reconnection, network timeouts, and error recovery. Wi-Fi fallback is optional and must be implemented, not assumed.

**Validation gates:** Peak current, RF performance, audio interference during transmit, SIM detection, connection/reconnection, power sequencing, antenna safety, and applicable compliance review.

## Common Functional Blocks

| Block | Candidate / interface | Mandatory design review |
|---|---|---|
| MCU | S3 / C3 / original ESP32 | Datasheet, strapping pins, flash/PSRAM, antenna keepout |
| Microphone | INMP441, ICS43434, validated equivalents | Logic voltage, clock timing, channel-select polarity, mechanical port |
| Amplifier | MAX98357A, HT517, validated alternatives | Supply, shutdown/gain behavior, output topology, speaker impedance |
| Speaker | 4 Ω / 8 Ω, 2–3 W candidate | Power rating, current, acoustics, thermal constraints |
| Display | SSD1306 I2C candidate | Pull-ups, I/O voltage, actual panel driver and pin order |
| Controls | Buttons, LED/RGB | Debounce, boot-strapping effects, ESD, resistor sizing |
| Wired host | Native USB or USB-UART | MCU-specific controller capability, ESD and connector wiring |
| Power | USB 5 V, 3.3 V regulation | Protection, inrush, regulator headroom, ripple, test points |
| Cellular | LTE modem / SIM / antenna | Vendor-specific power, RF, interface voltage, regulatory review |

### Audio routing

- Keep digital microphone clock/data routes short and referenced to ground.
- Verify input/output I2S controller assignments against the actual firmware drivers.
- Do not assume that all I2S peripherals can share a single clock configuration.
- Keep speaker switching currents away from microphone and analog-sensitive regions.
- Many class-D amplifiers use **bridge-tied outputs**: neither speaker terminal may be tied to GND unless explicitly permitted by the datasheet.
- Never copy amplifier `GAIN`, `SD`, or `EN` straps from an unrelated breakout board without verifying the selected module.

### Power architecture

1. Document input source(s) and allowable voltage range.
2. Protect against reverse polarity, ESD, and damaging overcurrent where appropriate.
3. Calculate MCU, audio, display, and modem **peak**, not just average, demand.
4. Check startup sequencing and brownout behavior.
5. Expose GND, 3V3, and relevant additional rails at labeled test points.
6. Require measured power-rail verification before first firmware bring-up.

### GPIO policy

**No global GPIO table is declared authoritative yet.** Each subvariant must publish its own reviewed `pinout.md` and a machine-readable mapping where feasible. GPIO allocations must be checked against boot straps, USB pins, JTAG/debug, flash/PSRAM connections, I2S clocks, I2C, UART direction, and module-specific restrictions.

## Firmware and Host Integration

**Firmware source of truth:** [ESP32-Local-Ai-Chatbot](https://github.com/johntsaiainow/ESP32-Local-Ai-Chatbot)

**Hardware source of truth:** This PCB repository, with one board directory and revision record per subvariant.

A board is not firmware-compatible merely because it uses an ESP32-S3. Firmware ports must explicitly account for:

- Build target, SDK/toolchain, flash/PSRAM mode and partitions.
- Audio codec/driver and microphone/speaker topology.
- Display controller, resolution, pins, and UI capabilities.
- USB host/device/serial semantics and external USB-UART bridges.
- Wi-Fi and optional cellular transport implementation.
- Recovery buttons, watchdog, NVS settings, and provisioning.
- Memory and latency budgets for audio buffering and networking.

### Board support contract

Every subvariant's `firmware-integration.md` should document:

```text
Board ID / PCB revision
MCU module and flash/PSRAM configuration
Build environment / firmware board profile
Pin assignment table
Audio capture and playback settings
Display and controls
USB / UART / Wi-Fi / LTE transport
Tested firmware commit SHA
Known unsupported features
Bring-up and recovery procedure
```

**Note:** The existing firmware may target specific development boards. Porting to the proposed PCBs is a planned engineering activity, not a claim of out-of-the-box binary compatibility. OpenAI/Gemini integration, if implemented, belongs in the appropriate firmware or host/backend layer rather than in the PCB itself.

## PCB and Manufacturing Standards

Designs are authored in **KiCad** and may use the companion [AI-Now PCB Templates](https://github.com/johntsaiainow/AI-Now-PCB-Templates) as a starting point. Fabrication outputs target **OSHPark or another documented fabricator** only after checking that service's *current* design rules.

### Required checks

- Confirm stackup, copper thickness, trace/space, drill, annular ring, edge clearance, solder mask, and board outline against the actual order type.
- Use approved footprints with manufacturer land-pattern references.
- Run KiCad ERC and DRC; record any intentional waivers with justification.
- Review antenna/module keepouts, mounting holes, connector access, and enclosure clearances.
- Validate supply widths, return paths, thermal performance, and decoupling.
- Inspect Gerber layers and Excellon drills in an independent viewer before release.
- Never label a design **fabrication-ready** until a reviewed release package exists.

### Release package

```text
releases/
└── V1.0-R1.0/
    ├── board.yaml
    ├── CHANGELOG.md
    ├── schematic.pdf
    ├── bom.csv
    ├── pinout.md
    ├── fabrication/
    │   ├── gerbers.zip
    │   └── drill-files/
    ├── assembly/
    │   ├── placement.csv
    │   └── assembly-notes.md
    └── validation/
        ├── test-plan.md
        └── test-results.md
```

This is a **target release structure**, not a list of currently available deliverables.

## Repository Organization

```text
AI-Now-Local-ESP32-Chatbots-PCB/
├── README.md
├── LICENSE                       # Add after choosing a license
├── CONTRIBUTING.md
├── docs/
│   ├── architecture.md
│   ├── family-matrix.md
│   ├── subvariant-policy.md
│   ├── electrical-design-rules.md
│   ├── firmware-integration.md
│   ├── manufacturing-guide.md
│   └── validation-plan.md
├── hardware/
│   ├── V1-ESP32-S3-2Layer/
│   │   ├── V1.0-Standard/
│   │   ├── V1.1-OLED/
│   │   └── V1.2-Expansion/
│   ├── V2-ESP32-C3-2Layer/
│   │   ├── V2.0-Minimal/
│   │   └── V2.1-Display/
│   ├── V3-ESP32-Classic-2Layer/
│   │   ├── V3.0-Reference/
│   │   └── V3.1-USB-UART/
│   ├── V4-ESP32-S3-VoiceNode-4Layer/
│   │   ├── V4.0-Advanced/
│   │   ├── V4.1-Display/
│   │   └── V4.2-Compatibility/
│   ├── V5-ESP32-Cellular-4Layer/
│   │   ├── V5.0-Modem-A/
│   │   ├── V5.1-Modem-B/
│   │   └── V5.2-Battery/
│   └── common/
│       ├── symbols/
│       ├── footprints/
│       └── schematic-blocks/
├── docs-per-board/                # Board-specific pinouts and integration notes
├── tests/
│   ├── electrical/
│   ├── audio/
│   ├── connectivity/
│   └── production/
├── releases/                      # Immutable fabrication release packages
└── images/
```

All listed subvariant folders are **proposed placeholders**. Only create them as the corresponding design work begins. Common libraries should be versioned and reviewed; avoid silently changing released footprints.

## Design and Release Workflow

### Stage 0 — Requirements

Define use case, interfaces, mechanical envelope, target cost, power source, firmware feature requirements, and board identifier.

### Stage 1 — Component selection

Select exact MCU module, microphone, amplifier, regulator, connectors, display, and optional modem. Collect authoritative datasheets and reference layouts.

### Stage 2 — Schematic and pin allocation

Draft power tree, boot/reset/programming, audio, UI, networking, and test points. Review MCU pin restrictions. Run ERC.

### Stage 3 — PCB layout

Apply the family layer count, current fabricator rules, antenna keepouts, grounding strategy, decoupling placement, and mechanical constraints. Run DRC.

### Stage 4 — Fabrication review

Review schematic-to-PCB consistency, BOM availability, Gerbers, drills, polarity/orientation, silkscreen identifiers, and release manifest.

### Stage 5 — Bring-up

Perform unpowered continuity checks, current-limited power-on, rail measurements, programming/recovery, peripheral smoke tests, and firmware integration.

### Stage 6 — Qualification and release

Capture repeatable test results and known issues. Tag the PCB revision and archive the exact fabrication package. A failed gate keeps the design in prototype status.

## Verification and Acceptance

| Test | V1 | V2 | V3 | V4 | V5 |
|---|:---:|:---:|:---:|:---:|:---:|
| Power rails / startup / brownout | ✓ | ✓ | ✓ | ✓ | ✓ |
| Boot, flash, recovery | ✓ | ✓ | ✓ | ✓ | ✓ |
| Microphone capture | ✓ | ✓ | ✓ | ✓ | ✓ |
| Speaker playback | ✓ | ✓ | ✓ | ✓ | ✓ |
| Host communication (supported transport) | ✓ | ✓ | ✓ | ✓ | ✓ |
| Wi-Fi connectivity (if fitted) | ✓ | ✓ | ✓ | ✓ | Optional |
| OLED / buttons (if fitted) | Optional | Optional | Optional | Optional | Optional |
| Audio noise and interference | ✓ | ✓ | ✓ | ✓ | ✓ |
| Cellular attach, reconnect, RF / power peaks | — | — | — | — | ✓ |
| Battery charge/discharge safety (if fitted) | — | — | — | — | Conditional |

A checkmark means **required for future acceptance**, not that testing has passed. Store measured outcomes per PCB revision, including firmware commit, test conditions, and known deviations.

## Contribution and Documentation Rules

1. Open an issue describing the proposed board or revision and its parent family.
2. Use the `Vx.y-Rz` naming convention and never reuse a released identifier for a different electrical design.
3. Commit native KiCad source, not only images or Gerbers.
4. Include schematic references, exact parts, pinout, power budget, and board metadata.
5. Document compatibility claims with tested firmware commit IDs.
6. Do not import third-party reference schematics, symbols, or footprints without checking their licenses and electrical correctness.
7. Avoid undocumented hardware substitutions; specify alternatives in the BOM and validate each one.
8. Record ERC/DRC results, review notes, and bring-up results before marking a board released.

## Roadmap

- [ ] Audit the existing AI-Now firmware source, peripheral drivers, and host transport.
- [ ] Freeze the initial V1.0 requirements and exact ESP32-S3 module.
- [ ] Publish V1.0 pinout, power budget, and KiCad schematic.
- [ ] Complete V1.0 2-layer PCB and prototype validation.
- [ ] Port and verify firmware against the V1.0 board profile.
- [ ] Derive V4.0 4-layer advanced voice node from validated functional blocks.
- [ ] Evaluate and prototype V2.0 ESP32-C3 compact configuration.
- [ ] Evaluate and prototype V3.0 original ESP32 configuration.
- [ ] Select modem and supply architecture for V5.0 cellular configuration.
- [ ] Publish release packages, BOMs, assembly instructions, and test records for validated revisions.
- [ ] Expand subvariants according to display, enclosure, audio, power, and connectivity requirements.

## Development Status

**Architecture and documentation planning.** No PCB design or firmware compatibility test is represented as completed by this README. Each subvariant will have an independent status in its `board.yaml` and release documentation.

## References, License, and Maintainers

### Related AI-Now repositories

- **Custom firmware:** [ESP32-Local-Ai-Chatbot](https://github.com/johntsaiainow/ESP32-Local-Ai-Chatbot)
- **PCB templates:** [AI-Now-PCB-Templates](https://github.com/johntsaiainow/AI-Now-PCB-Templates)

### Engineering references

- [Espressif ESP32-S3](https://www.espressif.com/en/products/socs/esp32-s3)
- [Espressif ESP32-C3](https://www.espressif.com/en/products/socs/esp32-c3)
- [Espressif ESP32](https://www.espressif.com/en/products/socs/esp32)
- [KiCad](https://www.kicad.org/)
- [OSHPark fabrication documentation](https://docs.oshpark.com/)

📜 License
This repository uses dual licensing to properly cover both hardware designs and soft/firmware components:

Hardware Designs (/hardware directory & schematic files):

Licensed under the CERN Open Hardware Licence Version 2 - Strongly Reciprocal (CERN-OHL-S).

You are free to copy, modify, and distribute these hardware files, provided that any derivative designs are also shared under the CERN-OHL-S license.

Documentation & Media (/docs directory & visual assets):

Licensed under Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0).

Firmware Source Code (/firmware directory):

Licensed under the GNU General Public License v3.0 (GPL-3.0).

🤝 Contributing

Contributions are welcome! Please open an issue to discuss proposed schematic changes or submit a pull request for firmware enhancements and documentation fixes.
### Maintainer

**AI-Now Technology, Inc. — Hardware and Embedded AI Engineering**

---

*Five hardware families. Many future subvariants. One independently engineered AI-Now voice platform.*
