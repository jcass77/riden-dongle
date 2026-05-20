# Data Models

## Modbus Register Map

Defined in `include/riden_modbus/riden_modbus_registers.h` as `enum class Register`.

Key register groups:
- **0-20:** Core readings (ID, serial, firmware, temp, V/I set/out, power, protection, output state)
- **32-41:** Battery mode, probe temp, Ah/Wh counters
- **48-53:** Date/time (year, month, day, hour, minute, second)
- **55-62:** Calibration values (V_OUT, V_BACK, I_OUT, I_BACK zero/scale)
- **66-72:** Settings (TakeOk, TakeOut, PowerOnBoot, Buzzer, Logo, Language, Brightness)
- **80-119:** Presets M0-M9 (4 registers each: V, I, OVP, OCP)
- **256:** System register
- **5633:** Bootloader magic number

## Core Structs

### AllValues
Complete PSU state snapshot. Used by HTTP server for status display.

### Preset
```cpp
struct Preset { double voltage, current, over_voltage_protection, over_current_protection; };
```

### Calibration
```cpp
struct Calibration { uint16_t V_OUT_ZERO, V_OUT_SCALE, V_BACK_ZERO, V_BACK_SCALE, I_OUT_ZERO, I_OUT_SCALE, I_BACK_ZERO, I_BACK_SCALE; };
```

### Enums
- `Protection`: OVP, OCP, None
- `OutputMode`: CONSTANT_VOLTAGE, CONSTANT_CURRENT, Unknown

## Configuration (EEPROM)

Managed by `RidenConfig`:
- Timezone name (string)
- UART baudrate (uint32_t, default 9600)
- Config portal on boot flag (bool)

## Value Conversions

`RidenModbus` handles unit conversions between raw register values and engineering units using multipliers:
- `v_multi` (100.0) — voltage
- `i_multi` (100.0) — current  
- `p_multi` (100.0) — power
- `v_in_multi` (100.0) — input voltage

These are set during `begin()` based on the detected PSU model.
