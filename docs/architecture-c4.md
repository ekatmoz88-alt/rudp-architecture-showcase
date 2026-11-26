# C4 Architecture – RUDP Emulator

## 1. Context Diagram (Level 0)

```mermaid
contextDiagram
    direction LR
    actor "Client" as C1
    actor "Server" as C2
    system "RUDP Emulator (binary)" as Sys

    C1 -> Sys : send CLI commands / UDP packets
    Sys -> C1  : return responses / data
    C2 -> Sys  : exchange RUDP sessions
    Sys -> C2  : session setup, teardown, NACKs
```

*Description:* The emulator sits as a stand‑alone binary that communicates with external entities (client tools, test harnesses, or other systems) via a command‑socket and/or UDP packets. It can act as both client and server depending on the invoked mode.

---

## 2. Container Diagram (Level 1)

```mermaid
containerDiagram
    direction LR
    boundary "RUDP Emulator Binary" as B1

    package "CLI & Message Parser" as P1 { \
        main.cpp \
        command‑handler.cpp \
    }
    package "Session Manager" as P2 { \
        session_manager.h \
        RUDPSessionManager.cpp \
    }
    package "Transport (UDP/TCP)" as P3 { \
        udp_socket.cpp \
        tcp_socket.cpp \
    }
    package "Package Library" as P4 { \
        rudp.h \
        ams_packet.h \
        ams_message.h \
        constants.h \
        crc.h \
        dc.h \
        cd.h \
        ppv.h \
        recently_watched.h \
        logger.h \
        logger.cpp \
        binary_config.h \
    }
    package "Configuration (JSON)" as P5 { \
        binary_config.h \
        BinaryConfig.cpp \
    }
    package "Logging" as P6 { \
        logger.cpp \
        logger.h \
    }

    B1 -> P1 : entry point, argument parsing
    B1 -> P2 : create / destroy sessions, lookup
    B1 -> P3 : send/receive UDP/TCP packets
    B1 -> P4 : build, parse, serialize RUDP packages
    B1 -> P5 : read/write BinaryConfig JSON
    B1 -> P6 : log messages (DBG/INFO/ERROR)
```

*Description:* The binary is decomposed into logical containers that each own a set of responsibilities. The **Package Library** is the core of the RUDP protocol, handling framing, CRC16/32, seven‑bit encoding, and error‑type packages. The **Session Manager** tracks per‑connection state, while the **Transport** abstraction sends/receives raw datagrams. The **CLI** parses command‑line arguments and forwards commands to the appropriate subsystem.

---

## 3. Component Diagram (Level 2) – Package Hierarchy

```mermaid
classDiagram
    class "RUDPPackage" {
        +uint8_t m_package_type
        +uint16_t m_crc16
        +int32_t m_error_code
        +virtual vector~uint8_t~ make_bytes()
        +virtual void make_from_bytes(vector~uint8_t~)
        +virtual void make_json(json_object*)
        +virtual void print()
    }

    class "RUDPErrorPackage" {
        <<abstract>>
        +uint32_t m_file_id (in subclasses)
    }

    class "RUDPErrorNACKPackage" {
        +uint32_t m_file_id
        +uint32_t m_segment_number
        +uint16_t make_from_bytes_extra(buffer)
        +vector~uint8_t~ make_bytes_extra()
        +void make_json_extra(json_object*)
        +void print_extra()
    }

    class "RUDPErrorCRC32Package" {
        +uint32_t m_calculated_crc32
        +uint32_t m_received_crc32
        +uint32_t m_file_id
        +... same virtual methods as RUDPErrorPackage
    }

    class "RUDPErrorOutOfMemoryPackage" {
        +uint32_t m_file_id
        +... same virtual methods
    }

    class "RUDPErrorWrongFileIdPackage" {
        +uint32_t m_file_id
        +... same virtual methods
    }

    class "RUDPErrorFlagPackage" {
        +uint8_t m_flag
        +uint32_t m_file_id
        +... same virtual methods
        +enum Flag { STOP_DATA, STOP_RECEIVE, CLIENT_TIMEOUT, SERVER_TIMEOUT }
    }

    "RUDPPACKAGE" <|-- "RUDPErrorPackage"
    "RUDPErrorPackage" <|-- "RUDPErrorNACKPackage"
    "RUDPErrorPackage" <|-- "RUDPErrorCRC32Package"
    "RUDPErrorPackage" <|-- "RUDPErrorOutOfMemoryPackage"
    "RUDPErrorPackage" <|-- "RUDPErrorWrongFileIdPackage"
    "RUDPErrorPackage" <|-- "RUDPErrorFlagPackage"
```

*Description:* The package class hierarchy follows the “template method” pattern. The base `"RUDPPackage"` defines the common serialization flow (`make_bytes`), CRC generation, and JSON conversion. Sub‑classes (`"RUDPErrorNACKPackage"`, `"RUDPErrorCRC32Package"`, `"RUDPErrorFlagPackage"`, etc.) implement `make_from_bytes_extra` and `make_bytes_extra` to encode/decode their specific fields (file‑id, segment‑number, flag, etc.). This design allows new package types to be added with minimal changes.

---
*All diagrams are expressed in Mermaid syntax and can be rendered by any Markdown viewer that supports Mermaid (e.g., GitHub, VS Code, GitLab).*