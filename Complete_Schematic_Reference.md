# PWM Solenoid Controller — Complete Schematic Reference
## Matches Finalized BOM Rev 1.1

---

## Master Net List

| Net Name | Description |
|----------|-------------|
| +24V_RAW | 24V from J1 before reverse polarity protection |
| +24V | 24V after D1 protection and F1 fuse |
| +3V3 | 3.3V regulated output from U2 |
| GND | Common ground (24V and 12V supplies) |
| +12V_EXT | 12V external supply input at J5 |
| GPIO0_FAULT | ESP32 GPIO0 → fault output MOSFET Q2 |
| GPIO1_PWM | ESP32 GPIO1 → PWM MOSFET Q1 gate |
| GPIO3_ADC | ESP32 GPIO3 → current sense voltage |
| GPIO4_DI1 | ESP32 GPIO4 ← optocoupler output (DI1) |
| GPIO5_DI2 | ESP32 GPIO5 ← optocoupler output (DI2) |
| GPIO6_DI3 | ESP32 GPIO6 ← optocoupler output (DI3) |
| GPIO7_DI4 | ESP32 GPIO7 ← optocoupler output (DI4) |
| GPIO8_DI5 | ESP32 GPIO8 ← optocoupler output (DI5) |
| GPIO9_DI6 | ESP32 GPIO9 ← optocoupler output (DI6) |
| GPIO10_DI7 | ESP32 GPIO10 ← optocoupler output (DI7) |
| GPIO11_DI8 | ESP32 GPIO11 ← optocoupler output (DI8) |
| SDA | ESP32 GPIO40 ↔ OLED I2C data |
| SCL | ESP32 GPIO41 → OLED I2C clock |
| USB_DP | ESP32 GPIO20 ↔ USB-C D+ |
| USB_DN | ESP32 GPIO19 ↔ USB-C D- |
| EN | ESP32 EN pin (active HIGH = run) |
| BOOT | ESP32 GPIO0 (LOW during reset = bootloader) |
| SOLENOID_SW | MOSFET Q1 drain → solenoid switched terminal |
| SENSE_V | MOSFET Q1 source → sense resistor top → GPIO3 |
| DTR | USB-C DTR signal for auto-reset |
| RTS | USB-C RTS signal for auto-boot |

---

## SECTION 1: POWER INPUT AND PROTECTION

### Components: J1, D1, F1, TVS1, C9

```
                    J1 (2-pin screw terminal, 5.08mm)
                    Pin 1: +24V input
                    Pin 2: GND

    J1-Pin1 (+24V) ────────────┬──────────── +24V_RAW
                               │
                          TVS1 (SMBJ26A)
                          Anode ── GND
                          Cathode ── +24V_RAW
                               │
                         D1 (SS54 Schottky)
                         Anode ── +24V_RAW
                         Cathode ──┬──────── After D1
                                   │
                              F1 (Polyfuse 1A/30V)
                              Pin 1 ── After D1
                              Pin 2 ──┬──────── +24V (protected rail)
                                      │
                                 C9 (100uF/35V electrolytic)
                                 + ── +24V
                                 - ── GND

    J1-Pin2 (GND) ─────────────────────────── GND
```

**Connection summary:**
- J1 Pin1 → TVS1 cathode → D1 anode
- D1 cathode → F1 Pin1
- F1 Pin2 = +24V (protected, fused rail)
- TVS1 anode → GND
- C9 between +24V and GND
- J1 Pin2 → GND

**Note:** D1 (SS54) drops approximately 0.5V, so the Traco module sees about 23.5V. This is well within its 4.75-36V input range. TVS1 clamps transients above 42V, protecting D1 and everything downstream.

---

## SECTION 2: VOLTAGE REGULATOR

### Components: U2, C1, C2

```
                         U2 (Traco TSR 1-2433)
                         ┌─────────────┐
                         │  VIN  GND  VOUT │
                         └──┬────┬────┬──┘
                            │    │    │
                   +24V ────┘    │    └──────┬────── +3V3
                                 │           │
                            GND ─┘      C2 (22uF/10V)
                                        + ── +3V3
                   C1 (22uF/35V)        - ── GND
                   + ── +24V
                   - ── GND
```

