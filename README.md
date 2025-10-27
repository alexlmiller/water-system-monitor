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

### Physical Control Panel
| Item | Qty | Unit Price | Total | Notes |
|------|-----|------------|-------|-------|
| SPDT Toggle Switch | 1 | $4 | $4 | Auto/Manual mode |
| Momentary Pushbuttons 16mm | 2 | $5 | $10 | Illuminated preferred |
| Panel Mount LEDs 12mm | 4 | $1.50 | $6 | 2x Green, 2x Red |
| 220Ω Resistors 1/4W | 4 | $0.10 | $0.40 | LED current limiting |

### Pump Interlock
| Item | Qty | Unit Price | Total | Notes |
|------|-----|------------|-------|-------|
| 30A Solid State Relay | 1 | $15 | $15 | Fotek SSR-40DA or similar |
| OR: 30A Contactor 240V | 1 | $20 | $20 | Siemens 3TH4031-0A |

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

**Total System Cost: $313** (with SSR) **or $318** (with contactor)

---

## GPIO Pin Assignments

```
Flow Meter:
  GPIO21 - Pulse input (with 10kΩ pull-up to 3.3V)

Temperature Sensors (1-Wire):
  GPIO22 - DS18B20 data line (with 4.7kΩ pull-up to 3.3V)

Valve & Pump Control:
  GPIO25 - Main valve relay control
  GPIO27 - Pump interlock relay control

Leak Detection:
  GPIO26 - Water leak sensor (INPUT_PULLUP)

Pump Monitoring:
  GPIO33 - Current sensor analog input

Water Quality:
  GPIO34 - TDS sensor analog input
  GPIO35 - pH sensor analog input

Pressure Sensors (I2C via ADS1115):
  GPIO21 (SDA) - I2C data (shared with flow meter pin)
  GPIO22 (SCL) - I2C clock (shared with 1-Wire pin)
  Note: ADS1115 channels A0, A1, A2 for 3 pressure sensors

Physical Control Panel Inputs:
  GPIO15 - Auto/Manual mode toggle switch (INPUT_PULLUP)
  GPIO14 - Manual Open pushbutton (INPUT_PULLUP)
  GPIO12 - Manual Close pushbutton (INPUT_PULLUP)

Status Indicator LEDs:
  GPIO16 - Valve Open LED (Green)
  GPIO17 - Valve Closed LED (Red)
  GPIO18 - Pump Enabled LED (Green)
  GPIO19 - Leak Alert LED (Red)

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

### Physical Control Panel

**Mode Toggle Switch:**
```
SPDT Toggle         ESP32
Common     ────→    GPIO15
Position 1 ────→    GND (Manual mode)
Position 2 ────→    No connection (Auto mode - pulled HIGH)
```

**Pushbuttons:**
```
Momentary Button    ESP32
Open Button:
  Pin 1      ────→  GPIO14
  Pin 2      ────→  GND

Close Button:
  Pin 1      ────→  GPIO12
  Pin 2      ────→  GND

Note: Both use INPUT_PULLUP, so button press pulls pin LOW
```

**Status LEDs (all 4 wired identically):**
```
ESP32 GPIO → 220Ω Resistor → LED Anode (+)
                              LED Cathode (-) → GND

GPIO16 → 220Ω → Green LED (Valve Open)
GPIO17 → 220Ω → Red LED (Valve Closed)
GPIO18 → 220Ω → Green LED (Pump Enabled)
GPIO19 → 220Ω → Red LED (Leak Alert)
```

### Pump Interlock Relay

**Control Side:**
```
Solid State Relay   ESP32 / Power
DC+ (Control)  ────→ GPIO27
DC- (Control)  ────→ GND
```

**Power Side (240V - IN PUMP CONTROL BOX):**
```
240V Circuit Breaker
       │
       ├────→ Relay Input (+)
       │      
       └────→ Pressure Switch → Relay Output (+) → Pump Hot
       
Neutral: Direct to pump (not through relay)
Ground: Direct to pump

