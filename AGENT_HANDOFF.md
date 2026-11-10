# Agent handoff — DIY Multimeter

Give this file (or paste it) to a new Cursor agent on the desktop.

## Project
- Repo: https://github.com/HassRadhi/Multimeter
- Based on MartinD STM32 OSM v1.5 (Eagle → KiCad import)
- Lab instrument only: **±60 V / ±2.5 A**, **not CAT-rated**, no mains use
- MCU: STM32F373CCT6 @ ~3.0 V (MCP1700)
- Goal: keep hardware AFE mostly intact; **rewrite firmware from scratch** with light inspiration from upstream only

## Repo layout
- `hardware/v15.sch|.brd|.lbr` — original Eagle
- `hardware/v15/` — KiCad project (`v15.kicad_pro`, sheets `v15.kicad_sch` + `v15_2.kicad_sch`)
- `hardware/DIGIKEY_PARTS.md` — DigiKey shopping notes
- `hardware/v15_BOM.txt` — upstream BOM (C17/C18/C20 TBD, R14 DNP)

## Differentiation plan (priority order)
1. **OLED/TFT + cleaner UI** ← in progress (OLED first)
2. Relative / min-max / hold (firmware)
3. USB logging (firmware + optional isolator board)
4. Finished enclosure (mechanical)

## Display change (done / in progress on desktop KiCad)
- Replacing Nokia **5110** with **SSD1306 128×64 SPI** via 7-pin header
- Nets to keep: `3V`, `DGND`, `LCD_CLK`, `LCD_DIN`, `LCD_DC`, `LCD_RST`
- **CS → DGND** (same idea as old CE tied low)
- No `LIGHT` pin
- Preferred header order (match common modules):  
  `1 GND, 2 VCC, 3 D0/CLK, 4 D1/MOSI, 5 RES, 6 DC, 7 CS→DGND`
- Footprint: `Connector_PinHeader_2.54mm:PinHeader_1x07_P2.54mm_Vertical`
- Use **DGND** power symbols only (not a `GND` label) to avoid ERC noise
- Eagle imports produce lots of **ERC warnings** — ignore most; care about shorts / OLED/`LCD_*` nets

## Schematic map (for the agent)
- **Sheet 1:** power, STM32, display, buttons, EEPROM, buzzer, EXP/SWD
- **Sheet 2:** voltage attenuator + muxes, current shunts + INA199, VREF, component-test

## Firmware intent
- From-scratch code on STM32F373
- Little reuse of MartinD firmware (inspiration only)
- Upstream firmware exists at:  
  https://github.com/MartinD-CZ/STM32F1-open-source-multimeter/tree/master/firmware-multimeter  
  (needed later for pin/ADC behaviour reference, not to copy wholesale)

## What to do next
1. Confirm OLED header wiring + footprint in KiCad; run ERC; triage real errors only
2. Commit/push schematic changes to GitHub
3. Then: SSD1306 bring-up plan (pin map from MCU nets → SPI driver skeleton)
4. Defer TFT, full AFE redesign, and enclosure CAD until display + basic UI work

## Constraints / style
- Prefer smallest change that works
- Australian English in user-facing copy
- Don’t invent CAT ratings or claim mains safety
