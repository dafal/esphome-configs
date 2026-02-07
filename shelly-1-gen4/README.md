# Shelly 1 Gen4

ESPHome configuration for the Shelly 1 Gen4 (ESP32-C6).

## Hardware
- MCU: ESP32-C6, ESP-IDF
- Connectivity: Wi‑Fi 2.4GHz, Bluetooth LE

## GPIO Map
- GPIO0: Status LED (inverted) - ⚠️ Not working with current ESPHome (works in stock firmware)
- GPIO1: Button (inverted, pullup)
- GPIO3: NTC temperature sensor (ADC 11 dB)
- GPIO5: Relay output ✅
- GPIO10: Physical switch input

## Features
- Relay restore modes: Restore (default OFF/ON), Always OFF/ON
- Switch modes: Edge (Toggle), Follow, Detached, Multi-Click, **Smart Multi-Click**, Pulse Counter
- Multi-Click & Smart Multi-Click events: single / double / triple / long; in Smart Multi-Click, single click also toggles the relay (events still fire)
- Pulse Counter: counts press+release cycles; persistent; reset button in HA
- Overheat protection with configurable threshold & hysteresis; NTC moving average filter
- BLE proxy active; scanner modes (Disabled default, Passive continuous). Scanner mode restored after 1s on boot
- Wi‑Fi AP fallback SSID: `esphome`; mDNS enabled; static device names (no MAC suffix)
- Diagnostics: Wi‑Fi signal, uptime, overheating status, switch state (Detached), web server v3 on port 80
- Flash write interval: 5 minutes; Logger: DEBUG @ 115200

## Multi-Click timing
- Press ≤ ~1.0s; gaps ≤ ~0.8s; final release ≥ ~0.3s; long press ≥ ~0.8s

## Home Assistant Events
Multi-click events are published as Home Assistant events (not entity events), preventing spurious triggers on boot/WiFi reconnect.

**Event type:** `esphome.button_click`

**Event data:**
- `device`: Device name (e.g., `shelly-1-gen4`)
- `friendly_name`: Device friendly name
- `click_type`: `single`, `double`, `triple`, or `long_press`
- `message`: Human-readable description

**Example automation trigger:**
```yaml
trigger:
  - platform: event
    event_type: esphome.button_click
    event_data:
      device: shelly-1-gen4  # Use your actual device name
      click_type: single
```

## Differences from Shelly 1 Mini Gen3
- **MCU**: ESP32-C6 (Gen4) vs ESP32-C3 (Mini Gen3)
- **Relay GPIO**: GPIO5 (Gen4) vs GPIO7 (Mini Gen3)
- **ADC attenuation**: 11dB (Gen4) vs 12dB (Mini Gen3)

## GPIO Pattern Notes
The Shelly 1 Gen4 uses **GPIO5** for relay output (same as 1PM Mini Gen3).
The Shelly 1 Mini Gen3 uses **GPIO7** for relay output.
Both Shelly 1PM models use different GPIOs (GPIO5 for Mini Gen3, GPIO4 for Gen4) because GPIO6/GPIO7 are reserved for UART communication with the BL0942 power meter chip.

## Differences from Shelly 1PM Gen4
- **No power metering**: This device does not have the BL0942 power meter chip
- **No UART**: No TX/RX pins used for power metering communication
- **No power sensors**: Voltage, current, power, energy, frequency sensors not available
- **No overpower protection**: No power limit configuration or overpower detection
- **No add-on module GPIOs**: GPIO9, 12, 16, 17 not exposed (can be added if needed)

## Known Issues
- **Status LED not working**: The GPIO0 status LED does not work with current ESPHome (same issue on 1PM Gen4). The LED works correctly with Shelly stock firmware. This appears to be an ESPHome framework compatibility issue with ESP32-C6. The configuration is correct but the LED remains off.

## Recent changes
- **v1.2.0**: Initial release based on Shelly 1 Mini Gen3 v1.2.0 with ESP32-C6 hardware from Shelly 1PM Gen4
- Multi-click events using `esphome.button_click` Home Assistant events (prevents boot/reconnect event replay)
- **Smart Multi-Click** mode (single click toggles relay; all click events still emitted)
- BLE Passive scan made continuous and restored after boot
- Static device names (no MAC suffix); fallback AP SSID set to `esphome`; mDNS explicitly enabled
- **Relay GPIO confirmed**: GPIO5 (tested on hardware)
