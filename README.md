# AndroSpectra: Android 15 Threat Modeling & C2 Architecture Research

**A comprehensive academic security research project focused on Command and Control (C2) topologies, trust-surface expansion, and defensive detection engineering within the Android 15 (API 35) ecosystem.**

[![Documentation Only](https://img.shields.io/badge/Status-Documentation_Only-blue.svg)](#)
[![Android 15 Research](https://img.shields.io/badge/Platform-Android_15_(API_35)-3DDC84.svg)](#)
[![MITRE ATT&CK](https://img.shields.io/badge/Framework-MITRE_ATT&CK-red.svg)](#)
[![University Approved](https://img.shields.io/badge/Ethics-Approved_Lab_Research-green.svg)](./ETHICAL_NOTICE.md)

---

## ⚠️ Executive & Ethical Summary
This repository contains **academic malware research documentation** for a Final Year Project (FYP). **No executable malware code, APKs, or exploit payloads are included.** All research was conducted in a strictly isolated, university-approved lab environment. 

Please read the [**ETHICAL_NOTICE.md**](./ETHICAL_NOTICE.md) before proceeding.

---

## 🔍 Research Scope & Objectives

As mobile operating systems mature, threat actors increasingly shift from exploiting kernel-level vulnerabilities to abusing legitimate platform features. Android 15 introduces strict boundaries, such as hardened Foreground Service (FGS) requirements, Restricted Settings for side-loaded applications, and granular scoped storage.

**AndroSpectra** is a conceptual, custom-designed Remote Access Trojan (RAT) architecture engineered solely to study these boundaries. The research focuses on:
1. **Capability Chaining:** How adversaries combine low-level permissions to achieve high-impact data extraction.
2. **C2 Architecture:** The decoupled design of modern mobile botnets and beaconing mechanisms.
3. **Detection Engineering:** Developing actionable Indicators of Compromise (IoCs) and behavioral signatures to aid Blue Teams and MDM administrators in identifying similar threats.

---

## 📂 Project Navigation

To understand the full scope of this research, explore the documentation in the following order:

### 1. Visual Evidence & Lab Demonstration
* [**Lab Demonstration & Screenshots (`/demo`)**](./demo/README.md) - Visual proof of the conceptual C2 dashboard, device session management, and data handling flows.

### 2. Threat Modeling & Offensive Architecture
* [**MITRE ATT&CK Threat Model**](./THREAT_MODEL.md) - Mapping AndroSpectra's theoretical capabilities to the MITRE ATT&CK for Mobile framework.
* [**C2 Server Topology**](./C2_SERVER_ARCHITECTURE.md) - Network diagrams, payload abstraction, and beaconing protocols.
* [**Capability Modules**](./modules-overview.md) - High-level summary of payload capabilities.

### 3. Defensive Engineering (Blue Team)
* [**Detection & Mitigation Strategies**](./DETECTION_AND_MITIGATION.md) - **[CRITICAL]** Network IoCs, host-based behavioral anomalies, and YARA rules for defending against this class of architecture.
* [**Isolated Lab Setup**](./LAB_SETUP.md) - Documentation of the safe, air-gapped environment used for this analysis.

---

## 🛡️ Key Defensive Findings (TL;DR)

1. **Accessibility Service Abuse:** Despite Android 15's "Restricted Settings," social engineering combined with session-based permission toggling remains the primary vector for UI-interaction hijacking.
2. **Asynchronous Exfiltration:** The most resilient C2 topologies do not rely on constant socket connections, but rather asynchronous, jitter-based HTTPS beaconing mimicking standard app analytics traffic.
3. **Detection Bottlenecks:** Traditional signature-based AV is ineffective against bespoke C2 payloads; detection must pivot to heuristic network analysis and Permission Combination Risk Scoring.

---
*Author: Syed Raffey Ali | Portfolio Project for Threat Intelligence & Offensive Security Roles*