**Connection summary:**
- U2 Pin VIN ← +24V
- U2 Pin GND ← GND
- U2 Pin VOUT → +3V3
- C1 between +24V and GND (within 10mm of U2 VIN pin)
- C2 between +3V3 and GND (within 10mm of U2 VOUT pin)

**Traco TSR 1-2433 pinout (bottom view, pins facing you):**
- Pin 1 (left): VIN
- Pin 2 (center): GND
- Pin 3 (right): VOUT

---

## SECTION 3: ESP32-S3 MODULE

### Components: U1, C4-C7, C8, R24

```
    U1 (ESP32-S3-WROOM-1-N4R2)
    ┌────────────────────────────────────────────────┐
    │                                                │
    │  3V3 (Pin 2) ─────────┬──── +3V3               │
    │                       ├── C4 (100nF) ── GND    │
    │                       ├── C5 (100nF) ── GND    │
    │                       ├── C6 (100nF) ── GND    │
    │                       ├── C7 (100nF) ── GND    │
    │                       └── C8 (10uF)  ── GND    │
    │                                                │
    │  GND (Pins 1,40,41,EP) ────────── GND          │
    │                                                │
    │  EN (Pin 3) ──────────┬── R_EN (10k) ── +3V3   │
    │                       └── C_EN (100nF) ── GND  │
    │                       └── Q3 collector (auto-reset) │
    │                                                │
    │  GPIO0 (Pin 27) ─────┬── R24 (10k) ── +3V3     │
    │                      ├── C_BOOT (100nF) ── GND │
    │                      ├── Q4 collector (auto-boot) │
    │                      └── R22 (1k) ── Q2 gate (fault) │
    │                                                │
    │  GPIO1 (Pin 28) ──── R19 (100R) ── Q1 gate (PWM) │
    │                                                │
    │  GPIO3 (Pin 4) ───── GPIO3_ADC (current sense) │
    │                                                │
    │  GPIO4 (Pin 5) ───── GPIO4_DI1                 │
    │  GPIO5 (Pin 6) ───── GPIO5_DI2                 │
    │  GPIO6 (Pin 7) ───── GPIO6_DI3                 │
    │  GPIO7 (Pin 8) ───── GPIO7_DI4                 │
    │  GPIO8 (Pin 9) ───── GPIO8_DI5                 │
    │  GPIO9 (Pin 10) ──── GPIO9_DI6                 │
    │  GPIO10 (Pin 11) ─── GPIO10_DI7                │
    │  GPIO11 (Pin 12) ─── GPIO11_DI8                │
    │                                                │
    │  GPIO19 (Pin 20) ─── USB_DN (USB D-)           │
    │  GPIO20 (Pin 21) ─── USB_DP (USB D+)           │
    │                                                │
    │  GPIO40 (Pin 36) ─── SDA                       │
    │  GPIO41 (Pin 37) ─── SCL                       │
    │                                                │
    │  All unused GPIOs ── No Connect (X)            │
    │                                                │
    └────────────────────────────────────────────────┘
```

**Critical ESP32-S3 connections:**
- All GND pins AND the exposed pad (EP) must connect to GND
- EN pin needs 10k pull-up to +3V3 AND 100nF cap to GND for stable power-on reset
- GPIO0 needs 10k pull-up to +3V3 (R24) for normal boot AND 100nF cap to GND (C_BOOT)
- Place decoupling caps C4-C7 within 5mm of the module's power pins
- Leave NO copper pour under the antenna end of the module (last ~5mm)

**Pin number reference (ESP32-S3-WROOM-1, counter-clockwise from Pin 1):**

