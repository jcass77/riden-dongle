# Workflows

## Build and Flash

```mermaid
graph LR
    A[pio run] --> B[get_version.py<br/>injects BUILD_VERSION]
    B --> C[get_build_time.py<br/>injects build timestamp]
    C --> D[Compile]
    D --> E[make_gz.py<br/>creates .bin.gz]
    D --> F[pio run -t upload]
```

## Release Workflow (CI)

```mermaid
graph LR
    A[GitHub Release published] --> B[Checkout + Setup Python]
    B --> C[Install PlatformIO]
    C --> D[pio run with VERSION env]
    D --> E[Upload firmware_*.bin + .gz to release]
```

## WiFi Connection Flow

```mermaid
graph TD
    A[Boot] --> B{Has WiFi credentials?}
    B -->|Yes| C[autoConnect]
    B -->|No| D[Start AP: RDxxxx-ssssssss]
    D --> E[Captive portal]
    E --> F[User enters WiFi credentials]
    F --> C
    C --> G{Connected?}
    G -->|Yes| H[Start services + mDNS]
    G -->|No| I[ESP.reset]
```

## SCPI Request Flow (Raw Socket)

```mermaid
sequenceDiagram
    participant Client
    participant RidenScpi
    participant SCPIParser
    participant RidenModbus
    participant PSU

    Client->>RidenScpi: TCP data on :5025
    RidenScpi->>SCPIParser: Feed input
    SCPIParser->>RidenModbus: get/set register
    RidenModbus->>PSU: Modbus RTU request
    PSU-->>RidenModbus: Modbus RTU response
    RidenModbus-->>SCPIParser: result
    SCPIParser-->>RidenScpi: formatted response
    RidenScpi-->>Client: TCP response
```

## VXI-11 Request Flow

```mermaid
sequenceDiagram
    participant Client
    participant RPCBind as RPC_Bind_Server
    participant VXI as VXI_Server
    participant Bridge as SCPI_handler
    participant SCPI as RidenScpi

    Client->>RPCBind: GETPORT request
    RPCBind-->>Client: VXI port number
    Client->>VXI: CREATE_LINK
    VXI->>Bridge: claim_control()
    Note over SCPI: Raw socket disabled
    Client->>VXI: DEVICE_WRITE (SCPI cmd)
    VXI->>Bridge: write()
    Bridge->>SCPI: write to parser
    Client->>VXI: DEVICE_READ
    VXI->>Bridge: read()
    Bridge-->>VXI: response
    VXI-->>Client: response
    Client->>VXI: DESTROY_LINK
    VXI->>Bridge: release_control()
```

## Web UI Control Flow

The control page uses AJAX polling (`/status` endpoint) to update readings every ~1 second, and POSTs to `/set_v`, `/set_i`, `/toggle_out` for control actions.

## OTA Update

Available via:
1. ArduinoOTA (mDNS-discoverable, PlatformIO compatible)
2. HTTP POST to `/firmware` (web interface upload)
