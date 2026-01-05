# Tuya Datapoints Reference - ProBreeze PB-D-24-WF

Complete reference of all Tuya MCU datapoints used in the ProBreeze PB-D-24-WF 20L dehumidifier.

## Overview

This document details the Tuya protocol datapoints discovered through reverse engineering the ProBreeze dehumidifier. These datapoints enable communication between the ESP32 and the original Tuya MCU controlling the device.

**Datapoint Sources:**
- Community contributions: [Home Assistant Core Issue #155364](https://github.com/home-assistant/core/issues/155364)
- Community forum: [ProBreeze Omnidry 12L Discussion](https://community.home-assistant.io/t/probreeze-omnidry-12l-pb-d-18w-wf-with-localtuya/812807)
- Tuya Developer Console observations

## Datapoint Summary Table

| DP ID | Type    | Function | Access | Value Range | Unit | Notes |
|-------|---------|----------|--------|-------------|------|-------|
| 1     | Boolean | Power | R/W | true/false | - | Main power on/off |
| 2     | Integer | Target Humidity | R/W | 30-80 | % | Steps of 5% |
| 4     | Enum    | Fan Speed | R/W | 0-1 | - | 0=Low, 1=High |
| 5     | Enum    | Operating Mode | R/W | 0-2 | - | See mode table below |
| 6     | Integer | Current Humidity | R | 0-100 | % | Sensor reading |
| 7     | Integer | Temperature | R | -40 to 125 | °C | Sensor reading |
| 10    | Boolean | Ionizer | R/W | true/false | - | Air ionizer function |
| 16    | Boolean | Child Lock | R/W | true/false | - | Lock physical buttons |
| 17    | Enum    | Timer | R ? | 0-24 | - | Auto-shutoff timer |
| 19    | Integer | Fault Bitmap | R | 0-255 | - | See fault table below |
| 20    | Boolean | Filter Reset | W | true | - | Reset filter counter |
| 23    | Integer | Filter Life | R | 0-100 | % | Remaining filter life |
| 24    | Enum    | Temperature Unit | R/W | 0-1 | - | 0=Celsius, 1=Fahrenheit |
| 102   | Integer | Client ID | R | varies | - | Tuya device identifier |
| 104   | Integer | Equipment Type | R | varies | - | Device type code |

**Access Key:**
- R = Read only (sensor/status)
- W = Write only (command)
- R/W = Read and write (control with state feedback)

## Detailed Datapoint Descriptions

### DP1 - Power Switch

**Type:** Boolean  
**Access:** Read/Write  
**Function:** Main power control

- `true` (1) = Device powered on
- `false` (0) = Device powered off (standby mode)

**Notes:** When powered off, the MCU enters standby mode but remains responsive to commands.

---

### DP2 - Target Humidity Setpoint

**Type:** Integer  
**Access:** Read/Write  
**Function:** Desired humidity level for auto mode

- **Range:** 30 to 80
- **Unit:** Percentage (%)
- **Step:** 5% increments
- **Valid values:** 30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80

**Behavior:** In Auto mode, the dehumidifier will run until ambient humidity reaches this setpoint, then cycle on/off to maintain it.

---

### DP4 - Fan Speed

**Type:** Enum  
**Access:** Read/Write  
**Function:** Fan speed selection

| Value | Description | Notes |
|-------|-------------|-------|
| 0 | Low | Quieter, slower dehumidification |
| 1 | High | Louder, faster dehumidification |

**Notes:** Fan speed affects both noise level and dehumidification rate.

---

### DP5 - Operating Mode

**Type:** Enum  
**Access:** Read/Write  
**Function:** Operating mode selection

| Value | Mode | Description |
|-------|------|-------------|
| 0 | Auto (Target) | Maintains target humidity (DP2), cycles on/off |
| 1 | Dry (Continuous) | Runs continuously regardless of humidity |
| 2 | Night | Quiet mode with reduced fan speed |

**Mode Behaviors:**
- **Auto:** Respects target humidity setting, most energy-efficient
- **Dry:** Maximum dehumidification, ignores target humidity
- **Night:** Optimized for bedroom use, quieter operation

---

### DP6 - Current Humidity

**Type:** Integer  
**Access:** Read only  
**Function:** Measured relative humidity

- **Range:** 0 to 100
- **Unit:** Percentage (%)
- **Update frequency:** Approximately every 5-10 seconds

**Notes:** This is the actual measured humidity from the device's sensor.

---

### DP7 - Temperature Sensor

**Type:** Integer  
**Access:** Read only  
**Function:** Measured ambient temperature

- **Unit:** °C or °F (depending on DP24 setting)

**Notes:** Used for frost protection and diagnostic purposes. Can be displayed in Celsius or Fahrenheit based on DP24 setting.

---

### DP10 - Ionizer

**Type:** Boolean  
**Access:** Read/Write  
**Function:** Air ionizer control

- `true` (1) = Ionizer active
- `false` (0) = Ionizer off

**Notes:** The ionizer function releases negative ions claimed to improve air quality. Effect on dehumidification performance is negligible.

---

### DP16 - Child Lock

**Type:** Boolean  
**Access:** Read/Write  
**Function:** Physical button lock

- `true` (1) = Buttons locked
- `false` (0) = Buttons unlocked

**Notes:** When enabled, physical buttons on the device are disabled. Can still be controlled via ESPHome/Home Assistant.

---

### DP17 - Timer (Auto-On/Off)

**Type:** Enum  
**Access:** Read/Write (possibly read-only in practice)  
**Function:** Countdown timer for automatic power on/off

| Value | Time | Value | Time | Value | Time | Value | Time |
|-------|------|-------|------|-------|------|-------|------|
| 0 | Cancel/Off | 7 | 7 hours | 13 | 13 hours | 19 | 19 hours |
| 1 | 1 hour | 8 | 8 hours | 14 | 14 hours | 20 | 20 hours |
| 2 | 2 hours | 9 | 9 hours | 15 | 15 hours | 21 | 21 hours |
| 3 | 3 hours | 10 | 10 hours | 16 | 16 hours | 22 | 22 hours |
| 4 | 4 hours | 11 | 11 hours | 17 | 17 hours | 23 | 23 hours |
| 5 | 5 hours | 12 | 12 hours | 18 | 18 hours | 24 | 24 hours |
| 6 | 6 hours | - | - | - | - | - | - |

**Behavior:** According to the device manual, this timer will automatically power off the device after the specified duration if currently on, or power on the device if currently off. Set to 0 to cancel an active timer.

**⚠️ Note:** In practice, this datapoint may behave as read-only and not accept changes via Home Assistant. Further testing is needed to confirm if timer values can be set programmatically or only via the physical buttons. Please report your findings!

---

### DP19 - Fault Bitmap

**Type:** Integer (Bitmap)  
**Access:** Read only  
**Function:** Device fault and warning states

**Bitmap Structure:**

| Bit | Hex | Decimal | Fault Description |
|-----|-----|---------|-------------------|
| 0 | 0x01 | 1 | Tank Full |
| 1 | 0x02 | 2 | Cleaning Required |
| 2 | 0x04 | 4 | Error E1 (General Error) |
| 3 | 0x08 | 8 | Temperature Too Low (<5°C) |
| 4 | 0x10 | 16 | Temperature Too High (>38°C) |
| 5 | 0x20 | 32 | Humidity Too Low (<25%) |
| 6 | 0x40 | 64 | Coil Sensor Error |
| 7 | 0x80 | 128 | Motor Error |

**Reading the Bitmap:**

Multiple faults can be active simultaneously. The value is the sum of all active fault bits.

**Examples:**
- `0` = No faults
- `1` = Tank full only
- `3` = Tank full (1) + Cleaning required (2)
- `9` = Tank full (1) + Temperature too low (8)
- `255` = All faults active (catastrophic failure)

**Individual Fault Descriptions:**

- **Bit 0 - Tank Full:** Water collection tank is full and must be emptied. Device will not operate until tank is replaced.
- **Bit 1 - Cleaning Required:** Filter maintenance reminder based on runtime hours.
- **Bit 2 - Error E1:** Generic error code. Check device display for details. May require power cycle.
- **Bit 3 - Temperature Too Low:** Ambient temperature below 5°C. Risk of frost damage to evaporator coil. Device may shut down to prevent damage.
- **Bit 4 - Temperature Too High:** Ambient temperature above 38°C. Outside operating range. Device may reduce performance or shut down.
- **Bit 5 - Humidity Too Low:** Ambient humidity below 25%. May indicate sensor fault or extremely dry environment.
- **Bit 6 - Coil Sensor Error:** Evaporator coil temperature sensor malfunction. May affect defrost cycle.
- **Bit 7 - Motor Error:** Fan motor fault. Check for obstructions or bearing failure.

---

### DP20 - Filter Reset

**Type:** Boolean  
**Access:** Write only  
**Function:** Reset filter life counter

- Send `true` (1) to reset DP23 (Filter Life) back to 100%

**Usage:** Trigger this after cleaning or replacing the filter to reset the maintenance reminder.

**Notes:** This is a write-only command. There is no readable state.

---

### DP23 - Filter Life Remaining

**Type:** Integer  
**Access:** Read only  
**Function:** Filter life percentage remaining

- **Range:** 0 to 100
- **Unit:** Percentage (%)

**Behavior:**
- Decreases based on runtime hours (exact calculation/threshold unknown)
- When approaching 0%, cleaning reminder activates (DP19 bit 1)
- Reset to 100% using DP20 after filter maintenance

**Notes:** 
- Approximate calculation based on operating hours, not actual filter condition
- The specific number of operating hours that correspond to the filter life percentage is unknown
- The device likely tracks cumulative runtime internally to calculate this value

---

### DP24 - Temperature Unit

**Type:** Enum  
**Access:** Read/Write  
**Function:** Temperature display unit

| Value | Unit |
|-------|------|
| 0 | Celsius (°C) |
| 1 | Fahrenheit (°F) |

**Notes:** Affects both device display and DP7 temperature reporting.

---

### DP102 - Client ID

**Type:** Integer  
**Access:** Read only  
**Function:** Tuya device client identifier (uncertain)

**Notes:** 
- Believed to be an internal Tuya protocol identifier
- Primarily useful for debugging communication issues
- Value is device-specific and not user-configurable
- ⚠️ The exact purpose and meaning of this datapoint is not fully confirmed

---

### DP104 - Equipment Type

**Type:** Integer  
**Access:** Read only  
**Function:** Device type/model identifier (uncertain)

**Known Values:**
- `4` = ProBreeze PB-D-24-WF (observed value)

**Notes:** 
- Believed to be used by Tuya ecosystem to identify device capabilities
- May vary for different models
- ⚠️ The exact purpose and meaning of this datapoint is not fully confirmed

---

## Diagnostic Commands

### Enable Tuya Debug Logging

To see raw Tuya datapoint messages in ESPHome logs:

```yaml
logger:
  level: DEBUG
  logs:
    tuya: VERBOSE
```

### Example Log Output

```
[12:34:56][D][tuya:264]: Datapoint 1 update (bool): ON
[12:34:57][D][tuya:264]: Datapoint 6 update (int): 65
[12:34:58][D][tuya:264]: Datapoint 7 update (int): 22
[12:35:00][D][tuya:264]: Datapoint 19 update (int): 0
```

## Discovering New Datapoints

If you want to discover additional datapoints for your device:

1. Enable VERBOSE logging (see above)
2. Interact with the device using physical buttons
3. Observe the logs for datapoint updates
4. Cross-reference with the Tuya MCU protocol documentation
5. Test by sending values via ESPHome
6. Document and contribute findings!

## Protocol Notes

- **Baud Rate:** 9600 (confirmed working)
- **UART:** 8N1 (8 data bits, no parity, 1 stop bit)
- **Protocol:** Tuya MCU Serial Protocol v3
- **Byte Order:** Big-endian for multi-byte values

## Resources

- [Tuya MCU Serial Protocol](https://developer.tuya.com/en/docs/iot/tuya-cloud-universal-serial-port-access-protocol?id=K9hhi0xxtn9cb)
- [ESPHome Tuya Component](https://esphome.io/components/tuya.html)
- [Tuya IoT Platform](https://iot.tuya.com/)

## Contributing

Found additional datapoints or corrections? Please submit an issue or pull request to the repository!

---

**Last Updated:** 2026  
**Device Model:** ProBreeze PB-D-24-WF (20L)  
**Firmware Version:** Compatible with standard Tuya MCU firmware
