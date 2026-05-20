# Architecture

## Overview

The firmware acts as a protocol bridge between a Riden power supply (connected via UART/Modbus RTU) and multiple network protocols (Modbus TCP, SCPI raw socket, VXI-11, HTTP).

```mermaid
graph TB
    PSU[Riden Power Supply] <-->|Modbus RTU / UART| RM[RidenModbus]
    RM --> MB[RidenModbusBridge<br/>Modbus TCP :502]
    RM --> SCPI[RidenScpi<br/>SCPI raw socket :5025]
    SCPI --> VXI[VXI_Server<br/>VXI-11]
    VXI --> RPC[RPC_Bind_Server<br/>Port mapper]
    RM --> HTTP[RidenHttpServer<br/>Web UI :80]
    
    subgraph Network Clients
        C1[Modbus TCP Client]
        C2[SCPI/VISA Client]
        C3[Web Browser]
    end
    
    C1 --> MB
    C2 --> SCPI
    C2 --> VXI
    C3 --> HTTP
```

## Initialization Flow

```mermaid
sequenceDiagram
    participant Main as main.cpp
    participant Config as RidenConfig
    participant Modbus as RidenModbus
    participant WM as WiFiManager
    participant Services as Network Services

    Main->>Config: begin()
    Main->>Modbus: begin() [retry up to 5s]
    alt PSU connected
        Main->>Modbus: get_serial_number()
        Main->>WM: autoConnect(hostname)
        Main->>Services: begin() [SCPI, ModbusBridge, VXI, RPC]
    else PSU not connected
        Main->>WM: autoConnect(nullptr)
        Note over Main: LED fast-flash, limited mode
    end
    Main->>Services: http_server.begin()
```

## Main Loop

All services use cooperative multitasking via `loop()` calls in `main.cpp::loop()`:
- `riden_modbus.loop()` — process pending RTU transactions
- `riden_scpi.loop()` — accept/handle SCPI socket clients
- `modbus_bridge.loop()` — handle Modbus TCP requests
- `rpc_bind_server.loop()` / `vxi_server.loop()` — VXI-11 protocol
- `http_server.loop()` — web interface requests
- `ArduinoOTA.handle()` — OTA firmware updates
- `MDNS.update()` — mDNS responder

## Design Patterns

- **Namespace:** All project code lives in `RidenDongle` namespace
- **Composition:** Services receive `RidenModbus&` reference for PSU access
- **Static instances:** All service objects are static globals in `main.cpp`
- **Conditional logging:** Logging macros compile to no-ops unless `MODBUS_USE_SOFWARE_SERIAL` is defined (development board only)
- **SCPI bridge pattern:** `SCPI_handler_interface` abstracts SCPI parser access so VXI-11 and raw socket share the same parser
