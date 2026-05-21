# RAT Lifecycle (high-level)

Stylized Mermaid diagram showing the RAT bootstrap, persistence, and surveillance modules.

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
