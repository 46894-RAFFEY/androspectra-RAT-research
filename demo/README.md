# AndroSpectra Demo

This page is the public-facing demo for the portfolio. It keeps the evidence in one place, explains what each screenshot shows, and avoids any build steps, code, or operational instructions.

The screenshots below are from a controlled lab setup used for academic research. They are included to document what the project does at a high level and to support discussion in interviews, reviews, or academic presentations.

## What the demo shows

- A working project dashboard
- A build/configuration view for the research prototype
- A live device session in a lab environment
- A server/status view for connectivity
- A file-browsing and capture workflow
- An audio capture result view
- A module/status overview
- A session console view

## Screenshot Gallery

### 1. Unified Threat Architecture Dashboard
![Project overview](./01-overview-modules.png)
*Displays the primary operator interface, showing aggregated metrics, active payload modules, and concurrent device session states.*

### 2. Payload Configuration Engine
![Builder interface](./02-builder-tool.png)
*The lab environment configuration panel used to dynamically generate the conceptual payloads, defining C2 callback URIs and initial permission manifests.*

### 3. Active Target Session
![Connected device](./03-connected-device.png)
*Demonstrates a successful heartbeat beacon from the Android 15 AVD target, establishing the secure asynchronous session.*

### 4. Reverse Proxy & Network Listener
![Server listener](./04-server-listener.png)
*The backend terminal view capturing incoming mTLS handshakes and payload extraction logs from the target.*

### 5. Remote File System Enumeration
![File browser view](./05-file-browser.png)
*Demonstrates the `MediaStore` and scoped-storage enumeration logic, allowing the operator to parse and stage local files for exfiltration.*

### 6. Environmental Audio Capture
![Audio capture view](./06-audio-capture.png)
*Visualizes the successful exfiltration and server-side rendering of audio captured via the theoretical abuse of the `MediaRecorder` API.*

### 7. Capability Routing Dashboard
![Module dashboard](./07-module-dashboard.png)
*The granular control matrix used to toggle specific persistence mechanisms and data-collection sub-routines on the infected endpoint.*

### 8. Interactive Shell Interface
![Session console](./08-session-shell.png)
*A conceptual reverse-shell interface simulating remote command execution within the application's local sandbox.*

---
*Note: All data visible in these screenshots is synthetically generated lab data.*
