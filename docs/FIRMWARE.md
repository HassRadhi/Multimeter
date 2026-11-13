# Firmware needed

What the meter must do in software, broken into small slices. Implement **one phase at a time**; tick boxes as we go.

Style: `docs/CODE_STYLE.md`. Board: `hardware/v15_1`. MCU: STM32F373CCT6 @ ~3.0 V.

**Lab only (±60 V / ±2.5 A). Not CAT-rated — no mains.**

Upstream MartinD firmware is **pin/ADC reference only**, not something we port wholesale:
https://github.com/MartinD-CZ/STM32F1-open-source-multimeter/tree/master/firmware-multimeter

---

## Pin map (v15_1)

| Net / role | MCU |
|------------|-----|
| `LCD_RST` / `LCD_DC` / `LCD_CLK` / `LCD_DIN` | PA15 / PB4 / PB3 / PB5 (SPI1; CS tied to DGND) |
| Buttons | PA11, PC13, PC14, PC15 |
| Buzzer | PA8 |
| LED R / G | PB6 / PB7 |
| `VSEL0` / `VSEL1` | PF6 / PF7 |
| `ISEL` | PB2 |
| `OFSEL` | PA12 |
| EEPROM `SCL` / `SDA` | PB8 / PB9 (I2C1) |
| SDADC V/I analogue | PE8/9, PB0/1 (exact channels: check upstream + sheet 2) |
| SWD | SWDIO / SWCLK |

---

## Phase 0 — Project skeleton

- [ ] `firmware/` tree per `CODE_STYLE.md` (`board`, `drivers`, `display`, `measure`, `ui`, `util`)
- [ ] **Toolchain: VS Code + official ST STM32 extension** (CubeMX + build + SWD flash/debug in one place). Not PlatformIO, not CubeIDE as the daily driver.
- [ ] Follow **Setup (VS Code + ST extension)** below
- [ ] Blink or GPIO sanity (LED or buzzer) on real silicon when board arrives
- [ ] `board/pins.h` with the table above

**Done when:** project builds and flashes; one pin wiggles.

### Setup (VS Code + ST extension)

Official pack name: **STM32CubeIDE for Visual Studio Code** (STMicroelectronics). Use **VS Code**, not Cursor, for build/flash/debug (ST’s extension targets VS Code).

