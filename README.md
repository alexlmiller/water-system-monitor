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
  GPIO4 - Pulse input (with 10kΩ pull-up to 3.3V)

Temperature Sensors (1-Wire):
  GPIO23 - DS18B20 data line (with 4.7kΩ pull-up to 3.3V)

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
  GPIO21 (SDA) - I2C data
  GPIO22 (SCL) - I2C clock
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

Note: GPIO21 and GPIO22 are the standard I2C pins on ESP32
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
Yellow (Signal)   ────→  GPIO4 ─┬─ 10kΩ resistor ─→ 3.3V
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
All Data (Yellow)──→ GPIO23 ─── 4.7kΩ resistor ─→ 3.3V
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

## Complete ESPHome YAML Configuration

**The complete ESPHome YAML configuration is in a separate file: `well_water_system.yaml`**

Key configuration notes:
- All GPIO pins have been corrected to avoid conflicts
- Flow meter uses GPIO4 with pulse counter
- I2C uses standard pins GPIO21 (SDA) and GPIO22 (SCL) for ADS1115 pressure sensors
- 1-Wire uses GPIO23 for DS18B20 temperature sensors
- Physical control panel uses GPIO12, GPIO14, GPIO15 for buttons and mode switch
- Status LEDs on GPIO16, GPIO17, GPIO18, GPIO19 with proper output definitions
- All safety interlocks and monitoring logic implemented
- Includes services for resetting counters

**Important:** Update the DS18B20 addresses in the YAML after first boot by checking the logs in DEBUG mode.

---

## Important Safety Logic & Design Decisions

### Leak Detection Interlock
The valve **cannot be opened** while the leak sensor detects water. This prevents accidentally flooding after a leak event. To reopen the valve:
1. Fix the leak source
2. Dry the leak sensor completely
3. Verify sensor shows "dry" in Home Assistant
4. Then open the valve from HA

If the valve immediately closes again after opening, the sensor is still detecting moisture.

### Continuous Flow Alert (Not Auto-Shutoff)
The system monitors for continuous flow >5 L/min for 30+ minutes and **alerts only** - it does NOT automatically close the valve. This prevents false shutoffs during legitimate high water use like:
- Filling a hot tub or pool
- Running lawn sprinklers for extended periods
- Multiple showers/laundry running simultaneously

**Recommended Home Assistant automation:**
```yaml
automation:
  - alias: "Water - Continuous Flow Alert"
    trigger:
      - platform: state
        entity_id: binary_sensor.well_water_system_continuous_flow_alert
        to: "on"
    action:
      - service: notify.mobile_app
        data:
          title: "High Water Usage Alert"
          message: "Water has been flowing continuously for 30+ minutes at high rate. Check for leaks or forgotten fixtures."
      # Optional: Auto-close only when away
      # - condition: state
      #   entity_id: input_boolean.away_mode
      #   state: "on"
      # - service: switch.turn_off
      #   entity_id: switch.well_water_system_main_valve
```

### Filter Pressure Drop - Flow Compensation
The filter alert only triggers when flow is low (<2 L/min) AND pressure drop is high (>15 PSI). During high flow, pressure drop is normal and expected. The 15 PSI threshold is conservative - adjust based on your filter specifications after observing normal operation.

### Pump Alerts - Time Delays
- **Low Pump Pressure**: Requires 10 seconds of sustained low pressure to avoid transient alerts during pump startup
- **Pressure Tank Issues**: Counts short cycles over 24 hours (>5 cycles < 1 gallon) rather than alerting on single occurrences

### Debug vs Production Logging
- **During setup**: Set `logger: level: DEBUG` to see DS18B20 addresses, raw sensor values, and detailed operation
- **Production**: Use `level: INFO` or `WARN` to reduce log spam while keeping important events (valve operations, leak alerts)

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
- **Current sensor**: Clamp around pump power wire (HOT wire only)
- **Leak sensor**: Center of mechanical room floor, lowest point

### Motorized Valve Location
**Install valve AFTER pressure tank** for these reasons:
- ✅ No risk of pump deadheading
- ✅ Standard industry configuration
- ✅ Mechanical room leak sensor catches tank/inlet failures
- ✅ Manual valve before tank already exists for service
- ✅ Pump interlock provides redundant protection

### Pump Interlock Installation
**CRITICAL - Licensed Electrician Required for 240V Work**

1. **Locate pump control box** (usually near pressure tank)
2. **Identify pump circuit**: 240V from breaker → pressure switch → pump
3. **Install SSR/Contactor**: Insert in line between pressure switch and pump
4. **Control wiring**: Run low-voltage wire from ESP32 enclosure to pump box
5. **Test thoroughly**: Verify pump only runs when GPIO27 is HIGH
6. **Label clearly**: "AUTOMATED INTERLOCK - DO NOT BYPASS"

