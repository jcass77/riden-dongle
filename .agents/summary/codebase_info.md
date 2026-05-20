# Codebase Information

## Project

- **Name:** Riden Dongle
- **Description:** Multi-protocol alternative firmware for the Riden WiFi module (ESP-12F/ESP8266)
- **License:** MIT
- **Language:** C++ (Arduino framework on ESP8266)
- **Build System:** PlatformIO
- **Repository:** https://github.com/morgendagen/riden-dongle

## Technology Stack

- **MCU:** ESP8266 (ESP-12F module)
- **Framework:** Arduino (via PlatformIO espressif8266 platform)
- **Protocols:** Modbus RTU, Modbus TCP, SCPI (raw socket + VXI-11), HTTP, mDNS, NTP, OTA
- **Formatter:** clang-format (Linux brace style, 4-space indent, no column limit)

## Build Environments

| Environment | Board | Purpose |
|---|---|---|
| `esp12e` | ESP-12E | Production target |
| `nodemcuv2` | NodeMCU v2 | Development (uses SoftwareSerial) |

## Supported Hardware

Riden power supplies: RD6006, RD6012, RD6018, RD6024, RD6030, RD6006P, RD6012P (RD6018P experimental)
