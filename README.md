# AndroSpectra — Android Security Research

> Final Year Project | Riphah International University | 2026
> 
> **Documentation only — no source code, binaries, or build instructions are included in this repository.**

---

## Project Overview

AndroSpectra is an academic research project built as my Final Year Project for a Bachelor's in Cybersecurity. The goal was to study how Android malware operates at a technical level — specifically how remote access trojans are structured, how C2 communication works, and what Android 15 security mechanisms exist to detect or block such threats.

The project was built entirely in an isolated lab environment under university supervision. All testing was performed on emulators and personal test devices with no third parties involved.

---

## What This Repo Contains

- High-level architecture documentation (C2 server, builder, and module design)
- Module capability overview (what each module does conceptually)
- Research methodology and development approach
- Defensive analysis — how each capability maps to a detection strategy
- Demo screenshots of the C2 dashboard and builder interface

## What Is NOT Here

- Source code of any kind
- APK files or compiled binaries
- Build instructions or deployment guide
- Signing keys or credentials
- Payload injection scripts

---

## System Architecture (High Level)

The research project consisted of three components built and studied in the lab:

```
┌─────────────────────────────────────────────────────┐
│                  AndroSpectra System                 │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────────┐    ┌──────────────────────────┐  │
│  │   Builder    │    │      C2 Dashboard        │  │
│  │              │    │                          │  │
│  │ • APK select │    │ • Device management      │  │
│  │ • Payload    │    │ • Command dispatch       │  │
│  │   injection  │    │ • Module control panel   │  │
│  │ • Signing    │    │ • Live feed view         │  │
│  └──────────────┘    └──────────────────────────┘  │
│          │                      │                   │
│          └──────────┬───────────┘                   │
│                     ▼                               │
│          ┌──────────────────────┐                   │
│          │    C2 Server Core    │                   │
│          │  • Auth & sessions   │                   │
│          │  • Command queue     │                   │
│          │  • Module router     │                   │
│          │  • Encrypted comms   │                   │
│          └──────────────────────┘                   │
│                     │                               │
│                     ▼                               │
│          ┌──────────────────────┐                   │
│          │   Android Payload    │                   │
│          │  (lab device only)   │                   │
│          │  • Persistent agent  │                   │
│          │  • Module executor   │                   │
│          │  • Encrypted beacon  │                   │
│          └──────────────────────┘                   │
└─────────────────────────────────────────────────────┘
```

---

## Module Capability Overview

The following table lists the modules researched and built for this project. Each entry includes the defensive countermeasure that would detect or block it — this was a core part of the research.

| Module | What It Does (Conceptual) | Android 15 Detection / Defence |
|---|---|---|
| Reverse Shell | Spawns a remote interactive shell session | Restricted non-SDK interfaces; Play Protect flags |
| Persistence | Survives reboots via boot receivers | Android 14+ restricts implicit broadcast receivers |
| Microphone | Captures ambient audio | Permission prompt; mic indicator dot (Android 12+) |
| Camera | Captures images/video | Permission prompt; camera indicator dot (Android 12+) |
| SMS Reader | Reads sent/received messages | `READ_SMS` permission; declared in manifest, visible to user |
| SMS Sender | Sends messages silently | `SEND_SMS` restricted; carrier filters on bulk sends |
| Call Log | Reads call history | `READ_CALL_LOG` permission; Privacy Dashboard (Android 12+) |
| Contact Dump | Exports device contacts | `READ_CONTACTS` permission; Privacy Dashboard |
| Location (GPS) | Precise location tracking | Precise location permission; background location needs separate grant |
| Location (Network) | Approximate location via cell/WiFi | Approximate location permission; Privacy Dashboard |
| File Browser | Lists and downloads files | Scoped Storage (Android 10+) restricts broad access |
| Keylogger | Captures typed input | Accessibility Service restrictions; Play Protect detection |
| Clipboard Monitor | Reads clipboard content | Android 10+ notifies user when clipboard is read |
| Screen Capture | Records screen | Media Projection API requires user consent dialog |
| Notification Reader | Reads notifications from other apps | Notification Listener Service requires explicit grant |
| App List | Lists installed applications | `QUERY_ALL_PACKAGES` permission; restricted from Play |
| WiFi Scanner | Scans nearby networks | Fine location permission required for WiFi scan |
| Network Info | Reads IP, MAC, network state | `ACCESS_NETWORK_STATE`; MAC randomised by default (Android 10+) |
| System Info | Dumps device model, OS, build info | No permission required; low sensitivity |
| Battery/Sensor | Reads battery state and sensors | No permission required; useful for sandbox detection |
| APK Injection | Injects payload into existing APK | Play Protect; signature verification; APK integrity check |
| Encrypted C2 Channel | Communicates with server over encrypted channel | Certificate pinning bypass is detectable; TLS inspection |

