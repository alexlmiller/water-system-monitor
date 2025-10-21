# water-system-monitor
monitoring a home's water system with ESP32 and sensors

# ESP32 Well Water Monitoring System - Complete Build Plan

## System Overview

This system monitors and controls a residential well water system with pressure tank, water filter, and water heater. All sensors connect to a single ESP32 running ESPHome, integrated with Home Assistant.

### Monitoring Capabilities

**Flow & Control:**
- Water flow rate (L/min) and total usage (gallons/m³)
- Remote motorized valve shutoff (normally-closed fail-safe)

**Pressure Monitoring (3 points):**
- Well pump output pressure (before tank)
- System pressure (after tank, before filter)
- Filtered water pressure (after filter)
- Calculated filter pressure drop

**Temperature Monitoring (2 points):**
- Incoming cold water temperature
- Hot water heater output temperature
- Calculated temperature rise

**Water Quality:**
- TDS (Total Dissolved Solids) - mineral content
- pH - acidity/alkalinity

**Pump Health:**
- Cycle counting
- Runtime tracking
- Current sensing

**Safety:**
- Leak detection (mechanical room floor)
- Automatic valve closure on leak
- High/low pressure alerts
- Scalding temperature alerts
- Water quality alerts

---

## Complete Hardware List

### Core Components
| Item | Qty | Unit Price | Total | Notes |
|------|-----|------------|-------|-------|
| ESP32 DevKit V1 | 1 | $6 | $6 | ESP-WROOM-32 |
| YF-B10 Flow Meter 1" | 1 | $18 | $18 | 660 pulses/L, brass |
| U.S. Solid Motorized Valve 1" SS | 1 | $52 | $52 | 2-wire normally-closed, 9-24V |
| 5V Relay Module | 1 | $6 | $6 | SRD-05VDC-SL-C |
| Mean Well LRS-50-12 | 1 | $22 | $22 | 12V 5A power supply |
| LM2596 Buck Converter | 1 | $2 | $2 | 12V to 5V |

### Pressure Monitoring
| Item | Qty | Unit Price | Total | Notes |
|------|-----|------------|-------|-------|
| ADS1115 ADC Module | 1 | $4 | $4 | 16-bit, 4-channel I2C |
| 0-100 PSI Pressure Sensor | 3 | $15 | $45 | 1/4" NPT, 0.5-4.5V output |
| 1/4" NPT Brass Tees | 3 | $4 | $12 | For sensor mounting |

### Temperature Monitoring
| Item | Qty | Unit Price | Total | Notes |
|------|-----|------------|-------|-------|
| DS18B20 Waterproof Probe 3m | 2 | $4 | $8 | 1-Wire digital sensor |
| 4.7kΩ Resistor | 1 | $0.10 | $0.10 | Pull-up for 1-Wire |
| Thermal Paste | 1 | $3 | $3 | Pipe contact |
| Stainless Hose Clamps | 2 | $2 | $4 | Sensor mounting |

### Pump Monitoring
| Item | Qty | Unit Price | Total | Notes |
|------|-----|------------|-------|-------|
| SCT-013 Current Sensor | 1 | $12 | $12 | Split-core CT, clamps on wire |
| 33Ω Burden Resistor | 1 | $0.20 | $0.20 | Current to voltage |
| 10µF Capacitor | 1 | $0.30 | $0.30 | Noise filtering |

### Water Quality
| Item | Qty | Unit Price | Total | Notes |
|------|-----|------------|-------|-------|
| TDS Sensor Module | 1 | $16 | $16 | Analog output |
| pH Sensor Module | 1 | $30 | $30 | Analog output with BNC |
| pH Calibration Buffers | 1 | $10 | $10 | pH 4.0 and 7.0 |
| Sensor Mounting Tees | 2 | $8 | $16 | 1/4" or inline chamber |

### Leak Detection & Safety
| Item | Qty | Unit Price | Total | Notes |
|------|-----|------------|-------|-------|
| Water Leak Sensor | 1 | $4 | $4 | Simple probe type |

