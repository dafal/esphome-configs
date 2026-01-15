# ESPHome Device Configurations

A collection of ESPHome configurations for various smart home devices, with a focus on Home Assistant integration.

## Available Configurations

### [Shelly 1 Mini Gen3](./shelly-1-mini-gen3/)
ESPHome configuration for Shelly 1 Mini Gen3 smart relay with ESP32-C3. Features configurable overheating protection, manual Bluetooth proxy control, and flexible relay restore modes - all configurable via Home Assistant without reflashing.

### [Shelly 1PM Mini Gen3](./shelly-1pm-mini-gen3/)
ESPHome configuration for Shelly 1PM Mini Gen3 with power metering (BL0942), overheating and overpower protection, configurable switch/restore modes, and BLE scanner select.

### [ProBreeze Dehumidifier 20L (PB-D-24-WF)](./probreeze-dehumidifier/)
ESPHome configuration for ProBreeze 20L PB-D-24-WF dehumidifier with Tuya MCU. Replace the original Tuya WBR3 WiFi board with an ESP32-C6 mini board for local control via Home Assistant.

## Using these configs as ESPHome packages
Add a minimal device YAML in your ESPHome dashboard and pull the package directly from this repo using `remote_package`.

Example (Shelly 1PM Mini Gen3):
```yaml
substitutions:
  device_name: shelly-1pm-mini-gen3
  friendly_name: Shelly 1PM Mini Gen3
  area: ""  # optional

packages:
  remote_package:
    url: https://github.com/dafal/esphome-configs
    ref: main
    files: [shelly-1pm-mini-gen3/shelly-1pm-mini-gen3.yaml]
    refresh: 0s

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
api:
  encryption:
    key: !secret api_encryption_key
ota:
  - platform: esphome
    password: !secret ota_password
```
Swap the file path for other devices (e.g., `shelly-1-mini-gen3/...`). Project name/version are already defined in the packages; no need to override. Keep your `secrets.yaml` local with `wifi_ssid`, `wifi_password`, `api_encryption_key`, `ota_password`, `fallback_password`.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Disclaimer

Modifying devices by replacing WiFi modules may void warranties and carries risk of damage or malfunction. Proceed at your own risk. Always ensure proper electrical safety when working with mains-powered devices.

---

**Author:** dafal  
**Year:** 2026
