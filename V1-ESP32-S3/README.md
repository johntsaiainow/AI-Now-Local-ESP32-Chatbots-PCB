# AI-Now Local ESP32 Chatbots PCB — V1

## ESP32-S3-DevKitC-1 Standard 44-Pin, 2-Layer AI Voice Carrier

**Document release:** V1.0 — Final README  
**Hardware design state:** **Engineering in progress — NOT fabrication-ready**  
**Target board:** Espressif ESP32-S3-DevKitC-1, 44 pins (2 × 22), N16R8 configuration  
**PCB:** 2-layer FR-4 carrier, OSHPark-oriented manufacturing  
**Firmware:** [AI-Now ESP32-Local-Ai-Chatbot](https://github.com/johntsaiainow/ESP32-Local-Ai-Chatbot) — independently developed firmware

> **Release distinction:** This is the complete V1 project README and hardware design specification. It does **not** certify the current KiCad design as electrically complete or production approved. The existing mechanical-baseline package includes an editable DevKit footprint and mapping tables, but not a fully wired schematic, routed PCB, validated ERC/DRC, or Gerbers. Do not submit the baseline files for fabrication.

---

## 1. Project Overview

V1 is the standard **two-layer carrier board** in the AI-Now Local ESP32 Chatbots PCB family. It transforms a breadboard ESP32-S3 voice chatbot into a repeatable, serviceable assembly. The ESP32-S3-DevKitC-1 remains removable for firmware flashing, debugging, and replacement. The carrier adds I2S audio, optional display, user controls, power distribution, and labeled interfaces.

V1 is a **hardware platform for AI-Now's own firmware**. Third-party ESP32 voice-chatbot hardware documentation may inform electrical design, but **XiaoZhi firmware is not a dependency, fork, or required runtime**. Features such as cloud model access, host-assisted speech processing, or local inference depend on the actual firmware build and connected host; this PCB alone does not implement those services.

### Goals

- Use the **official Espressif 44-pin ESP32-S3-DevKitC-1** as a replaceable controller.
- Provide a stable interface for one I2S microphone and one I2S Class-D speaker amplifier.
- Provide optional I2C OLED display and three user buttons.
- Keep programming/debugging through the DevKit's existing USB connection.
- Make the carrier manufacturable as a standard two-layer PCB after design verification.
- Define an explicit GPIO contract and reproducible BOM for future V1 subvariants.

### Out of scope for V1

- Cellular modem, SIM, battery charging, or LTE power subsystem (reserved for V5).
- Custom onboard ESP32-S3 RF/module assembly (a possible future V1 subvariant).
- Claims of verified 3 W audio output or production certification before testing.

## 2. Hardware Architecture

```text
                       USB power / programming
                                |
                   +--------------------------+
                   | ESP32-S3-DevKitC-1 N16R8 |
                   | Official 44-pin, 2 x 22  |
                   +--------------------------+
                      |        |        |    |
                   I2S RX   I2S TX    I2C  GPIO
                      |        |        |    |
                 +---------+ +---------+ |  +----------------+
                 | INMP441 | |MAX98357A| |  | Wake / Vol + / -|
                 | Mic     | | Amp     | |  +----------------+
                 +---------+ +----+----+ |
                                   |      +----> SSD1306 OLED
                              Speaker

                  Carrier PCB: 2-layer FR-4, common GND
```

### Core hardware

| Block | Candidate device / implementation | Required | Notes |
|---|---|---|---|
| Controller | Espressif ESP32-S3-DevKitC-1 N16R8, 44-pin | Yes | Verify actual module marking and board revision |
| Controller sockets | 2 × 1×22, 2.54 mm female headers | Yes | Match physical DevKit pin geometry and height |
| Digital microphone | INMP441 or ICS43434 I2S breakout | Yes | Choose one **specific** vendor/module before footprint lock |
| Audio amplifier | MAX98357A I2S Class-D breakout | Yes | Breakout supply, SD, GAIN, and pinout require validation |
| Speaker | 4 Ω or 8 Ω, suitable rated power | Yes | Differential amplifier output; no speaker lead to GND |
| Display | SSD1306 I2C OLED breakout | Optional | Validate connector order, address, and I2C voltage |
| Controls | Three momentary switches | Yes | Wake and volume up/down |
| Status LED | Optional GPIO-controlled indicator | Optional | Avoid conflicting with onboard RGB LED |
| Power | USB to DevKit; DevKit 3V3 to eligible peripherals | Yes | Regulator budget must be measured |
| Test points | GND, 3V3, selected signals | Recommended | For manufacturing and bring-up |

## 3. Controller and Mechanical Interface

### 3.1 Exact controller target

- **Board family:** Espressif ESP32-S3-DevKitC-1, official standard **44-pin** version.
- **Headers:** 22 pins per side, **2.54 mm** pitch.
- **Nominal header row spacing:** **22.86 mm**, based on the Espressif KiCad footprint reference; physically confirm against the board to be installed.
- **Controller configuration:** ESP32-S3 **N16R8** (16 MB flash, 8 MB PSRAM) as specified for this project; verify actual fitted module marking.
- **USB:** Use the DevKit's built-in connector for initial power and firmware/debug access.

The existing footprint is located at:

```text
footprints/AI_Now.pretty/ESP32-S3-DevKitC-1_44Pin_Carrier.kicad_mod
```

**Mechanical verification is mandatory:** The footprint's provisional **1.0 mm drills** and **1.7 × 2.2 mm pads** are engineering assumptions, not released manufacturing dimensions. Check socket manufacturer's recommended PCB holes, the exact board/header orientation, USB connector access, antenna overhang, component clearance, and any mounting features before routing.

### 3.2 Pin numbering and orientation

The project's footprint uses a working pad-number convention. The authoritative lookup is `docs/DEVKIT_PIN_MAP.csv` in the mechanical-baseline package, which maps each footprint pad to the Espressif **J1/J3** header and position. Do **not** assume a generic 44-pin ESP32-S3 breakout has the same mapping. Cross-check schematic symbol pins, footprint pads, the Espressif pin table, and the physical board in a four-way review.

**Board revision note:** The DevKitC-1 onboard RGB LED GPIO assignment can vary with revision (commonly GPIO48 versus GPIO38); do not hard-code either as a spare carrier GPIO without checking the specific revision.

### 3.3 Placement rules

1. Orient the DevKit so both USB connectors remain accessible after insertion.
2. Keep the ESP32-S3 module antenna area free of carrier copper, planes, traces, mounting metal, and nearby obstructing hardware as required by Espressif's antenna guidance.
3. Ensure the DevKit can be inserted/removed without contacting OLED, speaker connector, or mounting hardware.
4. Provide polarity and **pin-1** markings for every removable module and header.
5. Print the PCB family, revision, and firmware pin-map ID on silkscreen.

## 4. Proposed Electrical Net Assignment

**Status: candidate hardware contract — requires comparison with current AI-Now firmware board definitions before schematic sign-off.** These GPIOs originate from a hardware breadboard reference and are not evidence of implemented firmware support.

| Function | ESP32-S3 GPIO | External net / device | Direction / caution |
|---|---:|---|---|
| Microphone word select | GPIO4 | `MIC_WS` → mic WS/LRCLK | Output |
| Microphone bit clock | GPIO5 | `MIC_BCLK` → mic SCK/BCLK | Output |
| Microphone data | GPIO6 | `MIC_DOUT` ← mic SD | Input |
| Amplifier audio data | GPIO7 | `AMP_DIN` → MAX98357A DIN | Output |
| Amplifier bit clock | GPIO15 | `AMP_BCLK` → MAX98357A BCLK | Output |
| Amplifier word select | GPIO16 | `AMP_LRCLK` → MAX98357A LRC | Output |
| OLED SDA | GPIO41 | `I2C_SDA` | Bidirectional, pull-up required |
| OLED SCL | GPIO42 | `I2C_SCL` | Output/open-drain, pull-up required |
| Volume down | GPIO39 | `BTN_VOL_DOWN_N` | Active low |
| Volume up | GPIO40 | `BTN_VOL_UP_N` | Active low |
| Wake / BOOT candidate | GPIO0 | `BTN_WAKE_N` | **Boot strapping risk**; reconsider dedicated GPIO |

### GPIO0 caution

GPIO0 is a boot-mode strapping pin. A wake button that pulls it low during reset may enter the ROM download boot mode rather than the application. **Preferred final design:** assign Wake to a firmware-confirmed non-strapping free GPIO. If GPIO0 is retained, document the intentional BOOT-button behavior and test power-up/reset with the button held. Do not release the board with an untested GPIO0 wake design.

### Audio channel selection

For a mono INMP441 breakout, connect the channel-select pin (often `L/R`) to the level required by the chosen channel and firmware configuration; do not leave it floating. Confirm microphone breakout voltage and logic levels before assembly. The microphone and amplifier may use **separate I2S clock nets** as shown above; whether clocks can be shared or synchronized depends on the firmware and selected audio architecture.

### I2C display

Use pull-ups to **3.3 V** (typical starting point: 4.7 kΩ, subject to breakout pull-ups and bus capacitance). Do not install duplicate pull-ups blindly. Confirm OLED I2C address (commonly `0x3C`, sometimes `0x3D`) and exact pin order.

## 5. Schematic Design Requirements

The completed KiCad project should contain these functional sheets (hierarchical sheets or clearly labeled sections are both acceptable):

### Sheet A — DevKitC-1 Carrier

- Two 22-pin socket connections with every pad and signal explicitly named.
- Accessible EN/RESET, GND, 3V3, and selected debug/test nets.
- No accidental short between DevKit supply pins or external power sources.
- Explicit `NC` handling for unused pins and a documented GPIO reservation list.

### Sheet B — I2S Microphone

- 3V3 supply and ground.
- WS/BCLK/SD to verified DevKit GPIO nets.
- L/R channel select with a defined default state.
- 100 nF local decoupling close to microphone power pins/module connector.
- Keep mic placement away from speaker magnetic/acoustic coupling and noisy power traces.

### Sheet C — I2S Amplifier / Speaker

- MAX98357A module supply and ground, with 100 nF plus a local bulk capacitor as supported by module design.
- DIN/BCLK/LRCLK from controller.
- `SD`/enable and `GAIN` strapped or controlled **per the exact breakout's documentation**; do not assume all modules expose the same pin order.
- Two-wire speaker output connector for `SPK+` and `SPK−`.
- **Never tie `SPK−` to GND.** The Class-D output is bridge-tied/differential.
- Check speaker impedance, expected output level, thermal behavior, and power budget.

### Sheet D — OLED Display

- 4-pin display interface (3V3, GND, SDA, SCL), subject to actual breakout pinout.
- I2C pull-up provision and optional series/debug pads if required.
- OLED connector orientation clearly marked.

### Sheet E — Buttons / Status

- Volume up and down: momentary switch to GND, with firmware internal pull-up or fitted external pull-up.
- Wake: move to a non-strapping GPIO if possible; otherwise treat as a combined BOOT/Wake function with documented limitations.
- Optional LED with current-limiting resistor, on a verified unused GPIO.
- Consider ESD exposure if buttons or connectors are accessible outside an enclosure.

### Sheet F — Power / Test

- USB 5 V enters through the **DevKit**, not an improvised carrier back-feed path.
- DevKit 3V3 distribution to **3.3 V-compatible** breakout modules only.
- Test points for 3V3, GND, and relevant audio/I2C/control signals.
- Decoupling and bulk capacitance sized by the actual regulator and amplifier load.
- Reserve the option of a future separately engineered amplifier power rail if 3V3 is insufficient.

## 6. Power Budget and Safety

A DevKit 3V3 rail is **not automatically sufficient** for a Wi-Fi-enabled ESP32-S3, OLED, microphone, and loud speaker amplifier operating simultaneously. The final release requires a measured current budget and regulator headroom check.

| Load | Rail | Verification |
|---|---|---|
| ESP32-S3 + Wi-Fi | DevKit power architecture | Peak/transient current, brownouts |
| I2S microphone | 3V3 | Module operating range, ripple/noise |
| SSD1306 display | 3V3 if supported | Startup current and bus levels |
| MAX98357A amplifier | Selected module VIN | Speaker load, volume, peak current, heating |

**Power release gates:** Confirm regulator datasheet and actual DevKit circuit, measure idle/streaming/peak current, verify 3V3 at each peripheral under simultaneous Wi-Fi + audio, and ensure no 5 V signal reaches a 3.3 V-only GPIO. If insufficient, redesign power distribution rather than treating the initial power assumption as validated.

## 7. Two-Layer PCB Layout Requirements

- **Stackup:** 2-layer FR-4, copper weights and thickness selected from the fabricator's current options.
- **Ground:** Favor a continuous bottom GND plane; avoid splitting return paths beneath fast I2S signals.
- **Antenna:** Enforce Espressif RF keepout in both copper layers and mechanical assembly.
- **Microphone:** Locate away from the speaker, amplifier outputs, high-current loops, and enclosure resonance hot spots.
- **Speaker:** Route differential speaker outputs together, away from microphone and high-impedance inputs.
- **Decoupling:** Place at the load/module interface with short GND returns.
- **Headers:** Confirm row spacing, pad drills, solderability, insertion clearance, and USB access with actual parts.
- **Silkscreen:** Mark V1 family, revision, socket orientation, all module pins, `SPK+`/`SPK−`, and power test points.
- **DFM:** Use current OSHPark minimum trace/space, annular ring, hole, and board-edge rules; don't rely on outdated values.

A visually complete PCB is **not** a manufacturing release until all electrical nets are routed or intentionally excluded, zones are refilled, and DRC is clean.

## 8. BOM — Preliminary Procurement Structure

Quantities below describe the target assembly; **exact manufacturer part numbers, supplier URLs, connector footprints, and prices remain TBD** until actual modules are selected. This is not a purchasing-approved BOM.

| Ref | Qty | Item | Design note |
|---|---:|---|---|
| U1 | 1 | Espressif ESP32-S3-DevKitC-1, N16R8, 44-pin | Verify board revision and module marking |
| J1, J3 | 2 | 1×22 2.54 mm female sockets | Confirm height, hole/drill geometry |
| MIC1 | 1 | INMP441 **or** ICS43434 I2S breakout | Lock exact supplier pinout |
| AMP1 | 1 | MAX98357A I2S Class-D breakout | Lock exact supplier pinout and SD/GAIN |
| SPK1 | 1 | 4 Ω or 8 Ω speaker | Match mechanical and power ratings |
| JSPK | 1 | 2-position speaker connector | Match cable, current and pitch |
| DS1 | 0–1 | SSD1306 I2C OLED breakout | Optional; choose exact dimensions |
| SW1–SW3 | 3 | Momentary tactile switches | Select footprint and actuation height |
| C1–C4 | 4 | 100 nF ceramic bypass capacitors | Final placement/value subject to module design |
| C5 | 1 | 10 µF bulk capacitor | Final rating and value subject to power test |
| R1–R2 | 0–2 | I2C pull-up resistors, e.g. 4.7 kΩ | Fit only if required by breakout/bus |
| LED1, RLED | 0–1 each | Status LED and resistor | Optional, firmware GPIO dependent |
| TP* | As needed | Test points | 3V3, GND, critical nets |
| PCB1 | 1 | Custom 2-layer FR-4 carrier | Gerbers not yet released |

A machine-readable baseline is available as `docs/PROVISIONAL_BOM.csv` in the mechanical package. Update it with MPN, footprint, manufacturer, supplier, alternates, and assembly/DNP status before purchasing.

## 9. KiCad Project Organization

Proposed final repository structure:

```text
V1-ESP32-S3-DevKitC-Carrier/
├── README.md
├── hardware/
│   ├── AI-Now-V1.kicad_pro
│   ├── AI-Now-V1.kicad_sch
│   └── AI-Now-V1.kicad_pcb
├── footprints/
│   └── AI_Now.pretty/
│       └── ESP32-S3-DevKitC-1_44Pin_Carrier.kicad_mod
├── symbols/
│   └── AI_Now.kicad_sym
├── docs/
│   ├── DEVKIT_PIN_MAP.csv
│   ├── GPIO-Mapping.md
│   ├── BOM.csv
│   ├── Assembly.md
│   ├── Bring-Up.md
│   └── CHANGELOG.md
├── manufacturing/
│   ├── gerbers/
│   ├── drill/
│   └── fabrication-notes.md
└── firmware/
    └── board-config/
```

**Directory structure is a target, not a claim that every listed file already exists.** In the current mechanical-baseline package, only the documented footprint, pin map, provisional BOM, and supporting notes have been prepared. KiCad symbol, electrically connected schematic, routed PCB, and manufacturing outputs must still be completed.

## 10. Firmware Integration Contract

**Firmware repository:** https://github.com/johntsaiainow/ESP32-Local-Ai-Chatbot

Before electrical sign-off:

1. Identify the exact PlatformIO environment / board definition used for ESP32-S3-DevKitC-1 N16R8.
2. Compare **every proposed GPIO** with the firmware's I2S, display, button, USB, PSRAM, and board-specific definitions.
3. Record microphone and speaker sample format, channel selection, clocking, and duplex mode.
4. Verify the OLED driver, address, pin assignments, and optional-display behavior.
5. Decide whether Wake is a dedicated application input or a shared BOOT button; avoid unintended strapping behavior.
6. Preserve existing USB flashing and debugging capability.
7. Record a firmware commit SHA and matching hardware pin-map revision for each manufactured PCB revision.

Do not label OpenAI, Gemini, local LLM, Whisper, Piper, or host-agent integration as a **hardware feature**. Document such capabilities separately as firmware/host features and mark them supported only after testing the specific build.

## 11. Validation and Bring-Up

### Pre-fabrication checklist

- [ ] Confirm DevKit header spacing, physical orientation, socket height, USB clearance, and antenna keepout against actual hardware.
- [ ] Confirm pin-to-pad mapping against official J1/J3 tables and the real DevKit.
- [ ] Lock exact microphone, amplifier, OLED, speaker connector, and switch part numbers/footprints.
- [ ] Confirm firmware GPIO mapping and resolve GPIO0 boot-mode interaction.
- [ ] Complete electrically wired KiCad schematic; pass ERC with reviewed exceptions only.
- [ ] Complete 2-layer routing; refill copper zones and pass DRC.
- [ ] Validate 3V3 power budget and MAX98357A supply/enable/gain wiring.
- [ ] Inspect PCB in 3D and check all mechanical dimensions.
- [ ] Export and independently inspect Gerbers and drill files.
- [ ] Freeze BOM, assembly drawing, board revision, and fabrication notes.

### First-article bring-up checklist

- [ ] Inspect unpowered board and check 3V3-to-GND for shorts.
- [ ] Verify all connectors and module orientations before insertion.
- [ ] Power the DevKit through its normal USB input and measure all rails.
- [ ] Flash AI-Now firmware and verify USB console and Wi-Fi operation.
- [ ] Capture microphone audio and evaluate noise floor.
- [ ] Test low-volume speaker output; verify `SPK−` is not grounded.
- [ ] Test OLED address, startup, and display refresh.
- [ ] Test all buttons, including reset/boot while Wake is held.
- [ ] Test simultaneous Wi-Fi, microphone capture, and speaker playback for brownouts or resets.
- [ ] Record thermal, acoustic, and current measurements before approving fabrication revision.

## 12. Revision and Subvariant Policy

V1 is a **family**, not a single immutable PCB. Future variants may use the following identifiers:

| Identifier | Meaning | Example |
|---|---|---|
| `V1` | ESP32-S3 standard 2-layer family | V1 |
| `V1.0` | Initial DevKitC-1 carrier subvariant | V1.0 |
| `V1.1` | Future display/mechanical variation | Proposed only |
| `V1.2` | Future audio/power variation | Proposed only |
| `R1.0` | Released PCB revision | Only after validation |

Example full designation: `AI-Now-V1.0-R1.0`. Keep schematic, PCB, BOM, Gerber, assembly instructions, and firmware mapping aligned to the same revision. A **Final README** is documentation status; it is **not** equivalent to a released `R1.0` PCB.

## 13. Official References

- [Espressif ESP32-S3-DevKitC-1 User Guide](https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32s3/esp32-s3-devkitc-1/user_guide_v1.1.html)
- [Espressif KiCad Footprint — ESP32-S3-DevKitC](https://github.com/espressif/kicad-libraries/blob/main/footprints/Espressif.pretty/ESP32-S3-DevKitC.kicad_mod)
- [ESP32-S3 Technical Reference / Documentation](https://docs.espressif.com/projects/esp-idf/en/stable/esp32s3/index.html)
- [AI-Now ESP32-Local-Ai-Chatbot Firmware](https://github.com/johntsaiainow/ESP32-Local-Ai-Chatbot)
- [AI-Now PCB Templates](https://github.com/johntsaiainow/AI-Now-PCB-Templates)
- [Hardware breadboard reference (Feishu)](https://my.feishu.cn/wiki/EH6wwrgvNiU7aykr7HgclP09nCh) — hardware reference only; not a firmware dependency

## 14. Project Release Status

| Deliverable | Current status |
|---|---|
| V1 requirements and hardware specification | **Documented** |
| Official 44-pin controller choice | **Selected** |
| Editable DevKit carrier footprint | **Preliminary — physical verification required** |
| 44-pin J1/J3 mapping CSV | **Prepared — cross-check required** |
| Preliminary BOM | **Prepared — MPNs/footprints TBD** |
| Complete connected schematic | **Not yet completed** |
| KiCad ERC | **Not yet passed** |
| Routed two-layer PCB | **Not yet completed** |
| KiCad DRC | **Not yet passed** |
| Gerber/drill manufacturing package | **Not yet generated** |
| Assembled hardware functional test | **Not yet performed** |

**The next engineering milestone is a verified, fully connected KiCad schematic and routed PCB — not another README revision.**

---

**AI-Now Technology — V1 ESP32-S3 Standard 2-Layer AI Voice Carrier**  
Hardware platform for the independently developed AI-Now ESP32-Local-Ai-Chatbot firmware.
