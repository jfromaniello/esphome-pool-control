# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Building and Uploading
```bash
# Validate configuration
esphome validate configuration.yml

# Compile firmware
esphome compile configuration.yml

# Upload firmware to device
esphome upload configuration.yml

# View device logs
esphome logs configuration.yml

# Clean build files
esphome clean configuration.yml
```

### First-time Setup
1. Copy `secrets.yaml.example` to `secrets.yaml`
2. Fill in your WiFi credentials and generate an encryption key:
   ```bash
   openssl rand -base64 32  # For encryption_key
   ```

## Architecture Overview

This is an ESPHome-based pool control system running on ESP32 that manages:

### Core Components
- **3 Pool Valves**: Motorized valves with 10-second opening time, controlled via GPIO pins with manual button override
- **Water Pump**: Protected by power monitoring (PZEM sensor) that detects abnormal power consumption (outside 810-1050W range) to prevent damage
- **Automatic Fill System**: Uses a low water level sensor to automatically fill the pool when needed
- **LCD Display**: 16x2 I2C display showing system status and valve animations

### Package Architecture
The system uses ESPHome's package system for modularity:
- `configuration.yml`: Main entry point, defines global scripts and includes all packages
- `packages/valve.pkg/main.yaml`: Reusable valve control logic (instantiated 3 times with different pins)
- `packages/pump.pkg/main.yaml`: Pump control with power monitoring and safety shutoff
- `packages/fill.pkg/main.yaml`: Automatic water level management
- `packages/display.pkg/main.yaml`: LCD display logic

### Safety Features
1. **Valve Interlock**: Only one valve can be open at a time (enforced by `close_valves` script)
2. **Low Water Protection**: Pump automatically stops if water level is low
3. **Power Monitoring**: Pump shuts off if power consumption is abnormal (3 consecutive out-of-range readings)
4. **Startup Grace Period**: 40-second delay before power monitoring to allow pump startup surge

### Key Global Variables
- `process_cancelled`: Used to cancel valve opening sequences
- `pump_issue_detected`: Flags potential pump blockage
- `valve{1,2,3}_moving`: Track valve movement state for display animation

### GPIO Pin Assignments
- Valves: GPIO 26/27/13 (control), GPIO 4/5/15 (buttons)
- Pump: GPIO 18
- Fill valve: GPIO 25
- Water level sensor: GPIO 19
- Display: GPIO 21/22 (I2C)
- PZEM: GPIO 16/17 (UART)