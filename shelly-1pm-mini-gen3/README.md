# Shelly 1PM Mini Gen3

ESPHome configuration for the Shelly 1PM Mini Gen3 (ESP32-C3) with metering and protections.

## Hardware
- MCU: ESP32-C3, 8MB flash, ESP-IDF
- Connectivity: Wi‑Fi 2.4GHz, Bluetooth LE

## GPIO Map
- GPIO0: Status LED (inverted)
- GPIO1: Button (inverted, pullup)
- GPIO3: NTC temperature (ADC 12 dB)
- GPIO5: Relay output
- GPIO6: BL0942 TX
- GPIO7: BL0942 RX (pullup)
- GPIO10: Physical switch input

## Features
- Relay restore modes: Restore (default OFF/ON), Always OFF/ON
- Switch modes: Edge (Toggle), Follow, Detached, Multi-Click, **Smart Multi-Click**, Pulse Counter
- Multi-Click & Smart Multi-Click events: single / double / triple / long; in Smart Multi-Click, single click also toggles the relay (events still fire)
- Pulse Counter: counts press+release cycles; persistent; reset button in HA
- Power metering (BL0942): voltage, current, power, energy, frequency; overpower protection (default 2000 W, latch with auto-clear at 90%)
- Overheat protection with configurable threshold & hysteresis; NTC moving average filter
- BLE proxy active; scanner modes (Disabled default, Passive continuous). Scanner mode restored after 1s on boot
- Wi‑Fi AP fallback SSID: `esphome`; mDNS enabled; static device names (no MAC suffix)
- Diagnostics: Wi‑Fi signal, uptime, overheating/overpower status, switch state (Detached), web server v3 on port 80
- Flash write interval: 5 minutes; Logger: DEBUG @ 115200

## Multi-Click timing (unchanged)
- Press ≤ ~1.0s; gaps ≤ ~0.8s; final release ≥ ~0.3s; long press ≥ ~0.8s

## Home Assistant Events
Multi-click events are published as Home Assistant events (not entity events), preventing spurious triggers on boot/WiFi reconnect.

**Event type:** `esphome.button_click`

**Event data:**
- `device`: Device name (e.g., `shelly-1pm-mini-gen3`)
- `friendly_name`: Device friendly name
- `click_type`: `single`, `double`, `triple`, or `long_press`
- `message`: Human-readable description

**Example automation trigger:**
```yaml
trigger:
  - platform: event
    event_type: esphome.button_click
    event_data:
      device: shelly-1pm-mini-gen3  # Use your actual device name
      click_type: single
```

## Recent changes
- **v1.2.0**: Multi-click events changed to `esphome.button_click` Home Assistant events (prevents boot/reconnect event replay)
- Added **Smart Multi-Click** (single click toggles relay; all click events still emitted)
- BLE Passive scan made continuous and restored after boot
- Removed MAC suffix from names; fallback AP SSID set to `esphome`; mDNS explicitly enabled
