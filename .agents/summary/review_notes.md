# Review Notes

## Consistency Check

No inconsistencies found across documentation files.

## Completeness Check

### Well-documented areas
- Modbus register map (complete enum with all registers)
- SCPI command set (documented in SCPI_COMMANDS.md)
- Network interfaces and ports
- Build system and environments
- Hardware preparation (README.md)

### Areas with limited detail
- **HTTP API response formats:** The `/status` endpoint JSON schema is not documented; it's embedded in `riden_http_server.cpp`.
- **Web UI HTML generation:** The `html_content/extract_code.py` workflow for generating `http_static.h` from `mockup.html` is not documented beyond the readme in that directory.
- **Error handling patterns:** How Modbus RTU timeouts and failures propagate to network clients is implementation-specific and not explicitly documented.
- **VXI-11 implementation details:** The custom RPC/VXI-11 implementation has a README in `src/vxi11_server/` but protocol edge cases are not documented.

### Gaps from language/tooling limitations
- None — the codebase is entirely C++ with Python build scripts, both fully analyzable.

## Recommendations

1. Document the `/status` JSON response schema for clients building custom dashboards.
2. Add inline documentation to `extract_code.py` explaining the HTML→C header workflow.
3. Consider documenting the Modbus RTU timeout/retry behavior for users experiencing reliability issues.
