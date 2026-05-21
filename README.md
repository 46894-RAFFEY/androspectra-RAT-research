# AndroSpectra

AndroSpectra is an academic Android security research portfolio. The public version is intentionally limited to documentation, screenshots, and high-level descriptions so it can be shared safely.

## What is included

- A short project overview
- A single demo page with embedded screenshots
- A legal and ethical notice
- High-level architecture notes
- A module summary and research framing

## What is not included

- Source code
- APKs or binaries
- Signing keys or credentials
- Build or deployment instructions
- Step-by-step operational details

## Main entry points

- [demo/README.md](demo/README.md) for the visual proof
- [ETHICAL_NOTICE.md](ETHICAL_NOTICE.md) for public-use boundaries
- [INDEX.md](INDEX.md) for a simple map of the folder

## Short project summary

This portfolio shows a controlled laboratory research project about Android security, device monitoring, and defensive analysis. It is presented as documentation only, with the emphasis on safe disclosure and professional presentation.

## Architecture Diagrams

Detailed diagrams have been added to the `architecture/` folder. They provide high-level, non-operational views of the project's Command & Control (C2) components and the RAT lifecycle:

- [C2 overview](architecture/C2.md)
- [RAT lifecycle](architecture/RAT.md)

## Module Capability Table (conceptual)

Below is a concise, conceptual list of modules discussed in the analysis (non-operational descriptions only):

| Module | Purpose |
|---|---|
| Device Registration | Register device and assign an ID to a C2 session |
| Listener | Network listener accepting device check-ins (TCP/5555) |
| Command Dispatcher | Send and route operator commands to devices |
| Module Loader | Push and manage optional payload modules |
| Persistence | Maintain presence across reboots and package changes |
| Accessibility Abuser | Uses AccessibilityService to automate UI interactions |
| Permission Manager | Decode and attempt to obtain runtime permissions |
| APK Builder | Produce modified APKs for controlled experiments |
| Exfiltration | Batched, encrypted data exfiltration to C2 |
| Microphone Capture | Record ambient audio from device microphone |
| Camera Capture | Take photos / short video from device camera |
| Location | Periodic GPS / network-based location collection |
| Screenshots | Capture screen contents for UX monitoring |
| Contacts Grabber | Harvest contact lists and metadata |
| SMS & Calllogs | Read SMS messages and call history (where allowed) |
| File Manager | Browse and exfiltrate files under scoped storage limits |
| Remote Shell | Execute commands on the device shell (limited) |
| Keylogger | Capture user input through accessibility hooks |
| Crypto & Obfuscation | Encrypt C2 traffic and obfuscate code artifacts |
| Update Manager | Remote module updates and self-updating logic |
| Anti-Analysis | Emulator detection, debugger checks, and heuristics |
| Telemetry & Health | Device health, heartbeat, and sensor telemetry |

> Note: These are conceptual capability descriptions included for academic documentation only. No operational details, code, or binaries are provided.

## Key Research Findings — Android 15 defensive perspective

High-level observations from the research and implications for defenders (academic framing):

- Android platform hardening reduces the attack surface for many classic RAT techniques: stricter background-execution limits, scoped storage, and more granulated runtime permissions make silent data access harder.
- Accessibility-based automation remains a risky vector: recent platform changes increase friction, but accessible UI automation still enables permission-granting flows and UI manipulation in some configurations.
- Persistence via broadcast receivers and package observers is detectable: monitoring for unusual `BOOT_COMPLETED` handlers, package install listeners, and unexpected background services produces high-value signals.
- Network detection is effective: beaconing, periodic check-ins, and large-scale exfiltration are amenable to anomaly detection at perimeter and EDR agents.
- Supply-chain and installer protections (Play Protect, app signing checks, attestation) significantly raise the bar for wide-scale deployment of modified APKs.
- Defensive recommendations:

	- Monitor and alert on new `AccessibilityService` registrations and services started with unusually high priority.
	- Alert on processes that request a large set of dangerous permissions in short succession or that perform UI automation-like click sequences.
	- Detect anomalous outbound connections to uncommon domains/IPs, especially regular beaconing intervals or large POST payloads.
	- Use dynamic analysis on suspicious APKs and strengthen runtime telemetry that correlates permission usage with foreground user activity.
	- Harden endpoint detection to look for boot-receiver registrations and unexpected package install listeners.

These findings are intentionally framed for academic and defensive discussion. They reflect conceptual learnings and detection guidance rather than operational advice.