1. Install [VS Code](https://code.visualstudio.com/download).
2. Extensions → search **STM32** → install **STM32CubeIDE for Visual Studio Code** (extension pack; pulls CMake, clangd, debug adapters).
3. Open the STM32 sidebar icon → **STM32Cube Resources** → install **ST-Link USB Drivers** (needs admin on Windows).
4. Install classic **[STM32CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html)** (HAL1 / F3 — not CubeMX2-only flows). Default Windows path is typically under `C:\Program Files\STMicroelectronics\…\STM32CubeMX`.
5. Hardware: **ST-LINK/V2** (or clone) on the board **SWD** header (`SWDIO`, `SWCLK`, `GND`, and usually `3V`/`NRST` as your probe expects).
6. First project (when we scaffold `firmware/`):
   - STM32 sidebar → **Launch STM32CubeMX** (or open CubeMX)
   - MCU: **STM32F373CCT6**
   - Project Manager → Toolchain/IDE: **CMake** → Generate Code into `firmware/`
   - VS Code → **File → Open Folder** on that project
   - When prompted, **configure as STM32Cube project**
   - Status bar **Build**; Run and Debug → **STM32Cube: ST-Link GDB Server** (builds, flashes, breaks at `main`)

Toolchain (compiler, CMake, Ninja, GDB server) is pulled by the extension’s **bundle manager** on first build — no separate CubeCLT required for current extension versions.

ST docs: [installation](https://dev.st.com/stm32cube-docs/stm32cubeide-vscode/latest/en/docs/markup/getting_started/installation.html), [UM3512 PDF](https://www.st.com/resource/en/user_manual/um3512-stm32cube-for-visual-studio-code-installation-guide-stmicroelectronics.pdf).

---

## Phase 1 — Display bring-up (first real feature)

- [ ] SPI1 init (PB3 SCK, PB5 MOSI)
- [ ] GPIO: `LCD_DC`, `LCD_RST` (CS hard-wired low)
- [ ] SSD1306 init + clear + “hello” / splash
- [ ] Minimal font + draw string / number
- [ ] Optional: contrast / invert

**Done when:** OLED shows a known pattern without measuring anything.

---

## Phase 2 — Inputs and feedback

- [ ] Debounced buttons (4×)
- [ ] Simple UI state: mode select or menu stub on OLED
- [ ] Buzzer beep helper
- [ ] RGB LED status (at least R/G as in hardware)

**Done when:** button changes what’s on screen; beep/LED confirm input.

---

## Phase 3 — Core measurement path

Voltage / current AFE stays as on the PCB; firmware drives muxes and reads SDADC.

- [ ] SDADC init (V + I paths); DMA or polled — start simple
- [ ] Range GPIOs: `VSEL0`/`VSEL1`, `ISEL`, `OFSEL` (+ settle delay)
- [ ] Convert counts → mV / mA (use VREF 1.8 V, document assumptions)
- [ ] DC average display on OLED
- [ ] Over-range indication (UI only; no false safety claims)
- [ ] Battery / `ADC_VBAT` read (if we care early)

**Done when:** known lab source reads roughly correct on at least one V and one I range.

---

## Phase 4 — Meter modes (product behaviour)

- [ ] Voltage mode (multi-range auto or manual)
- [ ] Current mode
- [ ] Power (V×I) when both valid
- [ ] Continuity (threshold + buzzer)
- [ ] Optional: RMS path (after DC is trusted)
- [ ] Optional: frequency (TIM2 path, later)
- [ ] Optional: component test (TP* GPIOs — defer until basics work)

**Done when:** everyday DMM use works for V, I, continuity.

---

## Phase 5 — UI polish (differentiation)

Matches the product priority list: cleaner UI before fancy extras.

- [ ] Clear layout: big reading, unit, range
- [ ] Hold
- [ ] Min / max
- [ ] Relative (zero / delta)
- [ ] Splash / mode titles (Australian English copy)

**Done when:** hold / min-max / relative feel usable day to day.

---

## Phase 6 — Persistence

- [ ] AT24C02 via I2C1
- [ ] Save calibration / last mode / user prefs
- [ ] Sensible defaults if EEPROM empty or corrupt

**Done when:** mode (and later cal) survives power cycle.

---

## Phase 7 — USB logging (optional isolator board)

- [ ] USB CDC or UART-over-isolator path (match hardware you build)
- [ ] Stream timestamped V/I/P readings
- [ ] Document host-side capture (serial terminal is enough at first)

**Done when:** PC can log a session without blocking the UI badly.

---

## Explicitly later / out of scope for now

- TFT instead of OLED
- Full AFE redesign
- Enclosure firmware (none)
- Claiming CAT / mains safety in UI or docs
- RTOS unless we hit a real need

---

## Suggested order of work (bit by bit)

| Next slice | Why |
|------------|-----|
| **Phase 0** | Need a build before any feature |
| **Phase 1** | OLED is the main differentiator and the debug surface |
| **Phase 2** | Need input before modes |
| **Phase 3** | Real meter |
| **Phase 4 → 5** | Modes, then hold/minmax/relative |
| **Phase 6 → 7** | Nice-to-have once measuring |

When the PCB arrives, start at **Phase 0 → 1** unless you want a host-only stub first (fonts/layout without hardware).

---

## Status

| Phase | Status |
|-------|--------|
| 0 Skeleton | Not started |
| 1 Display | Not started |
| 2 Inputs | Not started |
| 3 Measure | Not started |
| 4 Modes | Not started |
| 5 UI polish | Not started |
| 6 EEPROM | Not started |
| 7 USB log | Not started |
