# Shelly 1PM Gen4 ESPHome Templates Comparison

## Executive Summary

This document compares two ESPHome configurations for the Shelly 1PM Gen4:
- **Template A**: Advanced feature-rich configuration (856 lines) - IMPLEMENTED
- **Template B**: Basic community template (180 lines) - USER-PROVIDED

**Verdict**: Template A is superior for production use, with Template B offering valuable improvements for power metering configuration.

**Recommended Action**: Enhance Template A with Template B's power metering best practices.

---

## Templates Overview

### Template A: Implemented Configuration
- **Location**: `shelly-1pm-gen4/shelly-1pm-gen4.yaml`
- **Lines**: 856
- **Origin**: Adapted from Shelly 1PM Mini Gen3
- **Status**: ✅ Compiles successfully
- **Complexity**: Advanced (6 switch modes, BLE, protection systems, events)

### Template B: User-Provided Simple Configuration
- **Source**: User message (inline YAML)
- **Lines**: ~180 (estimated)
- **Origin**: Community template or basic example
- **Status**: ⚠️ Not verified to compile
- **Complexity**: Basic (simple protection, minimal features)

---

## Critical Differences

### 1. GPIO Pin Configuration

| Function | Template A | Template B | Status |
|----------|-----------|-----------|--------|
| Relay Output | GPIO4 | GPIO4 | ✅ Same |
| Status LED | GPIO0 (inverted) | GPIO0 (inverted) | ✅ Same |
| Button | GPIO1 (pullup, inverted) | GPIO1 | ⚠️ A has pullup |
| Switch Input | GPIO10 | GPIO10 | ✅ Same |
| NTC Temp | GPIO3 (ADC 12dB) | GPIO3 (ADC 11dB) | ❌ Different attenuation |
| BL0942 UART | TX:GPIO6, RX:GPIO7 (pullup) | TX:GPIO6, RX:GPIO7 (pullup) | ✅ Same |
| Add-on GPIO12 | Digital In (pullup) | Digital In | ✅ Same |
| Add-on GPIO16 | Data In | Data In | ✅ Same |
| Add-on GPIO17 | Commented (ADC not supported) | Binary sensor | ❌ B incorrect |
| Add-on GPIO9 | Data Out (output + switch) | Not present | ⚠️ A has extra |

### 2. ADC Attenuation - Critical Finding

**Template A**: `attenuation: 12db` (GPIO3:254)
- ESPHome issued warning during compilation:
  ```
  WARNING `attenuation: 11db` is deprecated, use `attenuation: 12db` instead
  ```

**Template B**: `attenuation: 11db`
- Uses deprecated syntax

**Verdict**: ✅ Template A is correct

### 3. GPIO17 Pin Usage - Critical Error in Template B

**ESP32-C6 ADC Support**: Only GPIO0-6 support ADC

**Template B Error**:
```yaml
binary_sensor:
  - name: addon analog in  # ❌ Misleading name
    platform: gpio         # ❌ Uses as digital input
    pin: GPIO17            # ❌ Pin does NOT support ADC
```

**Template A Solution**:
```yaml
# NOTE: GPIO17 does not support ADC on ESP32-C6 (ADC only on GPIO0-6)
# Commented out until correct GPIO pin is confirmed
# - platform: adc
#   name: "Add-on Analog In"
#   pin: GPIO17  # ❌ Will not work for ADC
```

**Verdict**: ✅ Template A correctly documents the limitation

---

## Feature Comparison

| Feature | Template A | Template B | Winner |
|---------|-----------|-----------|--------|
| **Switch Modes** | 6 modes (Edge, Follow, Detached, Multi-Click, Smart Multi-Click, Pulse Counter) | Basic toggle only | 🏆 A |
| **Multi-Click Events** | Full (single/double/triple/long press) | None | 🏆 A |
| **Pulse Counter** | Persistent with reset button | None | 🏆 A |
| **BLE Scanner** | Active proxy + mode select (Disabled/Passive) | None | 🏆 A |
| **Relay Restore Modes** | 4 modes (configurable) | ALWAYS_ON only | 🏆 A |
| **Overpower Protection** | Configurable (500-2000W), auto-clear at 90% | Fixed 16A, 1s delay | 🏆 A |
| **Overheat Protection** | Configurable threshold + hysteresis (60-90°C) | Fixed 75°C | 🏆 A |
| **Temperature Filtering** | 5-sample moving average | None | 🏆 A |
| **WiFi Diagnostics** | WiFi signal + uptime sensors | None | 🏆 A |
| **Add-on GPIO9** | Output + switch control | Missing | 🏆 A |
| **Power Metering** | Good (missing line_freq, state_class) | Better (complete config) | 🏆 B |
| **Sensor Categories** | Basic | `entity_category: diagnostic` for V/A | 🏆 B |
| **Energy Tracking** | `device_class: energy` | `device_class: energy, state_class: total_increasing` | 🏆 B |

