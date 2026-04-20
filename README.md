# InkTime — Smartwatch

InkTime is an open-source, minimalist smartwatch built around an always-readable 1.54" e-paper display and optimized for month-class battery life (≥30 days on a 250 mAh LiPo). It features BLE 5.0 notifications, embedded step counting, haptic alerts, and USB-C charging.

---

## Block Diagram

```
                          ┌──────────────────────────────────────────┐
         USB-C            │           nRF52840 (U1)                  │
        ──────►──┐        │                                          │
                 │  VBUS  │  ┌─────────────────────────────────┐     │
                 ▼        │  │         I2C Bus                 │     │
          ┌──────────┐    │  │  SDA P0.06 / SCL P0.07          │     │
          │ BQ25180  │◄───┼──┤  Pull-up 3K3 to 3V3             │     │
          │  (IC1)   │    │  │                                 │     │
          │ Charger  │    │  │  BQ25180 (0x6A)  RT6160 (0x75)  │     │
          └────┬─────┘    │  │  MAX17048 (0x36) BMA421 (0x18)  │     │
               │ SYS      │  │  DRV2605L (0x5A)                │     │
          ┌────▼─────┐    │  └─────────────────────────────────┘     │
          │ RT6160   │◄───┤                                          │
          │  (IC9)   │    │  ┌─────────────────────────────────┐     │
          │Buck-Boost│    │  │        SPI Bus                  │     │
          └────┬─────┘    │  │  SCK P0.02 / MOSI P0.03         │     │
               │ 3V3      │  │  CS P0.05 / DC P0.15            │     │
               ▼          │  │  RST P0.16 / BUSY P0.17         │     │
    ┌──────────────────┐  │  │  PFET gate P1.01                │     │
    │   3V3 Power Rail │  │  │                                 │     │
    └─┬───┬───┬───┬────┘  │  │  E-Paper 1.54" 200x200 (J1)     │     │
      │   │   │   │       │  │  via SI1308EDL MOSFET (Q3)      │     │
      │   │   │   │       │  └─────────────────────────────────┘     │
      │   │   │   │       │                                          │
      │   │   │   │       │  GPIO                                    │
      │   │   │   └───────┼──► SW_UP P0.13 / SW_DN P0.14             │
      │   │   │           │    SW_ENT P1.00                          │
      │   │   │           │  INT: BMA421 INT1→P0.08 INT2→P1.08       │
      │   │   │           │       MAX17048 ALRT→P0.10                │
      │   │   │           │       BQ25180 /INT→P0.11                 │
      │   │   │           │       HAPTIC_EN→P0.12                    │
      │   │   │           │                                          │
      │   │   │           │  RF: ANT→L1(3.9nH)→L3(15nH)→ANT1         │
      │   │   │           │  USB: D+/D-→USBLC6(D3)→USB-C (J4)        │
      │   │   │           │  SWD: SWDIO/SWDCLK/RESET→TC2030 (J2)     │
      │   │   │           └──────────────────────────────────────────┘
      │   │   │
      │   │  ┌▼────────────┐    ┌──────────────┐
      │   │  │  DRV2605L   ├───►│  ERM Motor   │
      │   │  │  (IC2) I2C  │    │  (Haptic)    │
      │   │  └─────────────┘    └──────────────┘
      │   │
      │  ┌▼────────────┐
      │  │   BMA421    │
      │  │   (IC3) I2C │  INT1→P0.08 / INT2→P1.08
      │  └─────────────┘
      │
     ┌▼────────────┐
     │  MAX17048   │
     │  (U3) I2C   │  ALRT→P0.10
     └─────────────┘

  ┌──────────┐
  │  LiPo    │──VBAT──► BQ25180 (BAT) ──► MAX17048 (BAT/VDD)
  │ 250 mAh  │
  └──────────┘
```

---

## Bill of Materials (BOM)

