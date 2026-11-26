# C4 Architecture Diagrams – RUDP Emulator

## 1. Context Diagram (Level 0)

```mermaid
flowchart LR
    C1["Client Tool / Test Harness"]
    C2["Remote Peer / Server"]
    Sys["RUDP Emulator<br/>(C++17 CLI & Library)"]

    C1 -->|"CLI commands / UDP packets"| Sys
    Sys -->|"Responses / Telemetry"| C1
    C2 <-->|"RUDP session, data, ACK/NACK"| Sys
```

**Description:** The emulator sits as a stand-alone binary that communicates with external entities (client tools, test harnesses, or other systems) via a command-socket and/or UDP packets. It can act as both client and server depending on the invoked mode.

---

## 2. Container Diagram (Level 1)

```mermaid
flowchart TB
    subgraph Binary ["RUDP Emulator Binary"]
        direction LR
        P1["CLI & Message Parser<br/>(main.cpp, command-handler.cpp)"]
        P2["Session Manager<br/>(session_manager.h, RUDPSessionManager.cpp)"]
        P3["Transport Layer<br/>(udp_socket.cpp, tcp_socket.cpp)"]
        P4["Package Library<br/>(rudp.h, ams_packet.h, crc.h)"]
        P5["Configuration JSON<br/>(binary_config.h, BinaryConfig.cpp)"]
        P6["Logging<br/>(logger.h, logger.cpp)"]
    end

    P1 -->|"Parses args & routes commands"| P2
    P1 -->|"Loads settings"| P5
    P2 -->|"Creates/destroys sessions"| P3
    P2 -->|"Builds & parses packets"| P4
    P2 -->|"Writes DBG/INFO/ERROR"| P6
    P3 -->|"Serializes datagrams"| P4
```

**Description:** The binary is decomposed into logical containers that each own a set of responsibilities. The Package Library is the core of the RUDP protocol, handling framing, CRC16/32, seven-bit encoding, and error-type packages.

---

## 3. Component Diagram (Level 2) – Package Hierarchy

```mermaid
classDiagram
    class RUDPPackage {
        +uint8_t m_package_type
        +uint16_t m_crc16
        +int32_t m_error_code
        +make_bytes() vector~uint8_t~
        +make_from_bytes(vector~uint8_t~) void
        +make_json(json_object*) void
        +print() void
    }

    class RUDPErrorPackage {
        <<abstract>>
        +uint32_t m_file_id
    }

    class RUDPErrorNACKPackage {
        +uint32_t m_file_id
        +uint32_t m_segment_number
        +make_from_bytes_extra(buffer) uint16_t
        +make_bytes_extra() vector~uint8_t~
        +make_json_extra(json_object*) void
        +print_extra() void
    }

    class RUDPErrorCRC32Package {
        +uint32_t m_calculated_crc32
        +uint32_t m_received_crc32
        +uint32_t m_file_id
        +make_bytes_extra() vector~uint8_t~
    }

    class RUDPErrorOutOfMemoryPackage {
        +uint32_t m_file_id
        +make_bytes_extra() vector~uint8_t~
    }

    class RUDPErrorWrongFileIdPackage {
        +uint32_t m_file_id
        +make_bytes_extra() vector~uint8_t~
    }

    class RUDPErrorFlagPackage {
        +uint8_t m_flag
        +uint32_t m_file_id
        +Flag STOP_DATA_STOP_RECEIVE_TIMEOUT
        +make_bytes_extra() vector~uint8_t~
    }

    RUDPPackage <|-- RUDPErrorPackage
    RUDPErrorPackage <|-- RUDPErrorNACKPackage
    RUDPErrorPackage <|-- RUDPErrorCRC32Package
    RUDPErrorPackage <|-- RUDPErrorOutOfMemoryPackage
    RUDPErrorPackage <|-- RUDPErrorWrongFileIdPackage
    RUDPErrorPackage <|-- RUDPErrorFlagPackage
```

**Description:** The package class hierarchy follows the template method pattern. The base RUDPPackage defines the common serialization flow (make_bytes), CRC generation, and JSON conversion.