---

## Research Methodology

### Phase 1 — Threat Modelling
Mapped the capability set of known Android RAT families (AhMyth, SpyNote, DroidJack) to understand common patterns and architectural approaches before building anything.

### Phase 2 — Environment Setup
Built an isolated lab: VirtualBox host, Android 15 emulator, custom C2 server on a private network. No internet connectivity to the payload device at any point.

### Phase 3 — Component Development
Developed in stages: C2 server first, then the builder tool, then individual modules, then payload persistence. Each module was tested individually before integration.

### Phase 4 — Detection Analysis
For each module, I documented which Android permission, system mechanism, or security tool would detect or block it. This formed the defensive analysis section of the FYP report.

### Phase 5 — Report & Documentation
Produced a full academic report covering architecture, threat model, MITRE ATT&CK for Mobile mapping, and recommended detection strategies for blue teams.

---

## MITRE ATT&CK for Mobile Mapping (Selected)

| Tactic | Technique | Module |
|---|---|---|
| Collection | T1533 — Data from Local System | File Browser, Contact Dump |
| Collection | T1429 — Audio Capture | Microphone |
| Collection | T1512 — Video Capture | Camera |
| Collection | T1430 — Location Tracking | GPS / Network Location |
| Collection | T1636.004 — Contact List | Contact Dump |
| Command & Control | T1437 — Application Layer Protocol | Encrypted C2 Channel |
| Persistence | T1624 — Event-Triggered Execution | Boot Receiver |
| Credential Access | T1417 — Input Capture | Keylogger |
| Discovery | T1418 — Software Discovery | App List |

---

## Defensive Recommendations (Blue Team)

Based on building and studying this system, these are the most effective Android security controls against RAT-style malware:

1. **Enable Play Protect** — catches most known RAT signatures and suspicious permission combinations
2. **Audit app permissions regularly** — Accessibility Service and Notification Listener grants are high-risk
3. **Use the Privacy Dashboard** (Android 12+) — shows a timeline of which apps accessed mic, camera, and location
4. **Avoid sideloading APKs** — the injection technique only works on apps installed from untrusted sources
5. **Enterprise MDM enforcement** — restricts installation sources and can remotely wipe devices
6. **Network monitoring** — beacon traffic from a RAT has recognisable patterns (regular intervals, small encrypted packets)
7. **Certificate pinning** — makes C2 communication harder to establish on network-inspected environments

---

## Demo

See [demo/README.md](demo/README.md) for screenshots of the C2 dashboard and builder interface.

---

## Ethical Notice

See [ETHICAL_NOTICE.md](ETHICAL_NOTICE.md) for the full statement on responsible disclosure, permitted use, and the academic context of this project.

---

## About

- **Institution:** Riphah International University
- **Degree:** Bachelor of Science in Cybersecurity
- **Year:** 2026
- **Supervisor:** [University Supervisor — name withheld for privacy]
- **Status:** Completed FYP — documentation only released publicly

---

*This repository demonstrates research capability, not operational capability. It is intended for review by employers, academic peers, and the security research community.*
