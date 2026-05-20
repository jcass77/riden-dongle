# Components

## Core Components

### RidenModbus (`src/riden_modbus/`, `include/riden_modbus/`)
- **Role:** Modbus RTU client communicating with the Riden PSU over UART
- **Key files:** `riden_modbus.h`, `riden_modbus.cpp`, `riden_modbus_registers.h`
- **Responsibilities:** Read/write all PSU registers (voltage, current, presets, calibration, clock, settings)
- **Library:** `emelianov/modbus-esp8266`

### RidenModbusBridge (`src/riden_modbus_bridge/`, `include/riden_modbus_bridge/`)
- **Role:** Modbus TCP server bridging network clients to the PSU
- **Key files:** `riden_modbus_bridge.h`, `riden_modbus_bridge.cpp`
- **Port:** 502 (standard Modbus TCP)

### RidenScpi (`src/riden_scpi/`, `include/riden_scpi/`)
- **Role:** SCPI command parser and raw TCP socket server
- **Key files:** `riden_scpi.h`, `riden_scpi.cpp`
- **Port:** 5025
- **Library:** `sfeister/SCPI_Parser`

### VXI-11 Server (`src/vxi11_server/`)
- **Role:** VXI-11 (RPC-based) instrument control protocol
- **Key files:** `vxi_server.h/cpp`, `rpc_bind_server.h/cpp`, `rpc_packets.h/cpp`, `rpc_enums.h`
- **Note:** Custom implementation (not a library dependency)

### SCPI Bridge (`src/scpi_bridge/`)
- **Role:** Adapter connecting VXI_Server to RidenScpi's SCPI parser
- **Key file:** `scpi_bridge.h`
- **Pattern:** Implements `SCPI_handler_interface`

### RidenHttpServer (`src/riden_http_server/`, `include/riden_http_server/`)
- **Role:** Web interface for configuration, control, and monitoring
- **Key files:** `riden_http_server.h`, `riden_http_server.cpp`, `http_static.h`
- **Port:** 80
- **Sub-content:** `html_content/` contains mockup HTML and image assets; `extract_code.py` generates `http_static.h`

### RidenConfig (`src/riden_config/`, `include/riden_config/`)
- **Role:** Persistent configuration (timezone, baudrate, config portal flag) stored in EEPROM
- **Key files:** `riden_config.h`, `riden_config.cpp`, `timezones.h`

## Support Components

### Logging (`include/riden_logging/`)
- **File:** `riden_logging.h`
- **Behavior:** Macros that compile to Serial output only when `MODBUS_USE_SOFWARE_SERIAL` is defined (dev board)

### WiFi Management
- **Library:** `wnatth3/WiFiManager` (captive portal for WiFi credential setup)
- **Used in:** `main.cpp::connect_wifi()`

## Build Scripts (`scripts/`)

| Script | Purpose |
|---|---|
| `get_version.py` | Injects `BUILD_VERSION` from env var or defaults to "DEVELOPMENT" |
| `get_build_time.py` | Injects build timestamp (esp12e only) |
| `make_gz.py` | Compresses firmware binary to `.bin.gz` |
| `test_pyvisa.py` | PyVISA test script for SCPI verification |