| Pin | GPIO | Function in this design |
|-----|------|------------------------|
| 1 | GND | Ground |
| 2 | 3V3 | Power input |
| 3 | EN | Enable (pull-up + auto-reset) |
| 4 | GPIO4 | DI1 input |
| 5 | GPIO5 | DI2 input |
| 6 | GPIO6 | DI3 input |
| 7 | GPIO7 | DI4 input |
| 8 | GPIO15 | NC |
| 9 | GPIO16 | NC |
| 10 | GPIO17 | NC |
| 11 | GPIO18 | NC |
| 12 | GPIO8 | DI5 input |
| 13 | GPIO19 | USB D- |
| 14 | GPIO20 | USB D+ |
| 15 | GPIO3 | ADC current sense |
| 16 | GPIO46 | NC |
| 17 | GPIO9 | DI6 input |
| 18 | GPIO10 | DI7 input |
| 19 | GPIO11 | DI8 input |
| 20 | GPIO12 | NC |
| 21 | GPIO13 | NC |
| 22 | GPIO14 | NC |
| 23 | GPIO21 | NC |
| 24 | GPIO47 | NC |
| 25 | GPIO48 | NC |
| 26 | GPIO45 | NC |
| 27 | GPIO0 | BOOT + Fault output |
| 28 | GPIO35 | NC |
| 29 | GPIO36 | NC |
| 30 | GPIO37 | NC |
| 31 | GPIO38 | NC |
| 32 | GPIO39 | NC |
| 33 | GPIO40 | I2C SDA |
| 34 | GPIO41 | I2C SCL |
| 35 | GPIO42 | NC |
| 36 | RXD0 | NC |
| 37 | TXD0 | NC |
| 38 | GPIO2 | NC |
| 39 | GPIO1 | PWM output |
| 40 | GND | Ground |
| 41 | GND | Ground |
| EP | GND | Exposed pad - Ground |

**IMPORTANT:** The pin-to-GPIO mapping above is approximate. You MUST verify against the actual ESP32-S3-WROOM-1 datasheet before finalizing the schematic. The KiCad symbol from the Espressif library will have the correct mapping.

---

## SECTION 4: DIGITAL INPUTS (8 CHANNELS)

### Components: U3, U4, R3-R10, R11-R18, D2-D9, TVS2-TVS9, J2

Each channel follows identical circuit. Shown here for Channel 1 (DI1), repeat for all 8.

```
    J2 Terminal DI1+ ──┬── TVS2 (SMBJ26A) cathode
                       │   TVS2 anode ── GND
                       │
                       └── R3 (10k) ──┬── U3-TLP281 LED Anode (Pin 1)
                                      │
    J2 Terminal DI- ───┬── D2 (1N4148W) cathode
                       │   D2 anode ──── U3-TLP281 LED Cathode (Pin 2)
                       │
                       └── GND (DI- commoned to GND on PCB)


    +3V3 ── R11 (10k) ──┬── U3-TLP281 Collector (Pin 8)
                         │
                         └── GPIO4_DI1 (net label → ESP32 GPIO4)

    GND ──────────────────── U3-TLP281 Emitter (Pin 7)
```

### TLP281-4 Pin Mapping (U3 — Channels 1-4)

```
    U3 (TLP281-4, DIP-16 or SOP-16)
    ┌──────────────────────┐
    │                      │
    │  Pin 1  (1A) ← LED Anode Ch1   Pin 16 (4C) → Collector Ch4 │
    │  Pin 2  (1K) ← LED Cathode Ch1 Pin 15 (4E) → Emitter Ch4   │
    │  Pin 3  (2A) ← LED Anode Ch2   Pin 14 (3C) → Collector Ch3 │
    │  Pin 4  (2K) ← LED Cathode Ch2 Pin 13 (3E) → Emitter Ch3   │
    │  Pin 5  (3A) ← LED Anode Ch3   Pin 12 (2C) → Collector Ch2 │
    │  Pin 6  (3K) ← LED Cathode Ch3 Pin 11 (2E) → Emitter Ch2   │
    │  Pin 7  (4A) ← LED Anode Ch4   Pin 10 (1C) → Collector Ch1 │
    │  Pin 8  (4K) ← LED Cathode Ch4 Pin 9  (1E) → Emitter Ch1   │
    │                      │
    └──────────────────────┘

    NOTE: Pin numbering varies by manufacturer. The above is for standard
    TLP281-4 pinout. VERIFY against your specific part's datasheet.
```

### Complete Channel Mapping Table