---

## Protection System Implementation

### Template A: Global Variables Approach

```yaml
globals:
  - id: overtemp_latched (non-persisted)
  - id: overpower_latched (non-persisted)

# Overpower: Lambda in power sensor on_value
# - Checks if relay ON and power > limit
# - Sets latch, publishes error, turns off relay
# - Auto-clears at 90% threshold

# Overheat: Template binary sensor
# - Updates every sensor update
# - Checks temp >= threshold, sets latch
# - Clears when temp < (threshold - hysteresis)
```

**Pros**:
- ✅ Configurable thresholds via number entities
- ✅ Hysteresis prevents flapping
- ✅ Auto-clear at 90% for overpower
- ✅ Persistent relay state across reboots

**Cons**:
- ❌ More complex code
- ❌ More lines

### Template B: Condition Template Approach

```yaml
binary_sensor:
  - platform: template
    condition:
      any:
        - binary_sensor.is_on: error_overtemp  # Self-reference for latching
        - sensor.in_range: {id: temperature, above: 75.0}

  - platform: template
    condition:
      any:
        - binary_sensor.is_on: error_overpower  # Self-reference
        - for: {time: 1s, condition: sensor.in_range: {id: sensor_current, above: 16}}
```

**Pros**:
- ✅ Simpler to understand
- ✅ Self-documenting conditions
- ✅ Built-in debouncing (for: 1s)

**Cons**:
- ❌ Fixed thresholds (not configurable)
- ❌ No auto-clear mechanism
- ❌ Current-based overpower (16A) vs power-based (more accurate for reactive loads)
- ❌ Self-referencing may have edge cases

**Verdict**: 🏆 Template A (configurability + auto-clear outweigh simplicity)

---

## Power Metering Configuration Differences

| Aspect | Template A | Template B | Winner |
|--------|-----------|-----------|--------|
| Update Interval | 1s | 1s | Tie |
| Line Frequency | Not specified (50Hz default) | 60Hz | 🏆 B |
| Voltage Entity | `device_class: voltage` | `device_class: voltage, entity_category: diagnostic` | 🏆 B |
| Current Entity | `device_class: current` | `device_class: current, entity_category: diagnostic` | 🏆 B |
| Energy Tracking | `device_class: energy` | `device_class: energy, state_class: total_increasing` | 🏆 B |
| Frequency Accuracy | Not specified | `accuracy_decimals: 2` | 🏆 B |
| Energy Accuracy | Not specified | `accuracy_decimals: 3` | 🏆 B |

**Template A (lines 258-303)**:
```yaml
- platform: bl0942
  uart_id: bl0942_uart
  voltage:
    name: "Voltage"
    id: bvoltage
    device_class: voltage
  current:
    name: "Current"
    id: bcurrent
    device_class: current
  energy:
    name: "Energy"
    id: benergy
    device_class: energy
  frequency:
    name: "Frequency"
    id: bfreq
    accuracy_decimals: 2
    device_class: frequency
  update_interval: 1s
```

**Template B (better practices)**:
```yaml
- platform: bl0942
  line_frequency: 60Hz  # 🏆 Explicit frequency
  voltage:
    entity_category: diagnostic  # 🏆 Better categorization
  current:
    entity_category: diagnostic  # 🏆 Better categorization
  energy:
    state_class: total_increasing  # 🏆 Required for long-term stats
    accuracy_decimals: 3  # 🏆 More precise
  frequency:
    accuracy_decimals: 2  # ✅ Both have this
```

**Recommendation**: Adopt Template B's improvements for Template A

---

## Boot Behavior

| Aspect | Template A | Template B |
|--------|-----------|-----------|
| Boot Script | Priority -100, complex relay restore logic | None (relies on restore_mode) |
| Relay Restore | 4 selectable modes | ALWAYS_ON hardcoded |
| Temp Check on Boot | Yes - prevents ON if overheated | Yes - in relay turn_on_action |
| BLE Restoration | Restores scanner mode after 1s | N/A |
| Pulse Counter | Publishes saved count | N/A |

**Verdict**: 🏆 Template A (more control and flexibility)

---

## Correctness Assessment

### Template A Issues
1. ✅ **Fixed**: ADC attenuation corrected to 12dB (line 254)
2. ✅ **Fixed**: GPIO17 ADC commented out with explanation (lines 332-345)
3. ✅ **Verified**: Compiles successfully
4. ⚠️ **Missing**: BL0942 line_frequency not specified (defaults to 50Hz)
5. ⚠️ **Missing**: Energy sensor missing `state_class: total_increasing`
6. ⚠️ **Missing**: Voltage/Current sensors missing `entity_category: diagnostic`

