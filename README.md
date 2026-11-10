# Multimeter

DIY STM32F373 digital multimeter based on [MartinD OSM v1.5](https://github.com/MartinD-CZ/STM32F1-open-source-multimeter) — KiCad import, BOM, and firmware notes.

**Lab use only (±60 V / ±2.5 A). Not CAT-rated — do not use on mains.**

## Hardware

| Path | What |
|------|------|
| `hardware/v15.sch` / `.brd` / `.lbr` | Original Eagle 9.3 design |
| `hardware/V15_USB_isolator.*` | Optional USB isolator (Eagle) |
| `hardware/v15/` | KiCad import of the main meter (open `v15.kicad_pro`) |
| `hardware/schematics_sheet*.png` | Schematic previews |
| `hardware/v15_BOM.txt` | Upstream BOM |
| `hardware/DIGIKEY_PARTS.md` | Suggested DigiKey part links |
| `hardware/IMPORT_TO_KICAD.txt` | Eagle → KiCad import notes |

Upstream license terms apply to original files. See their repo `LICENSE` / file headers. Original additions in this repo are under the license in `LICENSE`.

## Next

1. Open `hardware/v15/v15.kicad_pro` in KiCad  
2. Run ERC / DRC  
3. Flash firmware from upstream `firmware-multimeter`  
