# ESPHome Pool Control System

An automated pool control system built with ESPHome and Home Assistant, featuring intelligent valve management, pump protection, and automatic water level control.

## Features

- **3 Pool Valve Control**: Automated control of filter, overflow edge, and vacuum line valves
- **Smart Pump Protection**: Monitors pump power consumption to detect blockages or mechanical issues
- **Automatic Water Level Management**: Maintains pool water level with automatic fill valve
- **Safety Interlocks**: Ensures only one valve operates at a time to protect the pump
- **LCD Status Display**: Real-time system status with valve animations
- **Manual Override**: Physical buttons for local control of each valve
- **Home Assistant Integration**: Full remote monitoring and control

## How It Works

### Valve Operation
1. Only one valve can be open at a time (enforced by software interlock)
2. When a valve is activated:
   - All other valves close
   - Pump stops
   - Selected valve opens (10-second motorized valve opening time)
   - Pump starts after valve is fully open
3. Manual buttons provide instant local control

### Pump Protection
- Continuously monitors power consumption via PZEM sensor
- Normal operating range: 810-1050W
- Automatic shutdown if 3 consecutive readings are out of range
- 40-second startup grace period to handle initial surge
- Display shows "Posible atasco!" (Possible blockage!) on detection

### Water Level Control
- Float switch monitors pool water level
- When low level detected:
  - All valves close
  - Pump stops (prevents damage from running dry)
  - Fill valve opens automatically
  - System resumes normal operation when level restored

## Hardware Components

### Main Controller
- **ESP32 DevKit** (ESP32-WROOM-32)

### Power Monitoring
- **PZEM-004T V3.0** - AC power meter
  - Monitors: Voltage, Current, Power (W), Energy (kWh), Frequency, Power Factor
  - Communication: Modbus RTU over UART
  - Used for pump protection and diagnostics

### Display
- **16x2 LCD Display with I2C Backpack** (PCF8574)
  - I2C Address: 0x27
  - Shows system status, active valve, pump state
  - Animated valve opening sequence

### Sensors
- **Float Switch** - Low water level detection
  - Normally open, closes when water level drops

### Actuators
- **3x Motorized Ball Valves** - Pool water routing
  - 10-second opening/closing time
  - Controls: Filter, Overflow Edge, Vacuum lines
- **1x Solenoid Valve** - Automatic pool filling
- **4x Relay Module** - Controls valves and pump
- **1x Pool Pump** - Main circulation pump

### User Interface
- **3x Push Buttons** - Manual valve control
  - Momentary switches with internal pull-up resistors

### Power Supply
- **5V Power Supply** - ESP32 and logic circuits
- **12V/24V Power Supply** - Motorized valves (check your valve specifications)
- **Surge Protection** - Recommended for pump circuit

## Pin Assignments

| Component | GPIO Pin | Description |
|-----------|----------|-------------|
| **Valves** |
| Valve 1 (Filter) | GPIO26 | Filter line valve relay |
| Valve 2 (Overflow) | GPIO27 | Overflow edge valve relay |
| Valve 3 (Vacuum) | GPIO13 | Vacuum line valve relay |
| Fill Valve | GPIO25 | Automatic fill valve relay |
| **Pump** |
| Pump Relay | GPIO18 | Main pump control |
| **Sensors** |
| Water Level | GPIO19 | Low level float switch (pulled up) |
| PZEM RX | GPIO16 | Power meter UART receive |
| PZEM TX | GPIO17 | Power meter UART transmit |
| **Buttons** |
| Button 1 | GPIO4 | Manual control valve 1 |
| Button 2 | GPIO5 | Manual control valve 2 |
| Button 3 | GPIO15 | Manual control valve 3 |
| **Display** |
| LCD SDA | GPIO21 | I2C data line |
| LCD SCL | GPIO22 | I2C clock line |

## Safety Features

