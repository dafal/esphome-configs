# ProBreeze PB-D-24-WF 20L Dehumidifier - ESPHome Configuration

ESPHome configuration for the ProBreeze PB-D-24-WF 20L dehumidifier with replaced WiFi module. This configuration replaces the original Tuya WBR3 WiFi board with an ESP32-C6 mini board for full local control via Home Assistant.

## Overview

This configuration enables complete local control of the ProBreeze dehumidifier without any cloud dependency. All device functions are exposed to Home Assistant, including sensors, controls, and fault detection.

### Features

- **Full Local Control** - No cloud dependency, all communication stays local
- **Power Management** - On/Off control and auto-shutoff timer (1-24 hours)
- **Operating Modes**
  - Auto (maintains target humidity)
  - Dry (continuous operation)
  - Night (quiet mode)
- **Climate Control**
  - Target humidity setting (30-80% in 5% steps)
  - Current humidity monitoring
  - Temperature monitoring
- **Fan Control** - Low/High speed settings
- **Additional Features**
  - Ionizer control
  - Child lock
  - Temperature unit selection (°C/°F)
- **Diagnostics & Monitoring**
  - Tank full detection
  - Filter life tracking and reset
  - Multiple fault sensors (temperature, humidity, coil, motor errors)
  - WiFi signal strength
  - Device uptime
- **Remote Management** - OTA updates and remote restart

## Hardware Requirements

### Components Needed

1. **ESP32-C6 Mini Development Board**
   - Board: ESP32-C6-DevKitC-1 (or compatible)
   - Any ESP32-C6 board with accessible GPIO21 and GPIO22
   
2. **Tools**
   - Screwdriver set (for opening the dehumidifier)
   - Soldering iron and solder
   - Wire strippers
   - Multimeter (recommended)
   - USB-C cable (for initial programming)

3. **Optional but Recommended**
   - Heat shrink tubing
   - Dupont wires or similar for connections
   - Hot glue or mounting tape

### Compatible Devices

This configuration has been tested with:
- ProBreeze PB-D-24-WF (20L capacity)

**Potentially compatible models** (untested):
- Other ProBreeze dehumidifiers with Tuya WiFi modules
- Generic Tuya-based dehumidifiers with similar Tuya datapoints

⚠️ **Always verify your device uses the same Tuya protocol and datapoints before proceeding!**

## Wiring Diagram

The ESP32-C6 needs to communicate with the Tuya MCU via UART:

```
┌─────────────────────┐         ┌──────────────────┐
│   ESP32-C6 Board    │         │    Tuya MCU      │
│                     │         │  (on main board) │
├─────────────────────┤         ├──────────────────┤
│                     │         │                  │
│  GPIO21 (RX) ───────┼────────►│ TX               │
│                     │         │                  │
│  GPIO22 (TX) ◄──────┼─────────│ RX               │
│                     │         │                  │
│  GND ───────────────┼─────────│ GND              │
│                     │         │                  │
│  5V ────────────────┼─────────│ 5V               │
│                     │         │.                 │
└─────────────────────┘         └──────────────────┘
```

### Pin Connections

| ESP32-C6 Pin | Tuya MCU Pin | Function |
|--------------|--------------|----------|
| GPIO21       | TX           | UART RX (ESP receives data) |
| GPIO22       | RX           | UART TX (ESP sends data) |
| GND          | GND          | Ground |
| 5V           | VCC          | Power supply |


### Important Safety Notes

⚠️ **WARNING:**
- Disconnect the dehumidifier from mains power before opening
- Work in a dry, well-lit environment
- Ensure proper insulation of all connections
- Double-check all wiring before powering on
- Do not operate with the case open while connected to mains power
- This modification will void your warranty
- Proceed at your own risk

## Installation Instructions

### Step 1: Prepare Your Environment

1. **Install ESPHome**
   ```bash
   # If using Home Assistant, ESPHome is available as an add-on
   # Or install via pip:
   pip install esphome
   ```

2. **Clone this repository**
   ```bash
   git clone https://github.com/dafal/esphome-configs.git
   cd esphome-configs
   ```

3. **Set up secrets**
   ```bash
   cp secrets.yaml.example secrets.yaml
   # Edit secrets.yaml with your WiFi credentials and keys
   ```

