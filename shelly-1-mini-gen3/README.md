# Shelly 1 Mini Gen3

ESPHome configuration for the Shelly 1 Mini Gen3 smart relay with ESP32-C3.

## Hardware Specifications

- **Microcontroller**: ESP32-C3 (single-core, RISC-V)
- **Flash Size**: 8MB
- **Connectivity**: WiFi 2.4GHz, Bluetooth 5.0 (LE)
- **Framework**: ESP-IDF

## GPIO Pin Assignments

| GPIO | Function              | Direction | Configuration              |
|------|-----------------------|-----------|----------------------------|
| 0    | Status LED            | Output    | Inverted (active low)      |
| 1    | Physical Button       | Input     | Inverted, internal pullup  |
| 3    | Temperature Sensor    | Input     | ADC, 12dB attenuation      |
| 7    | Relay Output          | Output    | Direct GPIO control        |
| 10   | Physical Switch       | Input     | Standard GPIO              |

## Temperature Sensor

- **Type**: NTC Thermistor
- **Configuration**: 10kΩ resistor, downstream configuration
- **Calibration**: β = 3350, Reference 10kΩ @ 25°C
- **Protection**: Overheating threshold with hysteresis

## Features

### Core Functionality
- Single relay control with physical switch and button
- Overheating protection with configurable threshold (default: 75°C)
- Temperature monitoring via NTC sensor
- Status LED indicator
- Multiple switch operating modes (Shelly firmware compatible)
- Pulse counter for usage tracking

### Switch Modes (Configurable via Home Assistant)
The physical switch (GPIO 10) supports 5 modes:

1. **Edge (Toggle)** – Toggle relay on each press
2. **Follow** – Relay follows the switch (press = ON, release = OFF)
3. **Detached** – Switch is independent; state/events exposed to HA
4. **Multi-Click** – Detects 1/2/3 clicks and long press
5. **Pulse Counter** – Counts press+release cycles

### Multi-Click Events (in Multi-Click mode)
- **Single**: Press ≤ 1.0s, release ≥ 0.3s
- **Double**: Two presses (each ≤ 1.0s) with gaps ≤ 0.8s, final release ≥ 0.3s
- **Triple**: Three presses (each ≤ 1.0s) with gaps ≤ 0.8s, final release ≥ 0.3s
- **Long**: Press ≥ 0.8s

These events appear in Home Assistant and can trigger automations.

### Pulse Counter Mode
- Counts complete switch press/release cycles (press + release = 1 pulse)
- Persistent across reboots (tracks long-term usage)
- Manual reset via button in Home Assistant
- Perfect for usage statistics and maintenance scheduling

**Use cases**:
- Track switch usage frequency
- Monitor device interaction patterns
- Schedule maintenance based on usage count
- Usage analytics and reporting

### Configurable Settings (via Home Assistant)
- **Switch Mode**: 5 operating modes (see above)
- **Overheating Threshold**: 60-90°C (default: 75°C)
- **Temperature Hysteresis**: 1-10°C (default: 5°C)
- **Relay Restore Mode**: 4 modes selectable at runtime
  - Restore (default OFF)
  - Restore (default ON)
  - Always OFF
  - Always ON
- **Bluetooth Scanner**: Mode select (Disabled default, Passive available)
- **Pulse Counter Reset**: Manual reset button

### Monitoring & Diagnostics
- WiFi signal strength
- Device uptime
- Temperature monitoring with moving average filter
- Overheating status
- Switch state (in Detached mode)

### Bluetooth Scanner / Proxy
- BLE proxy enabled; scanning controlled by a mode select (Disabled by default; Passive available)
- Passive scan only (runtime active toggle not supported)

## Safety Features

- **Overheating Protection**: Automatically turns off relay when temperature exceeds threshold
- **Hysteresis**: Prevents rapid on/off cycling near threshold temperature
- **Boot Protection**: Allows relay activation on boot even if temperature sensor is initializing
- **Flash Protection**: 5-minute write interval to reduce flash wear

## Configuration

See the main [secrets.yaml.example](../secrets.yaml.example) for required secrets:
- `wifi_ssid`: Your WiFi network name
- `wifi_password`: Your WiFi password
- `api_encryption_key`: Home Assistant API encryption key
- `ota_password`: Over-the-air update password
- `fallback_password`: Fallback access point password