1. **Valve Interlock System**
   - Hardware-level protection ensuring single valve operation
   - Prevents pump overload from multiple open paths

2. **Low Water Protection**
   - Immediate pump shutdown on low water detection
   - Automatic valve closure
   - Visual alert on LCD display

3. **Pump Health Monitoring**
   - Real-time power consumption tracking
   - Automatic shutdown on abnormal readings
   - Protects against:
     - Blockages (high power)
     - Dry running (low power)
     - Mechanical failures

4. **Fail-Safe Design**
   - All valves close on power loss
   - Pump defaults to OFF state
   - System status retained in memory

## Installation

1. Clone this repository:
   ```bash
   git clone https://github.com/yourusername/esphome-pool-control.git
   cd esphome-pool-control
   ```

2. Create your secrets file:
   ```bash
   cp secrets.yaml.example secrets.yaml
   ```

3. Edit `secrets.yaml` with your configuration:
   ```yaml
   wifi_ssid: "your_wifi_name"
   wifi_password: "your_wifi_password"
   fallback_ap_password: "fallback_password"
   encryption_key: "<generate with: openssl rand -base64 32>"
   ota_key: "your_ota_password"
   ```

4. Install ESPHome:
   ```bash
   pip install esphome
   ```

5. Compile and upload to your ESP32:
   ```bash
   esphome run configuration.yml
   ```

## Home Assistant Integration

The system automatically appears in Home Assistant with:

### Switches
- Valve 1 (Filter)
- Valve 2 (Overflow Edge)  
- Valve 3 (Vacuum)
- Water Pump

### Sensors
- Pump Current (A)
- Pump Voltage (V)
- Pump Power (W)
- Pump Energy (kWh)
- Pump Frequency (Hz)
- Pump Power Factor
- Water Level Status
- Valve Movement Timer

### Binary Sensors
- Low Water Level
- Valve Button States

## Wiring Diagram

```
ESP32 DevKit
├── Relay Module (4 channels)
│   ├── Valve 1 Relay
│   ├── Valve 2 Relay
│   ├── Valve 3 Relay
│   └── Pump Relay
├── PZEM-004T
│   ├── TX → GPIO17
│   ├── RX → GPIO16
│   └── AC Monitoring → Pump Circuit
├── LCD Display (I2C)
│   ├── SDA → GPIO21
│   ├── SCL → GPIO22
│   └── VCC → 5V
├── Push Buttons (3x)
│   └── Connected to GPIO with internal pullup
├── Float Switch
│   └── GPIO19 (normally open)
└── Fill Valve Relay → GPIO25
```

## Display Messages

| Display | Meaning |
|---------|---------|
| "Sistema Listo" | System ready, no active valves |
| "V1/V2/V3 Abriendo..." | Valve opening animation |
| "V1/V2/V3 Abierta" | Valve fully open |
| "Bomba: ON/OFF" | Pump status |
| "!NIVEL BAJO!" | Low water level alert |
| "Llenando..." | Filling pool |
| "Posible atasco!" | Pump blockage detected |

## Maintenance

- **Weekly**: Check pump power readings are within normal range
- **Monthly**: Test manual override buttons
- **Seasonally**: Clean float switch sensor
- **Annually**: Calibrate PZEM sensor if readings drift

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Pump won't start | Check water level sensor, ensure a valve is fully open |
| Valve won't open | Verify 12/24V supply, check relay connections |
| Power readings incorrect | Verify PZEM wiring, check CT orientation |
| Display not working | Check I2C connections, verify address (0x27) |
| WiFi connection issues | Check signal strength, verify credentials |

## Future Enhancements

- Temperature monitoring
- pH and chlorine sensors
- Scheduling system
- Water flow monitoring
- Energy usage tracking
- Weather-based adjustments

## License

This project is open source and available under the MIT License.

## Acknowledgments

Built with [ESPHome](https://esphome.io/) for seamless [Home Assistant](https://www.home-assistant.io/) integration.