### Miscellaneous
| Item | Qty | Unit Price | Total | Notes |
|------|-----|------------|-------|-------|
| IP65 Enclosure 200x200x95mm | 1 | $18 | $18 | Large enough for all components |
| 1N4004 Flyback Diodes | 2 | $0.10 | $0.20 | Valve protection |
| P6KE18CA TVS Diodes | 2 | $0.60 | $1.20 | Transient protection |
| 470µF Capacitor | 1 | $0.50 | $0.50 | Power filtering |
| 100µF Capacitor | 2 | $0.30 | $0.60 | Power filtering |
| 0.1µF Capacitor | 3 | $0.10 | $0.30 | Decoupling |
| Cable Glands M16 IP65 | 4 | $1.50 | $6 | Wire entry |
| Terminal Blocks 2.5mm² | 15 | $0.50 | $7.50 | Connections |
| Wire 18AWG/22AWG | - | $8 | $8 | Power and signal |

**Total System Cost: $278**

---

## GPIO Pin Assignments

```
Flow Meter:
  GPIO21 - Pulse input (with 10kΩ pull-up to 3.3V)

Temperature Sensors (1-Wire):
  GPIO22 - DS18B20 data line (with 4.7kΩ pull-up to 3.3V)

Valve Control:
  GPIO25 - Relay control output

Leak Detection:
  GPIO26 - Water leak sensor (INPUT_PULLUP)

Pump Monitoring:
  GPIO33 - Current sensor analog input

Water Quality:
  GPIO34 - TDS sensor analog input
  GPIO35 - pH sensor analog input

Pressure Sensors (I2C via ADS1115):
  GPIO21 (SDA) - I2C data
  GPIO22 (SCL) - I2C clock
  Note: ADS1115 channels A0, A1, A2 for 3 pressure sensors

Spare:
  GPIO4 - Future expansion (turbidity sensor, etc.)
```

---

## Wiring Diagrams

### Power Distribution
```
120V AC Wall Power
    ↓
Mean Well LRS-50-12 (12V 5A)
    ├─→ [LM2596 Buck] → 5V → ESP32 VIN + Relay VCC
    │                          ├─ 100µF capacitor to GND
    │                          └─ 0.1µF capacitor to GND
    └─→ 12V Valve Power (via relay NO contact)
```

### Flow Meter Connections
```
YF-B10 Flow Meter        ESP32 DevKit
Red (VCC)         ────→  5V (VIN)
Black (GND)       ────→  GND
Yellow (Signal)   ────→  GPIO21 ─┬─ 10kΩ resistor ─→ 3.3V
                                  └─ 0.1µF cap to GND
```

### Motorized Valve Control
```
ESP32               5V Relay           Valve            12V PSU
GPIO25    ────→     IN                 
5V        ────→     VCC                
GND       ────→     GND                
                    COM      ────→     Red (power)  ←── 12V+
                    NO       ────┐                      
                                 └───→ Red (power)
                                       Black (GND)  ──→ GND

Protection (across valve terminals):
  - 1N4004 diode: cathode to Red, anode to Black
  - P6KE18CA TVS diode: parallel across terminals
```

### Pressure Sensors (via ADS1115)
```
ADS1115 Module      ESP32
VDD        ────→    3.3V
GND        ────→    GND
SCL        ────→    GPIO22
SDA        ────→    GPIO21

Pressure Sensors → ADS1115:
  Pump Output Sensor:    VCC→5V, GND→GND, Signal→A0
  System Pressure:       VCC→5V, GND→GND, Signal→A1
  Filtered Pressure:     VCC→5V, GND→GND, Signal→A2
```

### Temperature Sensors
```
DS18B20 Probes      ESP32
All VCC (Red)  ────→ 5V
All GND (Black)────→ GND
All Data (Yellow)──→ GPIO22 ─── 4.7kΩ resistor ─→ 3.3V
```

### Current Sensor (Pump Monitoring)
```
SCT-013 CT Sensor
  Clamps around pump HOT wire (no cutting)
  Output via 3.5mm jack:
    Tip    ─── 33Ω resistor ─┬─→ GPIO33
    Sleeve ─── to GND        └─ 10µF capacitor to GND
```