| Channel | J2 Terminal | TVS | Input R | Diode | Opto IC | Opto Pins (A,K,C,E) | Pull-up R | Net Label | ESP32 GPIO |
|---------|------------|-----|---------|-------|---------|---------------------|-----------|-----------|------------|
| DI1 | DI1+ | TVS2 | R3 | D2 | U3 Ch1 | 1,2,10,9 | R11 | GPIO4_DI1 | GPIO4 |
| DI2 | DI2+ | TVS3 | R4 | D3 | U3 Ch2 | 3,4,12,11 | R12 | GPIO5_DI2 | GPIO5 |
| DI3 | DI3+ | TVS4 | R5 | D4 | U3 Ch3 | 5,6,14,13 | R13 | GPIO6_DI3 | GPIO6 |
| DI4 | DI4+ | TVS5 | R6 | D5 | U3 Ch4 | 7,8,16,15 | R14 | GPIO7_DI4 | GPIO7 |
| DI5 | DI5+ | TVS6 | R7 | D6 | U4 Ch1 | 1,2,10,9 | R15 | GPIO8_DI5 | GPIO8 |
| DI6 | DI6+ | TVS7 | R8 | D7 | U4 Ch2 | 3,4,12,11 | R16 | GPIO9_DI6 | GPIO9 |
| DI7 | DI7+ | TVS8 | R9 | D8 | U4 Ch3 | 5,6,14,13 | R17 | GPIO10_DI7 | GPIO10 |
| DI8 | DI8+ | TVS9 | R10 | D9 | U4 Ch4 | 7,8,16,15 | R18 | GPIO11_DI8 | GPIO11 |

### J2 Connector (9-pin screw terminal, 5.08mm)

| Pin | Label | Connection |
|-----|-------|------------|
| 1 | DI1+ | → TVS2, R3 → U3 Ch1 LED anode |
| 2 | DI2+ | → TVS3, R4 → U3 Ch2 LED anode |
| 3 | DI3+ | → TVS4, R5 → U3 Ch3 LED anode |
| 4 | DI4+ | → TVS5, R6 → U3 Ch4 LED anode |
| 5 | DI5+ | → TVS6, R7 → U4 Ch1 LED anode |
| 6 | DI6+ | → TVS7, R8 → U4 Ch2 LED anode |
| 7 | DI7+ | → TVS8, R9 → U4 Ch3 LED anode |
| 8 | DI8+ | → TVS9, R10 → U4 Ch4 LED anode |
| 9 | DI- | → GND (common return for all channels) |

---

## SECTION 5: PWM OUTPUT STAGE

### Components: Q1, R19, R20, D10, R21, TVS10, J5

```
                                        J5 Pin 1: +12V_EXT
                                            │
                                       TVS10 (SMBJ15A)
                                       Cathode ── +12V_EXT
                                       Anode ── GND
                                            │
                                       D10 (SS34 flyback)
                                   ┌── Cathode ── +12V_EXT
                                   │   Anode ──┐
                                   │           │
                              J5 Pin 2: SOL+ ──┤
                                               │
                                          [SOLENOID]
                                          (external)
                                               │
                              J5 Pin 3: SOL- ──┤
                                               │
                                   SOLENOID_SW (net)
                                               │
                                          Q1 (IRLZ44N)
                                          Drain ── SOLENOID_SW
    GPIO1_PWM ── R19 (100R) ──┬── Gate
                              │
                         R20 (10k)
                              │
                             GND ── Source ──┬── SENSE_V (net)
                                             │       │
                                             │   GPIO3_ADC
                                             │   (to ESP32 GPIO3)
                                             │
                                        R21 (0.5R, 2W)
                                             │
                                            GND

                              J5 Pin 4: GND ── GND
```

**Connection summary:**
- J5 Pin 1 (+12V_EXT) → D10 cathode, TVS10 cathode
- J5 Pin 2 (SOL+) → one solenoid terminal, D10 anode
- J5 Pin 3 (SOL-) → other solenoid terminal → Q1 drain (SOLENOID_SW net)
- Q1 gate ← R19 (100R) ← GPIO1_PWM
- Q1 gate ← R20 (10k) → GND (pull-down)
- Q1 source → SENSE_V junction → R21 (0.5R) → GND
- Q1 source → GPIO3_ADC (to ESP32 ADC)
- J5 Pin 4 → GND
- TVS10 anode → GND
- D10 anode → SOL- / Q1 drain junction

