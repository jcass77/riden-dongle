# Dependencies

## PlatformIO Libraries

| Library | Version | Purpose |
|---|---|---|
| `sfeister/SCPI_Parser` | ^2.2.0 | SCPI command parsing |
| `emelianov/modbus-esp8266` | ^4.1.0 | Modbus RTU and TCP |
| `wnatth3/WiFiManager` | 2.0.16-rc.2 | WiFi provisioning captive portal |
| `full-stack-ex/TinyTemplateEngine` | ^1.1 | HTML templating for web UI |

## Arduino/ESP8266 Core Libraries (bundled)

- `ESP8266WiFi` — WiFi connectivity
- `ESP8266WiFiGratuitous` — Gratuitous ARP for keep-alive
- `ESP8266mDNS` — mDNS/Bonjour advertising
- `ESP8266WebServer` — HTTP server
- `ArduinoOTA` — Over-the-air updates
- `EEPROM` — Persistent configuration storage
- `Ticker` — Timer-based LED blinking
- `SoftwareSerial` — Development board UART (nodemcuv2 env only)

## Build Dependencies

| Tool | Version | Purpose |
|---|---|---|
| PlatformIO | >=6.1.13 | Build system |
| Python | 3.12 (CI) | Build scripts |

## CI/CD

- **GitHub Actions** — release workflow triggered on published releases
- **act** — local workflow testing (see `act/` directory)

## Key Dependency Notes

- **WiFiManager** is pinned to `2.0.16-rc.2` (release candidate). The library is referenced via the `wnatth3` PlatformIO registry name (tzapu/WiFiManager upstream).
- **modbus-esp8266** provides both `ModbusRTU` and `ModbusTCP` classes used by different components.
- **VXI-11 server** is a custom in-tree implementation (`src/vxi11_server/`), not an external dependency.
