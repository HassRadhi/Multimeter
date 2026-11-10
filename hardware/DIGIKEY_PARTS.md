# DigiKey shopping list — STM32 OSM v1.5

Suggested MPNs for one board. DigiKey stock/pricing changes; open each link and confirm package matches the Eagle footprint before ordering.

**Search tip:** if a direct link fails, use  
`https://www.digikey.com/en/products/result?keywords=MPN`

## Critical ICs (order first)

| Ref | Suggested MPN | DigiKey search |
|-----|---------------|----------------|
| U1 | STM32F373CCT6 | https://www.digikey.com/en/products/result?keywords=STM32F373CCT6 |
| IC1 | MCP1700-3002E/TT (3.0 V LDO, SOT-23) | https://www.digikey.com/en/products/result?keywords=MCP1700-3002E%2FTT |
| IC2–IC4, IC7–IC8 (×5) | SN74LVC1G3157DBVR | https://www.digikey.com/en/products/detail/texas-instruments/SN74LVC1G3157DBVR/562896 |
| IC5 | INA199A1DCKR *(gain 50 V/V — confirm vs firmware)* | https://www.digikey.com/en/products/result?keywords=INA199A1DCKR |
| IC6 | MCP1501-18E/SN (1.8 V ref, SOIC-8) | https://www.digikey.com/en/products/result?keywords=MCP1501-18E%2FSN |
| IC9 | MCP6072-E/SN (dual op-amp SOIC-8) | https://www.digikey.com/en/products/result?keywords=MCP6072-E%2FSN |
| U2 | AT24C02D-SSHM-T or 24LC02B-I/SN | https://www.digikey.com/en/products/result?keywords=24LC02B-I%2FSN |
| Q2 | MMBT3904LT1G | https://www.digikey.com/en/products/result?keywords=MMBT3904LT1G |

## Passives & discretes

| Ref / qty | Value | DigiKey search (1206 unless noted) |
|-----------|-------|-------------------------------------|
| C× many | 100 nF 50 V X7R 1206 | https://www.digikey.com/en/products/result?keywords=1206+100nF+X7R |
| C1,C9,C11 | 22 µF 16 V 1206 | https://www.digikey.com/en/products/result?keywords=1206+22uF+16V |
| C4,C6,C10 | 1 µF 16 V 1206 | https://www.digikey.com/en/products/result?keywords=1206+1uF+X7R |
| C7,C8 | 10 nF 50 V 1206 | https://www.digikey.com/en/products/result?keywords=1206+10nF |
| C12,C13 | 22 pF C0G 1206 | https://www.digikey.com/en/products/result?keywords=1206+22pF+C0G |
| C17,C18,C20 | **TBD on schematic** — start with 100 nF C0G/X7R 1206 until you tune | same 100 nF link |
| R1,R2 | 2.2 kΩ 1% 1206 | https://www.digikey.com/en/products/result?keywords=1206+2.2k+1%25 |
| R3–R5 | 10 kΩ 1% 1206 | https://www.digikey.com/en/products/result?keywords=1206+10k+1%25 |
| R6,R7,R19,R20 | 1 kΩ 1% 1206 | https://www.digikey.com/en/products/result?keywords=1206+1k+1%25 |
| R8 | 100 Ω 1% 1206 | https://www.digikey.com/en/products/result?keywords=1206+100+ohm+1%25 |
| R9 | **5 mΩ** shunt (0R005), ≥1 W class | https://www.digikey.com/en/products/result?keywords=5+mOhm+shunt+1206 |
| R10 | **50 mΩ** shunt (0R050) | https://www.digikey.com/en/products/result?keywords=50+mOhm+shunt+1206 |
| R11 | 150 kΩ 0.1%/1% 1206 | https://www.digikey.com/en/products/result?keywords=1206+150k+0.1%25 |
| R12 | 1 MΩ 0.1%/1% 1206 | https://www.digikey.com/en/products/result?keywords=1206+1M+0.1%25 |
| R13 | 15 kΩ 0.1%/1% 1206 | https://www.digikey.com/en/products/result?keywords=1206+15k+0.1%25 |
| R14 | **DNP** — do not buy | — |
| R21,R22 | 220 Ω | https://www.digikey.com/en/products/result?keywords=1206+220+ohm+1%25 |
| R23,R24 | 2.2 kΩ | (same as R1) |
| R25,R26 | 220 kΩ | https://www.digikey.com/en/products/result?keywords=1206+220k+1%25 |
| FB1 | Ferrite bead ~1.8 µH / 1206 | https://www.digikey.com/en/products/result?keywords=1206+ferrite+bead |
| D2–D4 | SOD-80 / MiniMELF switching diode (e.g. LL4148) | https://www.digikey.com/en/products/result?keywords=LL4148 |
| Q1 | 8 MHz HC-49/U crystal | https://www.digikey.com/en/products/result?keywords=8MHz+HC-49 |
| F1 | 2.5 A fast fuse (match SHK20L footprint / holder) | https://www.digikey.com/en/products/result?keywords=2.5A+fast+fuse+5x20 |

## Connectors / UI / mech

| Ref | What | DigiKey search |
|-----|------|----------------|
| D1 | Nokia 5110 / PCD8544 LCD module | **not DigiKey** — AliExpress / Adafruit-style “Nokia 5110” |
| P1–P4 | PCB banana jack 4 mm | https://www.digikey.com/en/products/result?keywords=banana+jack+PCB+4mm |
| S1 | 6 mm tactile PTH | https://www.digikey.com/en/products/result?keywords=tactile+switch+6mm+through+hole |
| S2–S5 | 12 mm SMD tactile | https://www.digikey.com/en/products/result?keywords=tactile+switch+12mm+SMD |
| SW1 | small DPDT slide | https://www.digikey.com/en/products/result?keywords=DPDT+slide+switch |
| LED1 | 5 mm RGB common-anode | https://www.digikey.com/en/products/result?keywords=RGB+LED+5mm+common+anode |
| SG1 | magnetic buzzer ~3 V | https://www.digikey.com/en/products/result?keywords=magnetic+buzzer+3V |
| EXP / PWR / TP | 2.54 mm pin headers | https://www.digikey.com/en/products/result?keywords=pin+header+2.54mm |

## Also needed (not on main BOM)

- PCB fab from `v15.brd` (JLCPCB etc.) after Gerber export  
- Optional: `V15_USB_isolator` board + isolator parts  
- LiPo / battery holder, probes, 3D-printed case  
- ST-Link (or use SWD on EXP header) for programming  

## Before you click Buy

1. Confirm **INA199** gain (A1/A2/A3) against firmware.  
2. Confirm **MCP1700** is **3.0 V** version (`-3002`).  
3. Confirm **MCP1501** is **1.8 V** (`-18`).  
4. Fill **C17/C18/C20** (author left TBD).  
5. Skip **R14**.  
6. Match shunt packages to the Eagle footprints (power rating!).  
