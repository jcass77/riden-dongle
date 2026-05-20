# Riden Dongle — Documentation Index

## How to Use This Documentation

This index is designed as the primary context file for AI assistants. It contains enough metadata about each documentation file to determine which file(s) to consult for specific questions. Load this file first; then read individual files as needed.

## Table of Contents

| File | Purpose | Consult when... |
|---|---|---|
| [codebase_info.md](codebase_info.md) | Project metadata, tech stack, build environments, supported hardware | You need basic project facts, language, build system, or hardware compatibility |
| [architecture.md](architecture.md) | System design, initialization flow, main loop, design patterns | You need to understand how components connect, data flow, or the overall structure |
| [components.md](components.md) | Detailed breakdown of each module's role, files, and responsibilities | You need to find where specific functionality lives or understand a module's scope |
| [interfaces.md](interfaces.md) | Network protocols, ports, SCPI commands, HTTP endpoints, internal APIs | You need protocol details, API endpoints, VISA strings, or inter-component contracts |
| [data_models.md](data_models.md) | Modbus registers, structs, enums, EEPROM config, value conversions | You need register addresses, data structures, or unit conversion logic |
| [workflows.md](workflows.md) | Build, release, WiFi connection, SCPI/VXI-11 request flows, OTA | You need to understand a process end-to-end or debug a sequence of operations |
| [dependencies.md](dependencies.md) | External libraries, versions, build tools, CI/CD | You need library versions, what a dependency does, or build toolchain info |
| [review_notes.md](review_notes.md) | Documentation gaps and improvement recommendations | You want to know what's NOT well-documented |

## Quick Reference

- **Build:** `pio run` (production: `esp12e` env, development: `nodemcuv2` env)
- **Flash:** `pio run -t upload --upload-port <port>`
- **Source entry point:** `src/main.cpp`
- **All PSU access:** goes through `RidenModbus` class
- **Namespace:** `RidenDongle`
- **Formatter:** clang-format (Linux braces, 4-space indent)
- **Key constraint:** ESP8266 — limited RAM, cooperative multitasking, no RTOS
