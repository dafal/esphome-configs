# Shelly 1PM Gen4

ESPHome configuration for the Shelly 1PM Gen4 (ESP32-C6) with metering, protections, and add-on module support.

## Hardware
- MCU: ESP32-C6, 8MB flash, ESP-IDF
- Connectivity: Wi‑Fi 2.4GHz, Bluetooth LE

## GPIO Map

### Core Functions
- GPIO0: Status LED (inverted)
- GPIO1: Button (inverted, pullup)
- GPIO3: NTC temperature (ADC 12 dB)
- GPIO4: Relay output
- GPIO6: BL0942 TX
- GPIO7: BL0942 RX (pullup)
- GPIO10: Physical switch input

### Add-on Module
- GPIO12: Digital In (pullup)
- GPIO16: Data In
- GPIO9: Data Out
- GPIO17: Reserved (not configured - ADC not supported on this pin for ESP32-C6)

## Features
- Relay restore modes: Restore (default OFF/ON), Always OFF/ON
- Switch modes: Edge (Toggle), Follow, Detached, Multi-Click, **Smart Multi-Click**, Pulse Counter
- Multi-Click & Smart Multi-Click events: single / double / triple / long; in Smart Multi-Click, single click also toggles the relay (events still fire)
- Pulse Counter: counts press+release cycles; persistent; reset button in HA
- Power metering (BL0942): voltage, current, power, energy, frequency; overpower protection (default 2000 W, latch with auto-clear at 90%)
- Overheat protection with configurable threshold & hysteresis; NTC moving average filter
- **Add-on module support**: Digital In/Data In binary sensors, Data Out switch
- BLE proxy active; scanner modes (Disabled default, Passive continuous). Scanner mode restored after 1s on boot
- Wi‑Fi AP fallback SSID: `esphome`; mDNS enabled; static device names (no MAC suffix)
- Diagnostics: Wi‑Fi signal, uptime, overheating/overpower status, switch state (Detached), web server v3 on port 80
- Flash write interval: 5 minutes; Logger: DEBUG @ 115200

## Multi-Click timing
- Press ≤ ~1.0s; gaps ≤ ~0.8s; final release ≥ ~0.3s; long press ≥ ~0.8s

## Recent changes
- Initial Gen4 port from Mini Gen3
- Changed platform from ESP32-C3 to ESP32-C6
- Updated relay GPIO from GPIO5 to GPIO4
- GPIO17 Analog In commented out (ADC not supported on ESP32-C6 GPIO17)
- Added add-on module support (GPIO9, 12, 16, 17)