### Water Quality Sensors
```
TDS Sensor          ESP32
VCC        ────→    5V
GND        ────→    GND
Signal     ────→    GPIO34 ─── 0.1µF cap to GND

pH Sensor           ESP32
VCC        ────→    5V
GND        ────→    GND
Signal     ────→    GPIO35 ─── 0.1µF cap to GND
```

### Leak Sensor
```
Water Leak Sensor   ESP32
VCC        ────→    3.3V
Signal     ────→    GPIO26 (INPUT_PULLUP mode)
```

---

## Calibration Procedures

### Flow Meter Calibration
1. Fill a precise 10-liter container while monitoring ESPHome logs
2. Note total pulse count from logs
3. Calculate: `actual_pulses_per_liter = total_pulses / 10`
4. Update YAML multiplier: `multiply: 60 / actual_pulses_per_liter`
5. Reflash ESP32

### Pressure Sensor Calibration
1. Verify sensors read atmospheric pressure (~0 PSI) when disconnected
2. With pump running, compare readings to mechanical gauge
3. Adjust `calibrate_linear` values if needed

### pH Sensor Calibration
1. Rinse probe with distilled water
2. Immerse in pH 7.0 buffer, wait 2 minutes, note voltage
3. Immerse in pH 4.0 buffer, wait 2 minutes, note voltage
4. Update `calibrate_linear` values in YAML
5. Recalibrate monthly

### TDS Sensor Calibration
1. Use calibration solution (usually 342 ppm or 1413 µS/cm)
2. Immerse sensor, note voltage reading
3. Adjust `calibrate_linear` values accordingly

### Current Sensor Calibration
1. Measure actual pump current with clamp meter
2. Compare to ESPHome reading
3. Adjust `calibrate_linear` values to match

---

## Installation Notes

### Sensor Locations
- **Pump output pressure**: Before pressure tank (cut pipe, add tee)
- **System pressure**: After tank, before filter (may have test port)
- **Filtered pressure**: After filter (may have test port)
- **Incoming temp**: Strap to incoming cold water line with thermal paste
- **Hot water temp**: Strap to hot water outlet with thermal paste
- **TDS/pH**: After filtration, in tee or inline chamber
- **Current sensor**: Clamp around pump power wire (HOT)
- **Leak sensor**: Center of mechanical room floor

### Critical Safety
- GFCI protection required for all AC power
- Use cable glands for all enclosure penetrations
- Install flyback and TVS diodes on motorized valve
- Keep electronics away from potential water spray
- Label all wiring clearly

### First Boot Procedure
1. Flash ESPHome via USB first time
2. Check logs for DS18B20 addresses, update YAML
3. Verify all sensors reading reasonable values
4. Test valve operation (should close on boot)
5. Calibrate all sensors
6. Test leak sensor triggers valve
7. Monitor for 24 hours before relying on automation

---

## Home Assistant Integration

After ESPHome device is online, it will automatically appear in Home Assistant. Create automations for:

- **Filter replacement reminder** (pressure drop > 20 PSI)
- **Vacation mode** (close valve when away)
- **Leak alerts** (push notifications)
- **Water quality alerts** (TDS/pH out of range)
- **Pump maintenance** (runtime hours threshold)
- **Energy dashboard** (water usage tracking)

---

## Troubleshooting

**Flow meter reads zero:**
- Check 10kΩ pull-up resistor
- Verify flow direction arrow
- Ensure 5-inch straight pipe upstream

**Pressure sensors unstable:**
- Add averaging (already in config)
- Check power supply voltage
- Verify ADS1115 I2C address (0x48)

**Temperature sensors not found:**
- Check 4.7kΩ pull-up resistor
- Verify DS18B20 wiring (VCC=5V, not 3.3V recommended)
- Look in logs for addresses

**pH/TDS readings erratic:**
- Calibrate sensors
- Clean electrode/probe
- Check temperature (affects readings)
- Add capacitors for noise filtering

**Pump current sensor not working:**
- Verify CT clamp around only HOT wire
- Check burden resistor (33Ω)
- Ensure correct orientation of CT clamp

---

This document provides everything needed to build and deploy the system. Review with your knowledgeable friends, then proceed with hardware assembly and testing.