**IRLZ44N (TO-220) pinout (facing flat side, pins down, left to right):**
- Pin 1: Gate
- Pin 2: Drain
- Pin 3: Source
- Tab: Drain (electrically connected to Pin 2)

---

## SECTION 6: FAULT OUTPUT

### Components: Q2, R22, R23, R24, J6

```
    +3V3 ── R24 (10k) ──┬── GPIO0 (ESP32 Pin 27)
                         │       (BOOT net — shared with auto-program)
                         │
                         ├── C_BOOT (100nF) ── GND
                         │
                         ├── Q4 collector (auto-boot, see Section 8)
                         │
                         └── R22 (1k) ──┬── Q2 (2N7002) Gate
                                        │
                                   R23 (100k)
                                        │
                                       GND ── Q2 Source

                              Q2 Drain ──── J6 Pin 1 (DO1 / FAULT)

                                            J6 Pin 2 (COM) ── GND
```

**Connection summary:**
- ESP32 GPIO0 → R22 (1k) → Q2 gate
- Q2 gate → R23 (100k) → GND (keeps off during boot)
- Q2 source → GND
- Q2 drain → J6 Pin 1 (open-drain output to PLC)
- J6 Pin 2 → GND

**How it connects to PLC:**
```
    PLC 24V supply ──── PLC digital input COM
    J6 Pin 1 (DO1) ──── PLC digital input terminal
    J6 Pin 2 (COM) ──── PLC 24V supply GND
```
When Q2 turns ON (fault): J6 Pin 1 sinks current → PLC input sees ON.
When Q2 is OFF (normal): J6 Pin 1 is floating → PLC input sees OFF.

**2N7002 (SOT-23) pinout:**
- Pin 1: Gate
- Pin 2: Source
- Pin 3: Drain

---

## SECTION 7: OLED DISPLAY CONNECTOR

### Components: J3, R25, R26

```
    J3 (4-pin header, 2.54mm)
    ┌──────┐
    │ Pin 1 │── +3V3 (VCC for OLED module)
    │ Pin 2 │── GND
    │ Pin 3 │──┬── SDA (net) ── ESP32 GPIO40
    │       │  └── R25 (4.7k) ── +3V3
    │ Pin 4 │──┬── SCL (net) ── ESP32 GPIO41
    │       │  └── R26 (4.7k) ── +3V3
    └──────┘
```

**Connection summary:**
- J3 Pin 1 → +3V3
- J3 Pin 2 → GND
- J3 Pin 3 → SDA net → ESP32 GPIO40, R25 pull-up to +3V3
- J3 Pin 4 → SCL net → ESP32 GPIO41, R26 pull-up to +3V3

---

## SECTION 8: USB-C PROGRAMMING INTERFACE

### Components: J4, U6, R27, R28, Q3, Q4, C_EN, C_BOOT, SW1, SW2

```
    J4 (USB-C Receptacle, USB4125-GF-A)
    ┌─────────────────────────────────────────────┐
    │                                             │
    │  VBUS (A4,B4) ── Not connected to board     │
    │                  power (board is 24V powered)│
    │                                             │
    │  D- (A7) ──┬── U6 Pin 1 (I/O1)             │
    │            └── USB_DN ── ESP32 GPIO19       │
    │                                             │
    │  D+ (A6) ──┬── U6 Pin 4 (I/O2)             │
    │            └── USB_DP ── ESP32 GPIO20       │
    │                                             │
    │  CC1 (A5) ── R27 (5.1k) ── GND             │
    │  CC2 (B5) ── R28 (5.1k) ── GND             │
    │                                             │
    │  GND (A1,B1,A12,B12) ── GND                │
    │  Shield ── GND                              │
    │                                             │
    └─────────────────────────────────────────────┘

    U6 (USBLC6-2SC6, SOT-23-6)
    ┌──────────────┐
    │ Pin 1 (I/O1) │── USB_DN (D-)
    │ Pin 2 (GND)  │── GND
    │ Pin 3 (I/O2) │── USB_DP (D+)
    │ Pin 4 (I/O2) │── USB_DP (D+)
    │ Pin 5 (VBUS) │── +3V3 (reference voltage for ESD clamp)
    │ Pin 6 (I/O1) │── USB_DN (D-)
    └──────────────┘
```