When GPIO27 HIGH: Relay conducts, pump can run
When GPIO27 LOW: Relay open, pump disabled
```

**CRITICAL SAFETY:** 
- Install pump relay in existing pump control box
- Use strain reliefs on all 240V connections
- Verify all connections with licensed electrician
- Label clearly: "AUTOMATED INTERLOCK - DO NOT BYPASS"

---

## Troubleshooting

**Control panel not responding:**
- Verify toggle switch wiring (common to GPIO15, one position to GND)
- Check pushbutton connections (one side to GPIO, other to GND)
- Verify INPUT_PULLUP working (pins should read HIGH when not pressed)
- Check logs for button press events when testing

**LEDs not lighting:**
- Verify 220Ω resistors installed on each LED
- Check LED polarity (long leg = anode/+, short leg = cathode/-)
- Test LED directly with 3.3V and resistor
- Verify GPIO pins not used elsewhere
- Check interval is running (should update every 500ms)

**Valve won't open in MANUAL mode:**
- Check leak sensor not detecting water
- Verify toggle switch in MANUAL position (GPIO15 reads LOW)
- Check logs for "Manual OPEN command" message
- Test valve opens from Home Assistant in AUTO mode

**Buttons work in AUTO mode (shouldn't):**
- Verify toggle switch wiring correct
- Check lambda condition checking `manual_mode_switch` state
- Review logs to confirm mode switch state

**Pump runs with valve closed:**
- Check pump interlock relay wiring
- Verify GPIO27 follows valve state (HIGH=open, LOW=closed)
- Test relay manually - should click when GPIO27 changes
- Check pump control box wiring to relay

**Pump won't run with valve open:**
- Verify pump interlock relay energized (GPIO27 HIGH)
- Check SSR/contactor control voltage present
- Verify pressure switch working (manual test)
- Check relay output connections to pump

**Leak alert LED not blinking:**
- Verify leak sensor triggering (check logs)
- Test leak sensor with wire to GND
- Check LED wiring and resistor
- Verify interval running (should blink at 500ms)

**Valve won't open after leak:**
- Check leak sensor shows "dry" in Home Assistant
- Sensor may need time to fully dry (use fan/towel)
- Check logs for "Cannot open valve - leak still detected!" message
- Temporarily unplug sensor if emergency access needed (not recommended)

**Continuous flow alert triggering during normal use:**
- This is informational - no auto-shutoff occurs
- Adjust threshold in YAML if needed (currently 5 L/min for 30 min)
- Create Home Assistant automation for your preferred handling
- Consider "away mode" condition for auto-shutoff

**Filter alert too sensitive:**
- Adjust threshold from 15 PSI to higher (20-25 PSI)
- Verify alert only triggers during low flow (<2 L/min)
- Check actual pressure drop with mechanical gauge
- Some filters naturally have higher drop when new

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
- Look in logs for addresses (set logger to DEBUG)

**pH/TDS readings erratic:**
- Calibrate sensors
- Clean electrode/probe
- Check temperature (affects readings)
- Add capacitors for noise filtering

**Pump current sensor not working:**
- Verify CT clamp around only HOT wire
- Check burden resistor (33Ω)
- Ensure correct orientation of CT clamp

**Short cycle alerts when tank is fine:**
- This counts cycles over 24 hours (>5 short cycles)
- Single short cycle is normal and won't alert
- Adjust threshold if you have small fixtures causing micro-cycles

---

This document provides everything needed to build and deploy the system. Review with your knowledgeable friends, then proceed with hardware assembly and testing.

## Revision History

**Version 1.1** - Incorporated engineer feedback:
- Added leak sensor interlock to prevent valve opening during leak
- Changed continuous flow from auto-shutoff to alert-only (prevents false shutoffs during legitimate high use)
- Added flow condition to filter pressure drop alert (only alerts during low/no flow)
- Added time delays to pump pressure alerts (reduces transient false alerts)
- Improved pressure tank alert to count cycles over 24 hours instead of single occurrence
- Added proper globals for state tracking instead of static variables
- Enhanced Home Assistant integration examples with actionable notification automation
- Expanded troubleshooting section with new logic explanations
