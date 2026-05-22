
# Isolated Laboratory Environment Setup

To ensure compliance with academic ethical guidelines, local cybersecurity laws (PECA 2016), and to prevent accidental network egress, the AndroSpectra architecture was developed, compiled, and tested entirely within a strictly air-gapped virtual environment.

## 1. Virtualization Infrastructure

* **Host Machine:** Dedicated hypervisor (VMware Workstation).
* **Target Environment:** Android Virtual Device (AVD) running Android 15 (API Level 35) with Google APIs, strictly configured to use a "Host-Only" network adapter. No external internet routing was permitted.
* **C2 Environment:** A Debian Linux virtual machine acting as the Threat Actor infrastructure, hosted on the same isolated subnet.

## 2. Network Topology (Host-Only)

```mermaid
flowchart LR
    subgraph lab [Isolated Lab Subnet 192.168.50.0/24]
        A[Android 15 Target IP: .10] <--> B[Debian C2 Server IP: .20]
        C[Wireshark / Zeek Monitor] -.->|Port Mirroring| A
        C -.->|Port Mirroring| B
    end