### Auto-Reset / Auto-Boot Circuit (Q3, Q4)

This circuit allows the USB serial interface to automatically reset the ESP32 into bootloader mode for firmware upload, without pressing buttons.

```
    USB DTR signal ──────── Q3 (2N7002) Gate
                            Q3 Source ── GND
                            Q3 Drain ──┬── EN (ESP32 EN pin)
                                       ├── R_EN (10k) ── +3V3
                                       └── C_EN (100nF) ── GND

    USB RTS signal ──────── Q4 (2N7002) Gate
                            Q4 Source ── GND
                            Q4 Drain ──┬── GPIO0 / BOOT (ESP32 GPIO0)
                                       ├── R24 (10k) ── +3V3
                                       └── C_BOOT (100nF) ── GND
```

**How auto-reset works:**
1. IDE asserts DTR LOW → Q3 turns ON → pulls EN LOW → ESP32 resets
2. IDE asserts RTS LOW → Q4 turns ON → pulls GPIO0 LOW → bootloader mode
3. IDE releases DTR → EN goes HIGH (via R_EN pull-up) → ESP32 starts in bootloader
4. After upload, IDE releases RTS → GPIO0 goes HIGH → normal boot on next reset

**Note on DTR/RTS signals:** The ESP32-S3 uses native USB (not a UART bridge). The auto-reset circuit works differently than classic ESP32. The ESP32-S3's USB-Serial-JTAG peripheral handles the reset internally in most cases. However, including Q3/Q4 provides compatibility with tools that use DTR/RTS toggling. If using only native USB CDC, Q3 and Q4 may not be needed, but including them costs almost nothing and provides flexibility.

### Manual Buttons (SW1, SW2)

```
    SW1 (RESET)
    Pin A ── EN (ESP32 EN pin)
    Pin B ── GND

    SW2 (BOOT)
    Pin A ── GPIO0 (ESP32 GPIO0)
    Pin B ── GND
```

Press RESET: EN goes LOW → ESP32 resets.
Hold BOOT + press RESET + release RESET: ESP32 enters bootloader mode.

---

## SECTION 9: COMPLETE CONNECTION CHECKLIST

### Power Rail Connections

| Component Pin | Connects To |
|--------------|-------------|
| J1 Pin 1 | +24V_RAW → TVS1 cathode → D1 anode |
| D1 cathode | F1 Pin 1 |
| F1 Pin 2 | +24V rail |
| U2 VIN | +24V |
| U2 VOUT | +3V3 |
| U1 3V3 | +3V3 |
| J3 Pin 1 | +3V3 |
| U6 Pin 5 | +3V3 |
| All pull-up resistors (R11-R18, R24, R25, R26, R_EN) | +3V3 |
| J5 Pin 1 | +12V_EXT (external 12V supply) |

### Ground Connections

| Component Pin | Connects To GND |
|--------------|-----------------|
| J1 Pin 2 | GND |
| J5 Pin 4 | GND |
| J6 Pin 2 | GND |
| J3 Pin 2 | GND |
| U1 GND pins (1, 40, 41, EP) | GND |
| U2 GND | GND |
| U3 emitters (all 4 channels) | GND |
| U4 emitters (all 4 channels) | GND |
| U6 Pin 2 | GND |
| Q1 source (through R21) | GND |
| Q2 source | GND |
| Q3 source | GND |
| Q4 source | GND |
| R20 (gate pull-down) | GND |
| R23 (gate pull-down) | GND |
| C1-C9 negative | GND |
| C_EN, C_BOOT | GND |
| R27, R28 | GND |
| TVS1-TVS10 anodes | GND |
| J2 Pin 9 (DI-) | GND |
| J4 GND pins + shield | GND |
| SW1 Pin B | GND |
| SW2 Pin B | GND |

### Signal Connections — ESP32 GPIO to Component

