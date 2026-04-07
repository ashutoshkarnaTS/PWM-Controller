# Trace Width and Clearance Guide
## PWM Solenoid Controller Custom PCB

---

## Trace Widths

| Net | Current | Recommended Width |
|-----|---------|------------------|
| 24V input (J1 → D1 → F1 → U2) | ~0.3A | **1.0mm** |
| 12V solenoid path (J5 → D10 → Q1 drain → Q1 source) | **1.3A** | **2.0mm** or copper pour |
| Sense resistor to GND (R21 → GND) | 1.3A | **2.0mm** |
| 3.3V rail (U2 VOUT → ESP32, pull-ups, OLED) | ~0.3A | **0.5mm** |
| DI input traces (J2 → resistors → optocouplers) | 2.3mA | **0.25mm** |
| GPIO signals (ESP32 → gate resistors, I2C, ADC) | <10mA | **0.25mm** |
| USB D+/D- | <10mA | **0.25mm** (keep equal length, parallel, close together) |
| Ground | All return current | **Full copper pour on back layer** |

---

## Minimum Clearances

| Boundary | Clearance | Reason |
|----------|-----------|--------|
| 24V traces to 3.3V traces | **2.0mm** | IEC 60950 reinforced insulation |
| 24V traces to GND | **0.5mm** | Same circuit, functional insulation |
| Optocoupler input to output (across isolation barrier) | **2.5mm** | Maintains optocoupler's isolation rating |
| 12V solenoid traces to 3.3V logic | **1.5mm** | Different supply domains |
| USB connector to 24V traces | **2.5mm** | User safety — USB is touchable |
| DI channel to DI channel | **0.5mm** | Same voltage level |
| Any trace to board edge | **0.3mm** | Manufacturing margin |
| Any trace to any trace (default minimum) | **0.2mm** | Standard fab capability |

---

## KiCad Design Rules Setup

In **File → Board Setup → Design Rules → Constraints**, set:

| Parameter | Value |
|-----------|-------|
| Minimum track width | **0.25mm** |
| Minimum clearance | **0.2mm** |
| Minimum via diameter | **0.8mm** |
| Minimum via drill | **0.4mm** |
| Copper to edge clearance | **0.3mm** |

Then in **Design Rules → Net Classes**, create:

| Net Class | Track Width | Clearance | Assign To |
|-----------|-------------|-----------|-----------|
| Default | 0.25mm | 0.2mm | All signal traces |
| Power | 1.0mm | 0.5mm | +24V, +3V3 |
| HighCurrent | 2.0mm | 0.5mm | SOLENOID_SW, SENSE_V, +12V_EXT, Q1 drain/source |
| Isolation | 0.25mm | 2.5mm | Optocoupler input-side nets (DI1+ through DI8+) |

The isolation net class is the one most people forget — it ensures KiCad enforces the 2.5mm gap across the optocoupler barrier automatically during DRC.