## Web Interface

The device provides a web interface on port 80 (version 3) for diagnostics and monitoring.

## Usage Examples

### Switch Mode Examples

#### Detached Mode - Scene Control
Use the physical switch to trigger Home Assistant scenes while controlling the relay independently:

```yaml
# Home Assistant automation
automation:
  - alias: "Switch ON - Morning Scene"
    trigger:
      platform: state
      entity_id: binary_sensor.shelly_switch_state
      to: "on"
    action:
      - scene: scene.morning
```

#### Multi-Click - Advanced Control
Use different click patterns for different actions:

```yaml
# Single click - Toggle relay
automation:
  - alias: "Single Click"
    trigger:
      platform: event
      event_type: esphome.switch_single_click
    action:
      - service: switch.toggle
        target:
          entity_id: switch.shelly_relay

# Double click - All lights
  - alias: "Double Click"
    trigger:
      platform: event
      event_type: esphome.switch_double_click
    action:
      - service: light.toggle
        target:
          area_id: living_room

# Long press - All off
  - alias: "Long Press"
    trigger:
      platform: event
      event_type: esphome.switch_long_press
    action:
      - service: homeassistant.turn_off
        target:
          entity_id: all
```

#### Pulse Counter - Usage Tracking
Track switch usage for maintenance scheduling:

1. Set Switch Mode: "Pulse Counter"
2. Each complete press/release = +1 count
3. View count in Home Assistant: `sensor.shelly_pulse_counter`
4. Reset counter: Press "Reset Pulse Counter" button

**Home Assistant automation example**:
```yaml
automation:
  - alias: "High usage alert"
    trigger:
      platform: numeric_state
      entity_id: sensor.shelly_pulse_counter
      above: 1000
    action:
      service: notify.mobile_app
      data:
        message: "Switch used 1000+ times, consider maintenance"
```

### Multi-Click Timing

```
Single:  [PRESS (≤1s)]--(≥0.3s)
Double:  [P(≤1s)]_[P(≤1s)]--(≥0.3s)
Triple:  [P(≤1s)]_[P(≤1s)]_[P(≤1s)]--(≥0.3s)
Long:    [HOLD ≥0.8s]
```

- Press: up to ~1.0 s
- Gaps between presses: up to ~0.8 s
- Final release/confirm: at least ~0.3 s

## Notes

- **ESP32-C3 Single-Core**: WiFi and Bluetooth share the same core. Bluetooth proxy is configured for manual control to ensure WiFi stability.
- **Temperature Sensor**: Moving average filter (5 samples) smooths temperature readings for more stable operation.
- **Flash Write Interval**: Set to 5 minutes to reduce flash wear and improve ESP32-C3 stability.
- **Switch Modes**: 5 modes, all with overheat protection
- **Event Entities**: Require ESPHome 2023.4.0+ and Home Assistant 2023.4.0+

## Recent Improvements

### Pulse Counter Implementation (January 2026)
Replaced auto-off timer with proper pulse counter functionality:

- **Removed**: Auto-off timer/"Pulse Mode" (use Home Assistant automations for timed control)
- **Added**: Pulse Counter mode - counts complete press/release cycles
- **Tracking**: Persistent counter across reboots for long-term usage statistics
- **Reset**: Manual reset button in Home Assistant
- **Use cases**: Usage tracking, maintenance scheduling, interaction analytics

### Configuration Fixes (January 2026)
Three important fixes were applied to improve reliability and correctness:

1. **Full Relay Restore Mode Implementation**
   - All 4 restore modes now work correctly (previously only "Restore default OFF" worked)
   - "Restore (default ON)" now properly restores ON state with temperature safety
   - "Always ON" now includes temperature check for boot safety
   - Uses global variable to track and restore last relay state accurately

2. **Binary Sensor Return Value Correction**
   - Switch State sensor now returns explicit `false` instead of ambiguous `{}` when not in Detached mode
   - Improves compatibility across ESPHome versions

3. **Overheating Latch Refactoring**
   - Removed self-reference in overheating detection logic
   - Uses dedicated global variable for cleaner state management
   - More robust and maintainable implementation

**Impact**: These fixes ensure the relay restore modes work as documented, improve boot safety, and enhance code quality without adding new features.
