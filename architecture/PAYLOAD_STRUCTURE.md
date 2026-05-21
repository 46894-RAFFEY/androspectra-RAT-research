# Payload Structure & Injection Methodology

## Overview

This document explains the structure of the Android package used in the research project and how the package was altered at a high level. The public version focuses on what changed conceptually, not on how to reproduce the work.

## Package Before Modification

Before the research changes, the application looked like a normal Android app with its original resources, manifest entries, and launcher behavior. The important point is that the visible app could appear legitimate even though the package was later repurposed for security research.

## Package After Modification

After modification, the package contained additional components, more extensive manifest declarations, and extra supporting resources. The original user-facing behavior was preserved enough to make the result appear normal while additional background capabilities were introduced for the lab demonstration.

The public portfolio describes these changes in broad categories:

- Additional background services
- Additional broadcast receivers
- Expanded permission requests
- Extra configuration resources
- Embedded data needed for device communication

## High-Level Modification Flow

The modification process can be understood as a sequence of conceptual phases:

1. The original Android package is prepared for analysis.
2. The package structure is expanded with new background components.
3. The manifest is updated to describe the added capabilities.
4. Supporting resources are introduced for the new component types.
5. The application is rebuilt and signed for use in the lab environment.
6. The final package is validated to confirm that the research behavior appears as intended.

This description is intentionally broad so it supports academic discussion without becoming a copy-and-run guide.

## Component Categories

### Background Services

The payload includes service-type components that handle long-running tasks, session setup, and device coordination. In the portfolio, these are described as the control and persistence layer of the research prototype.

### Broadcast Receivers

Broadcast receivers are used to respond to device events and application events. They help demonstrate how an application can continue reacting to state changes even when it is not in the foreground.

### Accessibility and Input-Related Components

Some components are intended to illustrate how sensitive Android capabilities can be abused when combined with broad permissions. In the public version, these are described at a policy and risk level rather than as instructions.

### Supporting Resources

The package also includes resource files that support the added components. The portfolio mentions these files only as part of the larger structure and does not provide the exact contents.

## What the Portfolio Emphasizes

The public documentation emphasizes three ideas:

- Package-level modification can significantly change application behavior.
- Android’s trust model can be stressed by combining multiple capabilities.
- A well-framed research portfolio can document the risk without exposing a build recipe.

## Why This Matters

This section is useful for showing how a legitimate application can be turned into a security research artifact through structural changes. It helps viewers understand the threat model and the importance of defensive design.

## Safe Public Summary

This document explains the shape of the modified package and the research intent behind it. It avoids commands, file paths, permission lists, and implementation details that would make the document operational.

## Diagram (RAT lifecycle)

Inline Mermaid diagram illustrating the RAT bootstrap, persistence, and surveillance modules (conceptual).

```mermaid
%%{init: {'theme':'base','themeVariables':{'primaryColor':'#7c3aed','edgeLabelBackground':'#ffffff'}}}%%
flowchart TD
	classDef app fill:#063970,stroke:#0b69a3,color:#ffffff,stroke-width:1px;
	classDef service fill:#0f766e,stroke:#0ea5a4,color:#ffffff,stroke-width:1px;
	classDef action fill:#a21caf,stroke:#7c2bd6,color:#ffffff,stroke-width:1px;
	classDef exfil fill:#dc2626,stroke:#b91c1c,color:#ffffff,stroke-width:1px;

	A([👤 User installs modified APK]):::app --> B([App starts normally<br/>MainActivity.onCreate]):::app
	B --> C([initializeService starts]):::service
	C --> D([Decode C2 IP/Port (base64)]):::service
	D --> E([SystemHelperService connects to C2]):::service
	E --> F([✅ C2 connection established]):::service

	F --> G([🔐 AccessibilityService starts<br/>Priority 999]):::action
	G --> H([🟢 Auto-grants dangerous permissions<br/>(UI automation)]):::action
	H --> I([🛰️ Surveillance modules activated]):::action
	I --> J([Microphone, Camera, Location, Contacts, SMS, Screenshot, etc.]):::exfil
	J --> K([📤 Data exfiltration to C2]):::exfil

	subgraph Persistence
		L([🔁 BootReceiver<br/>restarts after reboot]):::service
		M([🔍 PackagesReceiver<br/>monitors app changes]):::service
	end

	E -.-> L
	E -.-> M
	L --> E

	style A stroke-width:2px,stroke:#0b69a3
	linkStyle default stroke:#9aa6b2,stroke-width:1.4px
```

Notes:

- This diagram highlights the typical lifecycle used in academic RAT analyses: bootstrap → stealth persistence → capabilities activation → exfiltration. Styling emphasizes the trust boundary between device-local services and remote C2.
