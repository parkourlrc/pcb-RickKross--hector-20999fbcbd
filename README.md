# Hector

<!-- TODO: replace with a real photo once available -->
![Hector](docs/build-guide/images/hero.jpg)

**Hector** is a 38-key split column-staggered keyboard with Hall Effect (magnetic) switches. The layout is inspired by [Totem](https://github.com/GEIGEIGEIST/TOTEM) / [TOTEM MX](https://github.com/azhizhinov/TOTEMX) and designed for MX-compatible magnetic switches.

Everything — schematic, PCB, case and firmware — was designed and built from scratch. Rev 1 is a working prototype: one complete keyboard has been built and hand-soldered without a hot plate. The firmware is around 95% complete and already fully usable.

[Build Guide](docs/build-guide/build-guide.md) · [Firmware](https://github.com/RickKross/qmk-config-hector)

## Features

- **Direct ADC sensing** — an STM32F303RE on each half has enough ADC inputs to connect every Hall sensor directly, without external multiplexers. All sensors are sampled continuously via DMA with no CPU involvement.
- **High scan rate** — around 3000 scans per second.
- **High resolution** — the STM32 ADC is native 12-bit, but only ~25% of its range is usable due to magnet polarity, strength and sensor precision. The firmware oversamples up to 14 bits and normalizes the usable portion back to the full 12-bit range: 4096 effective levels across the whole ~4 mm travel, roughly 0.001 mm per level.
- **Two operating modes**:
  - *Rapid Trigger* — uses all 4096 levels for smooth, high-resolution actuation and release detection;
  - *Classic mode* — a fixed actuation point set as a percentage (0–100%) of key travel.
- **Any magnet polarity** — magnetic switches with any magnet polarity can be used, and different polarities can even be mixed within the same keyboard.
- **Automatic calibration** — minimum and maximum values for every key are calibrated automatically during operation.
- **Flexible sensor choice** — any **linear** (ratiometric) Hall sensor in the same SOT-23 footprint should work (*in theory, not tested!*), as long as it is comfortable at 3.3 V. Switching and latching types are not usable, since they have a fixed trip point instead of a proportional output. Currently Honeywell **SS49E** is used and verified.
- **USB-C interconnect** — the halves communicate over UART using a regular USB-C ↔ USB-C cable, avoiding the hot-plugging issues of TRRS connectors.
- **Hand-solderable** — the PCB can be assembled by hand with a regular soldering iron, no hot plate required.
- **Wired only** — continuous analog sensing and the high scan rate draw around 100 mA, so a comfortable wireless experience is unrealistic by design.

<!-- TODO: add layout image (e.g. from keyboard-layout-editor / keyboard-layout.xyz) -->

## Bill of Materials

Quantities are for one complete keyboard (a single PCB contains both halves). For component placement, open the **interactive BOM**: [rendered version (htmlpreview)](https://htmlpreview.github.io/?https://github.com/RickKross/hector/blob/main/pcb/rev1/bom/ibom.html) · [local file](pcb/rev1/bom/ibom.html).

| Part | Qty | References                                 | Notes                                                                                                                                                                                                                                                                   |
| --- |-----|--------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Hector PCB rev1 | 1   | —                                          | order from [`pcb/rev1/hector_rev1.zip`](pcb/rev1/hector_rev1.zip); FR4, 1.6 mm, HASL, color of your choice                                                                                                                                                              |
| STM32F303RET6 | 2   | U1, U4                                     | one MCU per half                                                                                                                                                                                                                                                        |
| Hall sensor | 38  | HS1–HS38                                   | reference part: Honeywell SS49E (tested). Any sensor in the same footprint should work (untested), but it must be **linear** (ratiometric) — switching/latching types have a fixed trip point and won't work as intended — and it must operate comfortably at **3.3 V** |
| USB-C receptacle | 4   | J1–J2, J8–J9                               | any USB-C 16-pin SMT receptacle will fit (reference: G-Switch GT-USB-7010ASV); one host + one interconnect port per half                                                                                                                                                |
| USBLC6-2SC6 | 2   | U2, U5                                     | USB ESD protection                                                                                                                                                                                                                                                      |
| RT9013-33GB | 2   | U3, U6                                     | 3.3 V LDO                                                                                                                                                                                                                                                               |
| 8 MHz oscillator | 2   | OSC1, OSC2                                 | must be an **active** oscillator, not a passive crystal                                                                                                                                                                                                                 |
| Fuse 350 mA | 2   | F1–F2                                      |                                                                                                                                                                                                                                                                         |
| Ferrite bead BLM18KG121TN1D | 4   | FB1–FB4                                    |                                                                                                                                                                                                                                                                         |
| Capacitor 100 nF | 52  | C2–C7, C12, C14–C32, C34–C39, C44, C46–C64 | 38× sensor decoupling + MCU decoupling                                                                                                                                                                                                                                  |
| Capacitor 1 uF | 10  | C8–C11, C40–C43, C65–C66                   |                                                                                                                                                                                                                                                                         |
| Capacitor 10 uF | 4   | C1, C13, C33, C45                          |                                                                                                                                                                                                                                                                         |
| Resistor 5.1 kΩ | 4   | R1–R2, R7–R8                               | USB-C CC pull-downs                                                                                                                                                                                                                                                     |
| Resistor 5.1 kΩ (LED) | 2   | R3, R9                                     | *optional* — in series with the status LEDs. The same as above                                                                                                                                                                                                          |
| Resistor 40 kΩ | 4   | R5–R6, R11–R12                             |                                                                                                                                                                                                                                                                         |
| Resistor 1.5 kΩ | 2   | R4, R10                                    | USB D+ line pull-up                                                                                                                                                                                                                                                     |
| LED (red) | 2   | D1–D2                                      | +3.3 V status LED; *optional* — most likely not needed                                                                                                                                                                                                                  |
| Tactile switch TL3342 | 4   | SW1–SW2, SW4–SW5                           | RESET + BOOT per half                                                                                                                                                                                                                                                   |
| Pin header 1×4 | 2   | J6, J13                                    | *optional* — SWD debug (SWCLK/SWIO), most likely not needed                                                                                                                                                                                                             |
| Pin header 1×2 | 6   | J3–J5, J10–J12                             | *optional* — debug/option jumpers, most likely not needed                                                                                                                                                                                                               |
| MX-compatible magnetic switches | 38  | —                                          | *not part of the electronics BOM*; Gateron KS-20, AGS Radiant HE and Gravastar UFO have been tested                                                                                                                                                                     |
| Keycaps (MX) | 38  | —                                          | see note below                                                                                                                                                                                                                                                          |

### Silkscreen note

The silkscreen is kept intentionally minimal, for a cleaner look at the board:

- The generic **100 nF capacitors have no references printed on the board**. If you see an empty spot with two "pins", that's where one goes.
- Every other capacitor is labeled with its **value** on the silkscreen.
- The remaining few components are labeled with their **reference designator**.

### Keycap note

Because of the thumb row geometry, some classic uniform profiles (like **XDA**) can catch on the top part of the case. [**KLP Lame MX** keycaps](https://github.com/braindefender/KLP-Lame-Keycaps) have been tested and work without any problems.

## Build Guide

Step-by-step assembly instructions live in a separate document: **[docs/build-guide/build-guide.md](docs/build-guide/build-guide.md)**.

## Firmware

The firmware is QMK with a custom Hall Effect matrix scanning routine (DMA-based ADC sampling, oversampling, Rapid Trigger, per-key calibration):

**https://github.com/RickKross/qmk-config-hector**

## Status & Roadmap

- ✅ rev1 hardware — working prototype, hand-soldered
- 🔧 Fine-tuning Rapid Trigger parameters
- 🔜 VIA support — VIA v3 Custom UI looks like a perfect fit for exposing magnetic settings (actuation point, Rapid Trigger sensitivity, etc.). Vial is not planned.
- 🔜 Minor hardware improvements for the next revision (a switch to STM32F303RC is likely)

## Credits & Inspirations

- [Totem](https://github.com/GEIGEIGEIST/TOTEM) by GEIGEIGEIST — layout inspiration
- [TOTEM MX](https://github.com/azhizhinov/TOTEMX) by azhizhinov — MX variant that started this journey
- [KLP Lame Keycaps](https://github.com/braindefender/KLP-Lame-Keycaps) by braindefender — tested and recommended keycaps

## License

Hardware design is released under the [LICENSE](LICENSE) in this repository. Firmware licensing is defined in its own repository.
