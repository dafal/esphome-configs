# Shelly 1PM Mini Gen3

ESPHome configuration for the Shelly 1PM Mini Gen3 (ESP32-C3) with power metering and protections.

## GPIO Map
- GPIO0: Status LED (inverted)
- GPIO1: Button (input, pullup)
- GPIO3: NTC temperature (ADC 12 dB)
- GPIO5: Relay output
- GPIO6: BL0942 TX
- GPIO7: BL0942 RX (pullup)
- GPIO10: Physical switch input

## Features
- Relay with 4 restore modes (Restore default OFF/ON, Always OFF/ON)
- Switch modes: Edge, Follow, Detached, Multi-Click, Pulse Counter
- Multi-Click: 1/2/3 clicks, long press (timings ~1s press, gaps ~0.8s, final release ~0.3s)
- Pulse Counter: counts press+release cycles; publishes only in Pulse mode/boot/reset
- Power metering (BL0942): voltage, current, power, energy, frequency
- Protections: overheating (configurable threshold/hysteresis), overpower trip at configurable power limit (default 2000 W, with latch and auto-clear at 90%)
- BLE proxy enabled; scanner select (Disabled by default, Passive available), passive scan only
- Status: WiFi signal, uptime, switch state in Detached

## BLE Behavior
- Scanner modes: Disabled (default), Passive. Runtime active scan toggle is not supported.

## Notes
- Logging: DEBUG, baud 115200
- Web server v3 on port 80
- Flash write interval 5 minutes to reduce wear