4. **Generate API encryption key** (if you don't have one)
   ```bash
   openssl rand -base64 32
   # Add the output to secrets.yaml under api_encryption_key
   ```

### Step 2: Hardware Replacement

1. **Open the dehumidifier**
   - Unplug from power
   - Remove screws from back/bottom panel
   - Carefully access the WiFi module area

2. **Locate the Tuya WBR3 module**
   - It's typically near the control panel
   - Connected to the main MCU board via 4 wires (TX, RX, VCC, GND)

3. **Document original connections**
   - Take photos before disconnecting anything
   - Note wire colors and positions

4. **Remove Tuya WBR3 module**
   - Desolder or carefully disconnect the 4 wires
   - Remove the old WiFi module

5. **Install ESP32-C6 board**
   - Solder or connect wires according to the wiring diagram above
   - Ensure solid connections (soldering recommended)
   - Insulate exposed connections with heat shrink tubing
   - Mount the ESP32 board securely (hot glue or tape)

6. **Double-check all connections**
   - Verify TX/RX are not swapped
   - Check for shorts with multimeter
   - Ensure proper ground connection

### Step 3: Initial Flash

1. **Connect ESP32-C6 to computer via USB**
   - Before closing the dehumidifier case
   - This is for the initial flash only

2. **Compile the firmware**
   ```bash
   cd esphome-configs/probreeze-dehumidifier
   esphome compile probreeze-dehumidifier.yaml
   ```

3. **Flash using ESPHome Web Flasher** (recommended)
   - Visit: https://web.esphome.io/
   - Click "Install"
   - Select the compiled `.bin` file
   - Choose the connected serial port
   - Click "Install"

   OR use command line:
   ```bash
   esphome run probreeze-dehumidifier.yaml
   # Select the serial port when prompted
   ```

4. **Verify connection**
   - Check the ESPHome logs for successful boot
   - Verify WiFi connection
   - Confirm Tuya MCU communication

### Step 4: Configuration & Testing

1. **Add to Home Assistant**
   - Go to Settings → Devices & Services → ESPHome
   - Your device should appear automatically
   - Click "Configure" and enter your API encryption key

2. **Test basic functions**
   - Turn the dehumidifier on/off
   - Check sensor readings (humidity, temperature)
   - Test mode changes and fan speed
   - Verify all controls work

3. **Close the case**
   - Once everything works, carefully close the dehumidifier
   - Ensure no wires are pinched
   - Secure all screws

### Step 5: Future Updates (OTA)

After the initial flash, you can update wirelessly:

```bash
cd esphome-configs/probreeze-dehumidifier
esphome run probreeze-dehumidifier.yaml
# Select "Over The Air" when prompted
```

## Configuration Customization

### Change Device Name

Edit `probreeze-dehumidifier.yaml`:

```yaml
substitutions:
  device_name: my-dehumidifier-name
  friendly_name: My Custom Name
```

### Adjust Update Intervals

WiFi signal sensor (default: 60s):
```yaml
sensor:
  - platform: wifi_signal
    name: "WiFi Signal"
    update_interval: 30s  # Change to your preference
```

### Add Static IP (Optional)

Add to the `wifi:` section:
```yaml
wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  manual_ip:
    static_ip: 192.168.1.100
    gateway: 192.168.1.1
    subnet: 255.255.255.0
```

## Exposed Entities in Home Assistant

### Switches
- **Power** - Main power on/off
- **Ionizer** - Air ionizer function
- **Child Lock** - Prevent physical button presses
- **Filter Reset** - Reset filter life counter

### Controls
- **Target Humidity** (30-80%, 5% steps)
- **Fan Speed** (Low/High)
- **Mode** (Auto/Dry/Night)
- **Timer** (Cancel, 1-24 hours)
- **Temperature Unit** (Celsius/Fahrenheit)

### Sensors
- **Current Humidity** (%)
- **Temperature** (°C or °F)
- **Filter Life** (%)
- **WiFi Signal** (dBm)
- **Uptime** (seconds)

### Binary Sensors (Faults/Warnings)
- **Tank Full** - Water tank needs emptying
- **Cleaning Required** - Filter maintenance needed
- **Error E1** - General error code
- **Temperature Too Low** (<5°C)
- **Temperature Too High** (>38°C)
- **Humidity Too Low** (<25%)
- **Coil Error** - Evaporator coil sensor fault
- **Motor Error** - Fan motor fault

### Diagnostic Entities
- **Fault Bitmap** (raw) - All fault states as a single number
- **Client ID** - Tuya device identifier
- **Equipment Type** - Device type code

### Buttons
- **Restart** - Remotely restart the ESP32

## Troubleshooting

### No Communication with Tuya MCU

**Symptoms:** Entities show as unavailable, logs show no Tuya communication

**Solutions:**
1. Verify TX/RX are not swapped
2. Check baud rate is 9600
3. Ensure proper ground connection
4. Verify voltage levels (3.3V vs 5V)
5. Check solder joints for cold solder or shorts

### WiFi Connection Issues

**Symptoms:** Device not connecting to WiFi

**Solutions:**
1. Check WiFi credentials in `secrets.yaml`
2. Verify 2.4GHz WiFi network (ESP32 doesn't support 5GHz)
3. Check WiFi signal strength in device location
4. Try the fallback AP (SSID: "ProBreeze Dehumidifier Fallback")

### Enable Debug Logging

To get more detailed logs, edit the configuration:

```yaml
logger:
  level: DEBUG
  logs:
    tuya: VERBOSE
```

## Datapoints Reference

See [DATAPOINTS.md](./DATAPOINTS.md) for complete Tuya datapoint documentation.


## Resources

- [ESPHome Documentation](https://esphome.io/)
- [ESPHome Tuya Component](https://esphome.io/components/tuya.html)
- [Home Assistant ESPHome Integration](https://www.home-assistant.io/integrations/esphome/)
- [ESP32-C6 Technical Reference](https://www.espressif.com/en/products/socs/esp32-c6)

## Support & Contributing

For issues, questions, or contributions, please visit the main repository:
- Repository: https://github.com/dafal/esphome-configs
- Issues: https://github.com/dafal/esphome-configs/issues

## License

This configuration is licensed under the MIT License - see the [LICENSE](../LICENSE) file for details.

## Disclaimer

This modification involves replacing electronic components in a mains-powered appliance. Improper modification can result in:
- Fire hazard
- Electric shock
- Device malfunction or damage
- Voided warranty
- Property damage

**Proceed at your own risk.** The author assumes no liability for any damage or injury resulting from this modification. Always ensure proper electrical safety and never operate modified devices unattended until thoroughly tested.