| ESP32 GPIO | Pin on Module | → Component | Through | Net Name |
|-----------|---------------|-------------|---------|----------|
| GPIO0 | 27 | Q2 gate (fault) | R22 (1k) | GPIO0_FAULT |
| GPIO0 | 27 | Q4 drain (auto-boot) | direct | BOOT |
| GPIO0 | 27 | SW2 Pin A | direct | BOOT |
| GPIO0 | 27 | R24 (10k pull-up) | to +3V3 | BOOT |
| GPIO0 | 27 | C_BOOT | to GND | BOOT |
| GPIO1 | 39 | Q1 gate (PWM MOSFET) | R19 (100R) | GPIO1_PWM |
| GPIO3 | 15 | R21/Q1 source junction | direct | GPIO3_ADC |
| GPIO4 | 4 | U3 Ch1 collector | direct | GPIO4_DI1 |
| GPIO5 | 5 | U3 Ch2 collector | direct | GPIO5_DI2 |
| GPIO6 | 6 | U3 Ch3 collector | direct | GPIO6_DI3 |
| GPIO7 | 7 | U3 Ch4 collector | direct | GPIO7_DI4 |
| GPIO8 | 12 | U4 Ch1 collector | direct | GPIO8_DI5 |
| GPIO9 | 17 | U4 Ch2 collector | direct | GPIO9_DI6 |
| GPIO10 | 18 | U4 Ch3 collector | direct | GPIO10_DI7 |
| GPIO11 | 19 | U4 Ch4 collector | direct | GPIO11_DI8 |
| GPIO19 | 13 | J4 USB D- | via U6 ESD | USB_DN |
| GPIO20 | 14 | J4 USB D+ | via U6 ESD | USB_DP |
| GPIO40 | 33 | J3 Pin 3 (SDA) | direct | SDA |
| GPIO41 | 34 | J3 Pin 4 (SCL) | direct | SCL |
| EN | 3 | Q3 drain, SW1, R_EN, C_EN | direct | EN |

---

## SECTION 10: COMPONENT TOTAL COUNT VERIFICATION

| Type | Refs | Individual Count |
|------|------|-----------------|
| ESP32-S3 module | U1 | 1 |
| DC-DC module | U2 | 1 |
| Quad optocouplers | U3, U4 | 2 |
| OLED display | U5 | 1 (off-board) |
| ESD protection | U6 | 1 |
| N-MOSFET (TO-220) | Q1 | 1 |
| N-MOSFET (SOT-23) | Q2, Q3, Q4 | 3 |
| Resistors | R3-R28 | 26 |
| Capacitors | C1,C2,C4-C9,C_EN,C_BOOT | 10 |
| Schottky diodes | D1, D10 | 2 |
| Signal diodes | D2-D9 | 8 |
| TVS diodes | TVS1-TVS10 | 10 |
| Fuse | F1 | 1 |
| Connectors | J1,J2,J3,J4,J5,J6 | 6 |
| Switches | SW1, SW2 | 2 |
| **TOTAL components on PCB** | | **75** (excluding U5 off-board) |

---

## SECTION 11: NOTES FOR KICAD SCHEMATIC ENTRY

1. **Use hierarchical sheets** for clarity: put Sections 1-2 (power) on one sheet, Section 3 (ESP32) on another, Section 4 (digital inputs) on a third, and Sections 5-8 (output, fault, OLED, USB) on a fourth.

2. **Add PWR_FLAG symbols** on +24V, +3V3, +12V_EXT, and GND nets to prevent ERC warnings.

3. **Mark unused ESP32 pins** with No Connect flags (X symbol) to avoid ERC errors.

4. **The R_EN pull-up resistor** for the EN pin is NOT in your BOM. You need to **add it**: R_EN, 10kohm, pull-up on EN pin to +3V3. This is essential for the ESP32 to boot. Add it as line item in the BOM.

5. **GPIO0 is shared** between three functions: fault output (R22→Q2), auto-boot (Q4 drain), and boot button (SW2). This is fine because during normal operation Q4 is off and SW2 is open, so GPIO0 is free for the firmware to drive Q2.

6. **Verify the exact TLP281-4 pinout** for the package you select (DIP vs SOP). The pin numbering differs between packages. The channel mapping table in Section 4 assumes the standard DIP-16 pinout.
