# RUDP Emulator – Architecture Showcase

## Overview
The **RUDP Emulator** is a C++ reference implementation of a Reliable UDP (RUDP) transport layer. It is used for testing and demonstration of reliable, ordered, and error‑recovering data transfer over an unreliable UDP substrate. The emulator simulates the full protocol stack (packet construction, error detection, retransmission, NACK handling, and session management) and can be driven via a command‑line interface or integrated as a library into other projects.

## Solved Problem
- Provide a lightweight, portable RUDP implementation suitable for education, prototyping, and integration into embedded or network‑aware applications.
- Demonstrate robust packet framing, CRC checks, seven‑bit encoded integers for variable‑length fields, and selective retransmission driven by NACKs.
- Offer a clear, extensible class hierarchy for different package types (data, acknowledgments, error messages, control messages).

## Technology Stack
- **Language:** C++17 (compiled with GCC/clang)
- **Build System:** GNU Make (Makefile at project root) – CMake files also present but not the primary build.
- **External Dependencies:**
  - `json-c` – JSON parsing for configuration and serialization.
  - Standard C library (`<cstdio>`, `<cstdlib>`, `<cstring>`, `<cmath>`).
  - POSIX sockets (`<sys/socket.h>`, `<arpa/inet.h>`) for UDP/TCP emulation.
  - `pthread` for session management (optional).
- **Testing/Analysis:** `clang-tidy` for static code analysis; unit‑test‑friendly structure (packages have `make_from_bytes`, `make_bytes`, `make_json` methods).
- **Version Control:** Git (repository resides in `.git` folder, excluded from this showcase).

## Build Schema
```bash
# From the root of the emulator source
make               # builds the RUDPEmulator binary
make clean         # removes object files and binary
clang-tidy -checks=bugprone-*,performance-*,readability-* \
    --warnings-as-errors=* \
    $(find . -name '*.cpp' -o -name '*.h') \
    -export-fixes=clang-tidy-fixes.txt
```
The Makefile compiles the source files located under `RUDPEmulator/` and links them against `libjson-c`. The resulting binary `RUDPEmulator` can be executed with various CLI commands (see the original project's `main.cpp` for the full command list).

## Directory Layout (relevant part)
```
rudp-architecture-showcase/
│
├─ README.md                # you are here
├─ docs/
│   ├─ architecture-c4.md   # C4 context & container diagrams (Mermaid)
│   └─ protocol-and-sequence.md  # packet format & Mermaid sequence diagrams
├─ .github/
│   └─ workflows/
│       └─ ci.yml          # GitHub Actions CI pipeline
└─ LICENSE                 # (placeholder – see original project license)
```

---
*This showcase does **not** include the hidden `.git` folder or any company‑specific identifiers. All code snippets and descriptions are derived from the public source tree.*