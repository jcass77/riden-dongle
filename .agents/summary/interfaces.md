# Interfaces

## Network Interfaces

| Protocol | Port | VISA String | Discovery |
|---|---|---|---|
| Modbus TCP | 502 | N/A | mDNS |
| SCPI raw socket | 5025 | `TCPIP::<ip>::5025::SOCKET` | lxi-tools |
| VXI-11 | dynamic (RPC) | `TCPIP::<ip>::INSTR` | mDNS, TCP, UDP |
| HTTP | 80 | N/A | mDNS (`http://<hostname>.local`) |
| OTA (ArduinoOTA) | default | N/A | mDNS (`_arduino._tcp`) |

## SCPI Command Interface

See `SCPI_COMMANDS.md` for full list. Key commands:
- `*IDN?` — identification
- `*RCL {1-9}` — recall preset
- `VOLT` / `VOLT?` — set/get voltage
- `CURR` / `CURR?` — set/get current
- `OUTP` / `OUTP?` — set/get output state
- `MEAS:VOLT?` / `MEAS:CURR?` / `MEAS:POW?` — measurements

## Internal Interfaces

### SCPI_handler_interface (abstract)
```cpp
class SCPI_handler_interface {
    virtual void write(const char *data, size_t len) = 0;
    virtual scpi_result_t read(char *data, size_t *len, size_t max_len) = 0;
    virtual bool claim_control() = 0;
    virtual void release_control() = 0;
};
```
Used by VXI_Server to access the shared SCPI parser. Mutual exclusion: VXI-11 disables raw socket while active.

### RidenModbus public API
All PSU interactions go through `RidenModbus`. Methods return `bool` (success/failure). Data passed by reference. See `include/riden_modbus/riden_modbus.h` for full API.

## HTTP API Endpoints

| Method | Path | Purpose |
|---|---|---|
| GET | `/` | Home page |
| GET | `/psu` | PSU details page |
| GET | `/config` | Configuration page |
| POST | `/config` | Save configuration |
| GET | `/control` | Remote control page |
| GET | `/status` | JSON status (for control page AJAX) |
| POST | `/set_v` | Set voltage |
| POST | `/set_i` | Set current |
| POST | `/toggle_out` | Toggle output |
| POST | `/disconnect_client` | Disconnect a remote client |
| GET | `/reboot` | Reboot dongle |
| POST | `/firmware` | OTA firmware upload |
| GET | `/lxi/identification` | LXI identification XML |
