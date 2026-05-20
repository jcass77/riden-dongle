# AGENTS.md

Alternative firmware for the Riden WiFi module (ESP-12F/ESP8266). Bridges the PSU's Modbus RTU interface to Modbus TCP, SCPI (raw socket + VXI-11), and a web UI.

## Directory Map

```
src/
├── main.cpp                    # Entry point: setup/loop, WiFi, service orchestration
├── riden_modbus/               # Modbus RTU client — all PSU register access
├── riden_modbus_bridge/        # Modbus TCP server (port 502)
├── riden_scpi/                 # SCPI parser + raw TCP socket server (port 5025)
├── vxi11_server/               # Custom VXI-11/RPC implementation
├── scpi_bridge/                # Adapter: VXI_Server → RidenScpi parser
├── riden_http_server/          # Web UI (port 80), includes html_content/ assets
│   └── html_content/           # HTML mockup + extract_code.py → http_static.h
└── riden_config/               # EEPROM config (timezone, baudrate)

include/                        # Header files mirroring src/ structure
scripts/                        # Build helpers (version, build time, gzip) + test_pyvisa.py
```

## Architecture

All PSU communication flows through `RidenModbus` (Modbus RTU over UART). Network services receive a `RidenModbus&` reference. The VXI-11 and raw socket SCPI share a single SCPI parser instance via `SCPI_handler_interface`; VXI-11 claims exclusive access, disabling the raw socket while active.

Main loop is cooperative — each service has a `loop()` method called from `main.cpp::loop()`.

## Key Patterns

- **Namespace:** `RidenDongle` for all project code
- **Logging:** Macros in `include/riden_logging/riden_logging.h` — compile to no-ops on production (`esp12e`), only active when `MODBUS_USE_SOFWARE_SERIAL` is defined (dev board)
- **Conditional compilation:** `MOCK_RIDEN` enables software serial without a real PSU; `WM_DEBUG_UART` enables WiFiManager debug output on the hardware UART
- **Web UI generation:** `src/riden_http_server/html_content/mockup.html` → `extract_code.py` → `http_static.h` (gzipped C arrays)
- **Version injection:** `scripts/get_version.py` reads `VERSION` env var, defaults to `"DEVELOPMENT"`

## Build

- **Platform:** PlatformIO, espressif8266
- **Environments:** `esp12e` (production, optimized for size), `nodemcuv2` (development, SoftwareSerial, max 38400 baud)
- **Formatter:** `.clang-format` — Linux braces, 4-space indent, no column limit
- **CI:** GitHub Actions release workflow (`.github/workflows/release.yaml`)
- **Local CI test:** `act --container-architecture linux/amd64 ...` (see `DEVELOPMENT.md`)
- **Venv:** Project uses `.venv/` for PlatformIO — run `.venv/bin/pio` or activate first

### Build Commands

```bash
# Production build
VERSION=1.5.1 .venv/bin/pio run -e esp12e

# Development build (NodeMCU with SoftwareSerial debug)
.venv/bin/pio run -e nodemcuv2

# Debug build with WiFiManager logging (esp12e hardware)
# Replace WM_NODEBUG with WM_DEBUG_UART in platformio.ini [env:esp12e] build_flags
# Also add WM_DEBUG_UART guard to Serial.begin in main.cpp setup()
```

### Flashing

```bash
# Via Raspberry Pi Debug Probe (or similar USB-UART adapter)
# Put ESP into bootloader: hold GPIO0/PGM low, pulse RST, release GPIO0
/path/to/esptool --port /dev/ttyACM0 --baud 115200 \
  --before no-reset --after no-reset \
  write-flash 0x0 .pio/build/esp12e/firmware_esp12e_1_5_1.bin

# Via OTA (device must be on network)
.venv/bin/pio run -e esp12e -t upload --upload-port <device-ip>
```

## Debugging and Monitoring

### Serial Monitoring

The UART is shared between Modbus RTU (PSU communication) and debug output. On the `esp12e` production build, logging is compiled out. To get debug output:

1. **Development board (nodemcuv2):** Uses SoftwareSerial for Modbus, hardware Serial for debug at 74880 baud.
2. **Production board (esp12e) with debug probe:** The Modbus library reconfigures Serial to the PSU baudrate (default 9600, or whatever is configured). WiFiManager debug output appears at this baudrate.

```bash
# Monitor at PSU baudrate (typically 9600 or 115200 depending on config)
stty -F /dev/ttyACM0 9600 raw -echo && cat /dev/ttyACM0

# ROM boot messages appear at 74880 baud
stty -F /dev/ttyACM0 74880 raw -echo && cat /dev/ttyACM0
```

### WiFi Debug Build

To enable WiFiManager debug on the esp12e target:

1. In `platformio.ini` [env:esp12e], replace `-D WM_NODEBUG` with `-D WM_DEBUG_UART`
2. In `main.cpp` setup(), change `#ifdef MODBUS_USE_SOFWARE_SERIAL` to `#if defined(MODBUS_USE_SOFWARE_SERIAL) || defined(WM_DEBUG_UART)`
3. Monitor at the Modbus baudrate (not 74880)

**Note:** WiFiManager debug goes to the same UART as Modbus RTU. While debug is active, PSU communication will not work (no PSU needed for WiFi debugging).

### WiFi Troubleshooting

- ESP8266 only supports 2.4GHz WPA2-Personal
- If connection fails repeatedly, **reboot the router** — stale WPA2 session state on the router side is a common cause
- The firmware uses `setConnectTimeout(20)` and `setConnectRetries(3)` for resilience
- Known ESP8266 WiFi issues: https://github.com/tzapu/WiFiManager/issues/1067

## Configuration

- `.env` file in project root for serial port settings and extra build flags (not committed)
- PSU requires: UART Interface = TTL, Address = 1, baudrate matching dongle config

## Dependencies (non-obvious)

- `https://github.com/tzapu/WiFiManager.git#v2.0.17` — pulled from GitHub (PlatformIO registry `wnatth3` is outdated at 2.0.16-rc.2)
- `emelianov/modbus-esp8266` — provides both `ModbusRTU` and `ModbusTCP`
- VXI-11 server is fully in-tree (`src/vxi11_server/`), not a library

## Constraints

- **ESP8266:** ~80KB usable RAM, no RTOS, single-threaded cooperative multitasking
- **UART:** Shared between PSU communication and debug logging — only one at a time
- **WiFiManager:** Captive portal blocks until WiFi is configured; `ESP.reset()` on failure

## Custom Instructions

<!-- This section is maintained by developers and agents during day-to-day work.
     It is NOT auto-generated by codebase-summary and MUST be preserved during refreshes.
     Add project-specific conventions, gotchas, and workflow requirements here. -->