**Safety checklist:**
- [ ] All 240V connections in approved junction boxes
- [ ] Proper wire gauge for pump amperage
- [ ] Strain reliefs on all connections
- [ ] Ground bonding verified
- [ ] Cover plates secured
- [ ] Licensed electrician inspection

### Control Panel Assembly

**Enclosure top panel layout:**
```
┌─────────────────────────────────────────────┐
│                                             │
│   MODE: [AUTO  ●  MANUAL]  Toggle Switch   │
│                                             │
│   MANUAL: [OPEN] [CLOSE]   Pushbuttons     │
│                                             │
│   STATUS:  ●Open  ○Closed  ●Pump  ○Leak    │
│           LEDs   LEDs      LED    LED       │
│                                             │
└─────────────────────────────────────────────┘
```

**Drilling template:**
- Toggle switch: 12mm hole
- Pushbuttons: 16mm holes  
- LEDs: 12mm holes
- Space components 1" apart minimum

**Panel materials:**
- Polycarbonate or ABS plastic panel
- Mount on hinges for access to interior
- Use rubber grommets for all holes
- Label with permanent marker or label maker

### Critical Safety
- GFCI protection required for all AC power
- Use cable glands for all enclosure penetrations
- Install flyback and TVS diodes on motorized valve
- Keep 240V wiring in separate conduit from low voltage
- Label all wiring clearly
- Test ground continuity on all metal parts

### First Boot Procedure
1. **Flash ESPHome via USB** first time
2. **Check logs** for DS18B20 addresses, update YAML with actual addresses
3. **Verify all sensors** reading reasonable values in logs
4. **Test valve operation** (should be closed on boot, pump interlock off)
5. **Test control panel:**
   - Verify all LEDs illuminate correctly
   - Test AUTO mode - valve control from Home Assistant works
   - Test MANUAL mode - toggle switch, both buttons function
   - Verify mode switch prevents HA control in manual mode
6. **Test leak sensor** triggers valve AND pump interlock
7. **Calibrate all sensors** per procedures below
8. **Monitor for 24 hours** before relying on automation

### Control Panel Testing Sequence
1. **Power on in AUTO mode** (toggle switch to AUTO position)
   - Valve Closed LED should be ON (red)
   - Pump Enabled LED should be OFF
   - Open valve from Home Assistant
   - Verify Valve Open LED turns ON (green), Closed LED turns OFF
   - Verify Pump Enabled LED turns ON (green)

2. **Switch to MANUAL mode**
   - Toggle switch to MANUAL position
   - Press OPEN button
   - Verify valve opens, LEDs update
   - Press CLOSE button
   - Verify valve closes, LEDs update
   - Try to control valve from Home Assistant - should not work in MANUAL mode

3. **Test leak interlock**
   - Trigger leak sensor (short to ground)
   - Verify Leak Alert LED blinks
   - Verify valve closes automatically
   - Verify Pump Enabled LED turns OFF
   - Try to open valve (both manual and HA) - should be blocked
   - Remove leak sensor trigger
   - Verify can now open valve

4. **Return to AUTO mode**
   - Toggle switch back to AUTO
   - Verify Home Assistant control works
   - Test pump interlock follows valve state

---

## Home Assistant Integration

After ESPHome device is online, it will automatically appear in Home Assistant. Create automations for:

- **Continuous flow alert** (notification + optional auto-close when away) - **REQUIRED**
- **Leak alerts** (push notifications, close valve confirmation)
- **Filter replacement reminder** (pressure drop > 15 PSI at low flow)
- **Vacation mode** (close valve when away, alert on any activity)
- **Water quality alerts** (TDS/pH out of range)
- **Pump maintenance** (runtime hours threshold, short cycling)
- **Energy dashboard** (water usage tracking in m³)
- **Scalding prevention** (hot water temperature > 125°F)

**Example: Continuous Flow with Smart Shutoff**
```yaml
automation:
  - alias: "Water - Continuous Flow with Confirmation"
    trigger:
      - platform: state
        entity_id: binary_sensor.well_water_system_continuous_flow_alert
        to: "on"
    action:
      # Send notification
      - service: notify.mobile_app
        data:
          title: "High Water Usage Alert"
          message: "Water flowing continuously for 30+ minutes. Respond 'CLOSE' to shut valve."
          data:
            actions:
              - action: "CLOSE_VALVE"
                title: "Close Valve"
              - action: "DISMISS"
                title: "Ignore (Normal Use)"
      
  # Handle response
  - alias: "Water - Close Valve Response"
    trigger:
      - platform: event
        event_type: mobile_app_notification_action
        event_data:
          action: "CLOSE_VALVE"
    action:
      - service: switch.turn_off
        entity_id: switch.well_water_system_main_valve
      - service: notify.mobile_app
        data:
          message: "Main water valve closed."
```

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
- Added proper globals for state tracking instead of static variables
- Enhanced Home Assistant integration examples with actionable notification automation
- Expanded troubleshooting section with new logic explanations