| Qty | Ref | Component | Package | Description | JLCPCB | Datasheet |
|-----|-----|-----------|---------|-------------|--------|-----------|
| 1 | U1 | nRF52840-QIAA-R | aQFN73 | MCU BLE 5.0 + USB 2.0 | — | [Link](https://infocenter.nordicsemi.com/pdf/nRF52840_PS_v1.7.pdf) |
| 1 | IC1 | BQ25180YBGR | DSBGA-8 | LiPo charger / power path | [C3682423](https://jlcpcb.com/partdetail/TexasInstruments-BQ25180YBGR/C3682423) | [Link](https://www.ti.com/lit/ds/symlink/bq25180.pdf) |
| 1 | IC9 | RT6160AWSC | WLCSP-15 | Buck-boost 3.3V regulator | [C7065276](https://jlcpcb.com/partdetail/Richtek-RT6160AWSC/C7065276) | [Link](https://www.richtek.com/assets/product_file/RT6160A/DS6160A-00.pdf) |
| 1 | U3 | MAX17048G+T10 | DFN-8 | Fuel gauge I2C | — | [Link](https://datasheets.maximintegrated.com/en/ds/MAX17048-MAX17049.pdf) |
| 1 | IC3 | BMA421 | LGA-12 | Accelerometer + step counter | — | [Link](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bma421-ds000.pdf) |
| 1 | IC2 | DRV2605YZFR | DSBGA-9 | Haptic driver ERM/LRA | — | [Link](https://www.ti.com/lit/ds/symlink/drv2605l.pdf) |
| 1 | Q3 | SI1308EDL-T1-GE3 | SC-70 | N-ch MOSFET EPD power gate | — | [Link](https://www.vishay.com/docs/63587/si1308edl.pdf) |
| 1 | DMG2305UX-7 | DMG2305UX-7 | SOT-23 | P-ch MOSFET EPD drive | — | — |
| 1 | ANT1 | 2450AT18B100E | SMD 3216 | 2.45 GHz chip antenna | — | [Link](https://www.johansontechnology.com/datasheets/2450AT18B100E/2450AT18B100E.pdf) |
| 1 | J4 | KH-TYPE-C-16P | THT | USB-C 16-pin connector | — | — |
| 1 | D3 | USBLC6-2SC6Y | SOT-23-6 | USB ESD protection | — | [Link](https://www.st.com/resource/en/datasheet/usblc6-2.pdf) |
| 3 | D2,D4,D5 | MBR0530 | SOD-123 | Schottky diode 30V 500mA | — | — |
| 1 | J2 | TC2030-IDC | PCB footprint | Tag-Connect SWD debug | — | [Link](https://www.tag-connect.com/wp-content/uploads/bsk-pdf-manager/TC2030-IDC_1.pdf) |
| 1 | J1 | 503480-2400 | FPC 24-pin | E-paper FPC connector 0.5mm | — | [Mouser](https://www.mouser.co.uk/ProductDetail/Molex/503480-2400) |
| 3 | SW_UP,SW_DN,SW_ENT | EVP-AKE31A | SMD | Tactile button Panasonic | — | — |
| 1 | X1 | 32 MHz crystal | 2016 SMD | HFXO nRF52840 | — | — |
| 1 | X2 | 32.768 kHz crystal | 3215 SMD | LFXO RTC nRF52840 | — | — |
| 1 | L7 | FTC252012SR47MBCA | 2520 | 0.47 µH RT6160 switching | [C5832368](https://jlcpcb.com/partdetail/6763488-FTC252012SR47MBCA/C5832368) | — |
| 1 | L2 | 10 µH | 0402 | RT6160 power stage | — | — |
| 1 | L3 | 15 nH | 0402 | RF matching network | — | — |
| 1 | L1 | 3.9 nH | 0402 | RF matching network | — | — |
| 1 | L5 | 68 µH | 0402 | EPD boost converter | — | — |
| 5 | C5,C7,C8,C12,C19 | 100 nF | 0201 | Decoupling nRF52840 | — | — |
| 4 | C1,C2,C17,C18 | 12 pF | 0201 | Crystal load caps | — | — |
| 6 | C29–C32,C37,C38 | 1 µF | 0201 | Bulk decoupling | — | — |
| 3 | C23,C27,C42 | 0.1 µF | 0201 | Decoupling | — | — |
| 2 | C25,C33 | 22 µF | 0402 | RT6160 output caps | — | — |
| 3 | C14,C20,C21 | 4.7 µF | 0402 | Bulk decoupling | — | — |
| 1 | C43 | 4.7 µF | 0402 | DECUSB nRF52840 | — | — |
| 1 | C24 | 10 µF | 0402 | RT6160 input cap | — | — |
| 9 | EPD_C1..C12 | 1 µF/50V | 0402 | EPD supply caps | — | — |
| 1 | C15 | 1.0 µF | 0201 | DEC4/6 nRF52840 | — | — |
| 1 | C11 | 100 pF | 0201 | RF matching | — | — |
| 2 | C3,C4 | 1 pF | 0201 | RF matching | — | — |
| 1 | C9 | 820 pF | 0201 | RF decoupling | — | — |
| 2 | R17,R18 | 3.3 kΩ | 0201 | I2C pull-ups SDA/SCL | — | — |
| 3 | R5,R7,R8 | 10 kΩ | 0201 | Button pull-ups | — | — |
| 2 | R1_USB,R2_USB | 5.1 kΩ | 0201 | USB CC1/CC2 resistors | — | — |
| 14 | TP_* | Test pad | TP20R | Test points | — | — |
| 1 | SJ1 | Solder jumper | SMD | Config jumper | — | — |

---

## Hardware Functionality Description

### Power Architecture

```
USB 5V → BQ25180 (IN) → BQ25180 (SYS) → RT6160 (VIN) → 3.3V rail
LiPo   → BQ25180 (BAT) and LiPo → MAX17048 (BAT/VDD)
```

**BQ25180 (IC1)** handles LiPo charging and power-path management. When USB is present, it powers the system from USB and charges the battery simultaneously without brownout on connect/disconnect. TS/MR strapped to GND (R9, 10kΩ) for Rev A. Decoupling: C37 (1µF on IN), C39 (10µF on SYS), C38 (1µF on BAT).

**RT6160 (IC9)** is a buck-boost maintaining stable 3.3V across the full LiPo discharge range (4.2V → 3.0V). VSEL tied low for 3.3V output. EN tied to VSYS (always on when SYS present). Switching inductor L7 (0.47µH) placed close to SW pins. Output caps C25 + C33 (22µF each).

**MAX17048 (U3)** reports battery state-of-charge over I2C (0x36) using the ModelGauge algorithm, requiring no sense resistor. ALRT output (active-low) on P0.10 wakes MCU when battery drops below threshold.

### MCU — nRF52840 (U1)

ARM Cortex-M4F, 64 MHz, BLE 5.0, USB 2.0 Full Speed. Normal-voltage mode (VDD = 3.3V). Internal DC/DC regulator (REG1) enabled in firmware to reduce active-mode current.

Decoupling placed directly at MCU pins: DEC1 → C7 (100nF), DEC3 → C8 (100nF), DEC4/DEC6 → C15 (1µF), DECUSB → C43 (4.7µF). DC/DC network: DCC → L2 (10µH) → L3 (15nH) → DEC4/6 node.

### I2C Bus

SDA P0.06 / SCL P0.07, pull-ups R17+R18 (3.3kΩ to 3V3), 100 kHz default.

| Device | Address | Function |
|--------|---------|---------|
| BMA421 (IC3) | 0x18 | Accelerometer + step counter |
| MAX17048 (U3) | 0x36 | Fuel gauge |
| DRV2605L (IC2) | 0x5A | Haptic driver |
| BQ25180 (IC1) | 0x6A | Charger / power path |
| RT6160 (IC9) | 0x75 | Buck-boost regulator |

### E-Paper Display (J1)

1.54" 200×200 e-paper via Molex 503480-2400 24-pin FPC connector (0.5mm pitch). SPI communication. VEPD switched by SI1308EDL N-ch MOSFET (Q3, gate P1.01) for complete power gating between updates. EPD boost circuit: D2, D4, D5 (MBR0530) + L5 (68µH) + DMG2305UX-7 generates PREVGL/PREVGH panel voltages. Optional DNP series resistors on SPI lines prevent back-powering when VEPD off.

### Accelerometer — BMA421 (IC3)

Embedded step counter running autonomously. I2C 0x18 (CSB high, SDO low). INT1 → P0.08 for motion/step wake; INT2 → P1.08 for secondary events. Placed away from ERM motor to avoid false motion coupling.

### Haptics — DRV2605L (IC2)

Drives ERM motor via I2C (0x5A). EN pin P0.12 shuts down driver completely between events. Built-in waveform library selectable by effect ID over I2C.

### RF — ANT1

Matching network: ANT → L1 (3.9nH) → L3 (15nH) → C3/C4 (1pF to GND) → 2450AT18B100E chip antenna. Antenna at PCB edge with physical cutout and GND keepout underneath.

### USB-C (J4)

USB 2.0 FS. D+/D- differential pair with matched lengths. USBLC6-2SC6Y (D3) ESD protection at connector entry. R1_USB + R2_USB (5.1kΩ each) on CC1/CC2 for USB-C sink detection.

### Energy Budget

| Condition | Current | Fraction | Avg µA |
|-----------|---------|----------|--------|
| Deep sleep | ~3 µA | 98.8% | ~3.0 |
| EPD partial refresh (2s/min) | ~8 mA | 0.13% | ~10.4 |
| BLE notification | ~15 mA | 0.05% | ~7.5 |
| Haptic event | ~50 mA | 0.02% | ~10.0 |
| **Total** | | | **~31 µA** |

250 mAh / 0.031 mA ≈ **~33 days** typical use.

---

## nRF52840 Pin Assignment

| Pin | Net | Dir | Connected to | Rationale |
|-----|-----|-----|-------------|-----------|
| P0.02 | SCK | OUT | EPD display | SPI — high-speed Port 0 peripheral |
| P0.03 | MOSI | OUT | EPD display | SPI data out |
| P0.05 | EPD_CS | OUT | EPD display | SPI chip select |
| P0.06 | SDA | I/O | All I2C ICs | Nordic reference I2C pin |
| P0.07 | SCL | OUT | All I2C ICs | Nordic reference I2C pin |
| P0.08 | IMU_INT1 | IN | BMA421 INT1 | GPIO sense, wake from deep sleep |
| P0.10 | ALERT | IN | MAX17048 ALRT | Low battery wake |
| P0.11 | PMIC_INT | IN | BQ25180 /INT | Charger event interrupt |
| P0.12 | HAPTIC_EN | OUT | DRV2605L EN | Haptic driver shutdown between events |
| P0.13 | SW_UP | IN | Button UP | GPIO sense + internal pull-up, wake capable |
| P0.14 | SW_DN | IN | Button DOWN | GPIO sense + internal pull-up, wake capable |
| P0.15 | EPD_DC | OUT | EPD display | Data/command select |
| P0.16 | EPD_RST | OUT | EPD display | Panel reset |
| P0.17 | EPD_BUSY | IN | EPD display | Busy status |
| P0.18/RESET | nRESET | IN | TC2030 pin 6 | Dedicated hardware reset |
| P0.00/XL1 | LFXO | — | X2 crystal | 32.768 kHz RTC |
| P0.01/XL2 | LFXO | — | X2 crystal | 32.768 kHz RTC |
| XC1/XC2 | HFXO | — | X1 crystal | 32 MHz RF/CPU |
| ANT | RF | — | L1→L3→ANT1 | Antenna matching network |
| VBUS | Detect | IN | USB-C VBUS | USB attach detection |
| D+/D- | USB | I/O | USBLC6→J4 | USB 2.0 FS differential pair |
| SWDIO | SWD | I/O | TC2030 pin 2 | Programming + debug |
| SWDCLK | SWD | IN | TC2030 pin 4 | Programming + debug |
| P1.00 | SW_ENT | IN | Button ENTER | GPIO sense + pull-up, wake capable |
| P1.01 | EPD_PWR | OUT | Q3 gate | EPD power gate control |
| P1.08 | IMU_INT2 | IN | BMA421 INT2 | Secondary accel events |
| DCC | DC/DC | — | L2 (10µH) | Internal regulator inductor |
| DEC1 | Decoupling | — | C7 (100nF) | Core decoupling |
| DEC3 | Decoupling | — | C8 (100nF) | Core decoupling |
| DEC4/DEC6 | Decoupling | — | C15 (1µF) | DC/DC output node |
| DECUSB | Decoupling | — | C43 (4.7µF) | USB PHY decoupling |

---

## PCB Design Notes

**Stackup:** 2-layer, FR4, 1.6mm. Top + Bottom copper (1oz), green soldermask, white silkscreen.

**Placement:** nRF52840 centered; RF matching + ANT1 at top-right board edge with physical PCB cutout and GND keepout under antenna; BQ25180 + RT6160 grouped away from antenna; BMA421 separated from ERM motor; all 100nF decoupling within 0.5mm of VDD pins; TC2030 at board edge.

**Trace widths:** Power (3V3, VBAT) = 0.3mm; signals (SPI, I2C, GPIO) ≥ 0.15mm; USB D+/D- = 0.15mm matched length. No 90° angles.

**GND:** Polygon on Top + Bottom layers, via stitching throughout, especially dense around RF section.

---

## Repository Structure

```
├── Hardware/
│   ├── schemTSC.sch
│   └── schemaPCB.brd
├── Manufacturing/
│   ├── gerbers.zip
│   ├── bom.csv
│   └── pick-n-place.csv
├── Mechanical/
│   ├── casedPCB.step
│   └── exploded_case.step
├── Images/
│   ├── exploded_render.png
│   ├── PCB_render.png
│   ├── PCB_top.png
│   └── PCB_bottom.png
├── LICENSE
└── README.md
```