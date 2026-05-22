# Command and Control (C2) Architecture & Topology

## Architectural Overview

The AndroSpectra Command and Control (C2) architecture is designed as a decoupled, asynchronous system. Rather than relying on fragile, persistent TCP sockets (which are easily flagged by mobile operating systems), the architecture mimics modern microservice interactions. It utilizes HTTPS polling and a centralized database to queue commands and receive exfiltrated data stealthily.

## Target & Infrastructure Topology

The following diagram illustrates the theoretical data flow between the infected Android 15 endpoint and the Red Team infrastructure. 

```mermaid
flowchart TD
    A[Start C2 GUI] --> B[Main Window - PyQt5]
    B --> C{User selects tab}

    C -->|Tab 1| D[Dashboard]
    C -->|Tab 2| E[APK Builder]
    C -->|Tab 3| F[Server Listener]
    C -->|Tab 4| G[Module Grabber]

    subgraph Backend
        H[Listener Thread\nTCP port 5555]
        I[Device Handler\n1 per connection]
    end

    F --> H
    H -->|accept connection| I
    I -->|signal: device online| D

    D -->|user types command| J[Send command via socket]
    J --> I
    I -->|device response| K[Console + Table update]

    G -->|button click e.g., Contacts| I
    I -->|file/completion marker| L[Save to ./output/]