### Template B Issues
1. ❌ **Error**: Uses deprecated `attenuation: 11db`
2. ❌ **Misleading**: GPIO17 labeled "analog_in" but used as digital binary sensor
3. ⚠️ **Fixed threshold**: Overpower at 16A (3.68kW @ 230V) - may not match user needs
4. ⚠️ **Not verified**: Compilation not tested
5. ⚠️ **No hysteresis**: Overheat protection may flap near threshold
6. ⚠️ **No auto-clear**: Requires manual intervention to clear faults

---

## Use Case Recommendations

### ✅ Use Template A When:
- You need **multiple switch modes** (toggle, follow, detached, multi-click, pulse counter)
- You want **configurable protection thresholds** via Home Assistant UI
- You need **multi-click events** for automations (single/double/triple click, long press)
- You want **BLE scanning** capability
- You need **flexible relay restore modes** for different scenarios
- You want **diagnostic sensors** (WiFi signal, uptime, pulse counter)
- You need **GPIO9 add-on output** control
- You prefer **temperature hysteresis** to prevent protection flapping
- You want **auto-clearing overpower** protection (at 90% threshold)
- You need **persistent pulse counting** across reboots

### ✅ Use Template B When:
- You want a **minimal, simple configuration**
- You only need **basic on/off control** (no advanced switch modes)
- Fixed protection thresholds are acceptable (75°C, 16A)
- You don't need BLE, multi-click, or pulse counting
- You prefer **simpler code** that's easier to understand
- You're comfortable with **template condition** syntax
- You need **60Hz line frequency** (Template A defaults to 50Hz)

---

## Recommended Improvements to Template A

Based on Template B's best practices:

1. **Add line_frequency to BL0942** (line ~303):
   - Add `line_frequency: 60Hz` (or 50Hz based on region)
   - Makes frequency detection explicit

2. **Improve energy sensor** (line ~292):
   - Add `state_class: total_increasing`
   - Required for Home Assistant long-term statistics

3. **Categorize diagnostic sensors** (lines 260-274):
   - Add `entity_category: diagnostic` to voltage sensor
   - Add `entity_category: diagnostic` to current sensor
   - Hides them from main dashboard by default

4. **Add accuracy to energy** (line ~292):
   - Add `accuracy_decimals: 3`
   - More precise energy tracking

---

## Summary Table

| Criteria | Template A | Template B |
|----------|-----------|-----------|
| **Lines of Code** | 856 | ~180 |
| **Compilation** | ✅ Verified | ⚠️ Not tested |
| **Features** | 🟢 Advanced (6 modes, BLE, events) | 🟡 Basic (on/off) |
| **Configurability** | 🟢 High (UI thresholds, modes) | 🔴 Low (hardcoded) |
| **Protection** | 🟢 Hysteresis, auto-clear, configurable | 🟡 Fixed, manual clear |
| **Complexity** | 🔴 High (steep learning curve) | 🟢 Low (easy to understand) |
| **GPIO17 Handling** | 🟢 Correct (documented) | 🔴 Misleading (digital as "analog") |
| **ADC Attenuation** | 🟢 Correct (12dB) | 🔴 Deprecated (11dB) |
| **Power Metering** | 🟡 Good (missing extras) | 🟢 Better (complete) |
| **Maintenance** | 🟡 Requires ESPHome knowledge | 🟢 Simpler |
| **Production Ready** | 🟢 Yes | 🟡 With fixes |

---

## Conclusion

**🏆 Template A is superior for production use** due to:
- ✅ Verified compilation
- ✅ Rich feature set (6 switch modes, BLE, events, pulse counter)
- ✅ Configurable protection (thresholds, hysteresis, auto-clear)
- ✅ Correct GPIO handling (GPIO17 limitation documented)
- ✅ Advanced automation capabilities (multi-click, detached mode)
- ✅ Flexible relay restore modes
- ✅ Persistent state management

**💡 Template B offers valuable lessons**:
- ✅ Simpler protection implementation approach
- ✅ Better power metering sensor configuration
- ✅ Cleaner entity categorization

**🎯 Best Approach**:
Merge the strengths by enhancing Template A with Template B's power metering improvements while preserving all advanced features.

---

## Implementation Plan

1. ✅ Keep Template A as primary configuration
2. ⏳ Enhance with Template B's power metering best practices:
   - Add `line_frequency` parameter
   - Add `state_class: total_increasing` to energy sensor
   - Add `entity_category: diagnostic` to voltage/current sensors
   - Add `accuracy_decimals: 3` to energy sensor
3. ✅ GPIO17 limitation already documented
4. ⏳ Create "lite" variant based on Template B for users wanting simplicity (optional)

---

*Generated: 2026-02-04*
*Template A Version: 1.0.0*
