# Threat Model & Security Analysis

## Executive Summary

AndroSpectra demonstrates a complete attack chain from APK injection through full device compromise. This document models the threat using STRIDE methodology and analyzes the attack surface, exploitation techniques, and defense mechanisms.

---

## STRIDE Threat Model Analysis

### S - Spoofing

**Threat**: Malware masquerades as legitimate game application

**Attack Vector**:
```
Legitimate-looking APK (Flappy Bird)
    └─ Contains malicious services in manifest
       └─ Services appear as "System Helper" in notifications
          └─ Users believe it's normal app functionality
```

**Exploitation**:
- App icon identical to legitimate game
- Launcher activity preserved (game still playable)
- Services hidden with generic names
- Notifications labeled "System Helper" or "Background Task"

**Impact**: Users unknowingly install malware thinking it's a game

**Mitigation**:
- ✓ Code signing (but uses debug key, attackable)
- ✓ Manifest analysis tools
- ✓ Security scanner detection
- ✓ User education

---

### T - Tampering

**Threat 1**: Modification of app during delivery

**Attack Vector**:
- Man-in-the-middle (MITM) during download
- Malicious store/repository
- Compromised distribution server

**Threat 2**: In-device tampering after installation

**Attack Vector**:
```
Payload execution
    └─ Accessibility service gains system access
       └─ Can modify system settings
          └─ Can disable security features
             └─ Can tamper with other apps
```

**Exploitation**:
- AccessService (priority 999) modifies system behavior
- Can disable other security apps via UI automation
- Can inject content into WebViews
- Can modify app data via file access

**Impact**: System integrity compromise, spreading to other apps

**Mitigation**:
- ✓ HTTPS for downloads (not shown here)
- ✓ Signature verification
- ✓ SELinux enforcement
- ✓ Integrity checking tools

---

### R - Repudiation

**Threat**: Attacker denies involvement in malware deployment

**Attack Vector**:
```
APK builder → Customized payload → Deployed to victim
    └─ No logs tying builder to deployment
    └─ Payload configured via GUI (no code)
    └─ Multiple distribution channels possible
```

**Exploitation**:
- GUI-based APK customization (no audit trail)
- C2 credentials embedded in binary
- Multiple possible attackers could use same tool
- Device logs show only installed app, not origin

**Impact**: Attribution is impossible (plausible deniability)

**Mitigation**:
- ✓ App store logging
- ✓ Device logs with installation source
- ✓ Network forensics on C2 communication
- ✓ Certificate analysis

---

### I - Information Disclosure

**Threat**: Unauthorized access to sensitive device data

**Attack Vectors** (Multiple simultaneous):

1. **Contact Theft**
   ```
   Malware → ContentProvider query → All contacts
   └─ Includes names, phone numbers, emails
   └─ No per-contact granular permissions
   └─ Exported silently to C2
   ```

2. **SMS Interception**
   ```
   Malware → SMS ContentProvider → All messages
   └─ Includes 2FA codes, banking OTPs
   └─ Complete message history
   └─ Sent/received distinction preserved
   ```

3. **Call Log Disclosure**
   ```
   Malware → CallLog database → All calls
   └─ Phone numbers, durations, timestamps
   └─ Reveals communication patterns
   └─ Identifies frequently-called contacts
   ```

4. **Location Tracking**
   ```
   Device GPS/Network → LocationService
   └─ Continuous geolocation
   └─ Movement patterns revealed
   └─ Home/work locations identified
   ```

5. **Visual Data**
   ```
   Device screen → MediaProjection → Screenshots
   └─ Password entry screens captured
   └─ Banking app interfaces visible
   └─ Private conversations revealed
   ```

6. **Audio Data**
   ```
   Device microphone → MediaRecorder → WAV files
   └─ Ambient sounds captured
   └─ Phone call audio recorded
   └─ All conversations accessible
   ```

7. **File System Access**
   ```
   /storage/emulated/0/ → Full directory traversal
   └─ WhatsApp private data
   └─ Browser history and cookies
   └─ App cache files
   └─ Private documents
   ```

**Impact Scope**:
- Complete privacy violation
- Financial account compromise (banking apps)
- Identity theft (contacts + messages)
- Relationship mapping (communication patterns)
- Behavioral profiling (location history)

**Mitigation**:
- ✓ Data encryption at rest (rarely enabled)
- ✓ Permission granularity (limited by design)
- ✓ Runtime permission checks (not re-verified)
- ✓ Content query logging (minimal in practice)

---

### D - Denial of Service

**Threat 1**: Resource exhaustion

**Attack Vectors**:
```
Microphone recording 24/7
    └─ ~500MB per hour
    └─ Fill device storage
    └─ Battery drain to 0%
    └─ Device unusable
```

```
Location tracking @ 100ms intervals
    └─ GPS keeps screen on
    └─ Battery drain 50%+ per hour
    └─ Network saturation
    └─ Lag/freeze
```

```
Screenshot capture every 1 second
    └─ CPU usage 80%+
    └─ Thermal throttling
    └─ Screen becomes unresponsive
```

**Impact**: Device rendered unusable by user

**Threat 2**: Service disruption

**Attack Vectors**:
```
AccessService clicks "disable" on security apps
    └─ Device protection removed
    └─ Vulnerability to other malware
    └─ No user awareness
```

**Mitigation**:
- ✓ Resource monitoring
- ✓ Task management limiting services
- ✓ Battery optimization (but can be disabled)
- ✓ Thermal monitoring

---

### E - Elevation of Privilege

**Threat**: Gain system-level access beyond app permissions

**Attack Chain**:

```
Level 0: App installed
    ├─ Standard app permissions only
    └─ Isolated from other apps (sandbox)
        ↓
Level 1: Manifest permissions granted at install
    ├─ RECORD_AUDIO, READ_CONTACTS, etc.
    ├─ User barely reviews list
    └─ 31 permissions in single dialog
        ↓
Level 2: Accessibility Service enabled
    ├─ User manually enables or AccessService auto-enables via Level 1
    ├─ Service runs with highest priority (999)
    └─ Can now intercept/automate any UI action
        ↓
Level 3: Auto-grant remaining permissions
    ├─ AccessService detects permission dialogs
    ├─ Simulates "Allow" button clicks
    ├─ User never sees permission requests
    └─ All sensitive permissions now granted
        ↓
Level 4: Device Admin (optional)
    ├─ AccessService clicks "Activate Device Admin"
    ├─ Can now prevent uninstallation
    ├─ Can reset device if discovered
    └─ Full control over device lifecycle
        ↓
Level 5: System Compromise Complete
    ├─ All 23+ modules operational
    ├─ All data accessible
    ├─ All capabilities operational
    └─ User has no way to disable without uninstall
```

**Key Insight**: Elevation happens incrementally without user awareness

**Mitigation**:
- ✓ Prevent Accessibility Service abuse (separate permission per action)
- ✓ Re-verify permissions during runtime
- ✓ Monitor AccessibilityService actions
- ✓ Better permission UI (show true impact)

---

## Attack Surface Analysis

### Entry Points (Ways Malware Reaches Device)

| Vector | Difficulty | Detection | Examples |
|--------|-----------|-----------|----------|
| **Fake App Store** | Easy | Low | "Flappy Bird 2" on sketchy stores |
| **Direct APK Distribution** | Easy | Medium | Email, SMS, messaging apps |
| **Compromised Supply Chain** | Medium | Low | Legitimate store hacked |
| **Social Engineering** | Medium | Low | SMS with download link |
| **Drive-by Download** | Hard | Medium | Malicious website serving APK |
| **Fake Updater** | Medium | Low | "System Update" notification |
| **Trojanized Game** | Easy | Very Low | Hidden in legitimate game APK |

---

### Exploitation Phases

```
Phase 1: Installation
├─ User downloads from app store/email/SMS
├─ Appears to be legitimate game
├─ Requests 31 permissions in single dialog
└─ User approves without reading

Phase 2: Initial Execution
├─ MainActivity.onCreate() runs
├─ Hooks initializeService
├─ Game loads and runs normally
└─ Malware silently initializes in background

Phase 3: Initialization
├─ initializeService extracts C2 credentials
├─ Decodes base64-encoded IP/port
├─ Starts SystemHelperService
└─ Connects to C2 server

Phase 4: Privilege Escalation
├─ Services request additional permissions
├─ AccessService starts with priority 999
├─ Detects permission dialogs automatically
├─ Auto-clicks "Allow" buttons via UI automation
└─ User never sees requests

Phase 5: Persistence Establishment
├─ BootReceiver registers for BOOT_COMPLETED
├─ PackagesReceiver monitors app changes
├─ ScreenReceiver responds to user interaction
├─ MyJobService schedules periodic tasks
└─ Survives device reboots indefinitely

Phase 6: Full Capability Activation
├─ All 23+ modules now operational
├─ Microphone recording starts
├─ Location tracking begins
├─ Screenshot capture activated
├─ File system accessible
└─ Complete surveillance achieved

Phase 7: Data Exfiltration
├─ Contacts → C2 server
├─ SMS messages → C2 server
├─ Call logs → C2 server
├─ Location history → C2 server
├─ Screenshots → C2 server
├─ Microphone recordings → C2 server
└─ All accessible files → C2 server
```

---

## Vulnerability Chain (Root Causes)

### 1. Accessibility Service Architecture Flaw
**Problem**: Single permission grants system-level device access  
**Impact**: Can control entire device UI  
**Root Cause**: Designed for assistive technology, not security-aware  
**Fix**: Implement per-action permission model

### 2. Permission Auto-Grant Possibility
**Problem**: User grants once, checked never again  
**Impact**: Permissions misused beyond original scope  
**Root Cause**: Android 6.0 runtime permissions not re-verified  
**Fix**: Periodic re-verification or per-operation approval

### 3. Broadcast Receiver Always-On
**Problem**: Cannot be disabled without uninstall  
**Impact**: Persistence vector impossible to stop  
**Root Cause**: Receivers always listening for intents  
**Fix**: Allow user to disable specific receivers

### 4. Content Provider Query Unrestricted
**Problem**: No per-contact or per-message permissions  
**Impact**: All contacts/SMS/calls accessible without individual consent  
**Root Cause**: Permissions designed for all-or-nothing access  
**Fix**: Implement granular per-item permissions

### 5. Manifest Permissions Overwhelming
**Problem**: 31 permissions shown at once, user doesn't review  
**Impact**: User approves dangerous permissions unknowingly  
**Root Cause**: Grouped as single approval dialog  
**Fix**: Step-by-step permission introduction or clearer UI

### 6. No Foreground Service Restriction
**Problem**: Apps can run services indefinitely  
**Impact**: Malware cannot be easily suspended  
**Root Cause**: Services designed for legitimate background work  
**Fix**: Timeout or user-initiated suspension options

---

## Detection Methods & Evasion

### Static Analysis Detection

**Method**: Scan APK for suspicious patterns

**Detected by**:
- String scanning for known C2 IPs
- Permission pattern analysis (31+ permissions unusual)
- Service manifest inspection
- Accessibility service registration

**Evasion Used**:
- Base64 encoding of C2 credentials
- Randomized class names
- Generic service names ("System Helper")
- Multiple services (hard to see as coordinated)

**Evasion Effectiveness**: ✓ Moderate (defeats casual scanners)

---

### Dynamic Analysis Detection

**Method**: Monitor runtime behavior

**Monitored by**:
- ContentProvider query logging
- File access monitoring
- GPS location querying
- Microphone access tracking
- Network traffic inspection

**Evasion Used**:
- Timing attacks (activity during normal app use)
- Behavioral blending (appears as normal game)
- Encrypted C2 communication (not shown in this implementation)

**Evasion Effectiveness**: ✓ Moderate (defeats many heuristics)

---

### Permission-Based Detection

**Method**: Alert on suspicious permissions

**Red Flags**:
- RECORD_AUDIO + INTERNET (audio exfiltration)
- LOCATION + INTERNET (continuous tracking)
- ACCESSIBILITY + DATA_PERMISSIONS (privilege escalation)
- BOOT_COMPLETED + FOREGROUND_SERVICE (persistence)

**This Malware's Pattern**:
- Has ALL suspicious combinations
- 31 total permissions is extremely suspicious
- Multiple services for same function (coordinated attack)

**Detection Effectiveness**: ✓ Very High (pattern is obvious)

**Why Malware Includes It Anyway**:
- User doesn't read permission list
- App store warnings insufficient
- Visible in manifests only to security professionals

---

## Defense Layers & Bypass Methods

### Layer 1: Permission Model
**Defense**: User grants only needed permissions  
**Bypass**: Get all permissions at install time  
**Result**: ✓ BYPASSED

### Layer 2: Runtime Permission Checks
**Defense**: OS re-verifies permissions before access  
**Bypass**: Permissions checked once at grant time, not re-verified  
**Result**: ✓ BYPASSED

### Layer 3: Accessibility Service Restrictions
**Defense**: Service designed for accessibility, not general control  
**Bypass**: AccessService can automate any UI action via accessibility APIs  
**Result**: ✓ BYPASSED

### Layer 4: SELinux Enforcement
**Defense**: Kernel-level mandatory access control  
**Bypass**: Malware operates within app sandbox (SELinux allows app operations)  
**Result**: ✓ BYPASSED (within design parameters)

### Layer 5: App Isolation
**Defense**: Apps cannot interact with other apps  
**Bypass**: ContentProviders are explicit inter-app communication mechanism  
**Result**: ✓ BYPASSED (by design, for legitimate sharing)

### Layer 6: Manifest Validation
**Defense**: Android checks manifest during install  
**Bypass**: Manifest is plaintext XML, no limit on permissions  
**Result**: ✓ BYPASSED (manifests accepted as-is)

### Layer 7: Signature Verification
**Defense**: APK must be signed by trusted key  
**Bypass**: Debug keystore is publicly available  
**Result**: ✓ BYPASSED (debug signing accepted)

---

## Impact Assessment

### Data Compromise Scope
```
High Confidentiality Impact:
├─ All contacts (names, numbers, emails)
├─ All SMS messages (including 2FA codes)
├─ All call logs (communication patterns)
├─ Location history (movement patterns)
├─ Screen content (visual data, passwords)
├─ Audio recordings (ambient + calls)
├─ All accessible files (documents, photos)
└─ Banking/email credentials (if visible on screen)

High Integrity Impact:
├─ Can modify files (via shell)
├─ Can send SMS (via SendSMS permission)
├─ Can make calls (via CallPhone permission)
├─ Can change WiFi settings
├─ Can disable lock screen
└─ Can uninstall other apps

High Availability Impact:
├─ Battery drain (continuous services)
├─ Storage exhaustion (large recordings)
├─ Network saturation (data exfiltration)
├─ Processing power consumed (screen capture)
└─ Device becomes unusable
```

### Business Impact (Enterprise)
- **Intellectual Property**: All files accessible
- **Financial Risk**: Banking credentials, payment info
- **Customer Data**: Contacts + info from accessed apps
- **Regulatory**: GDPR/CCPA violations for data handling
- **Reputational**: Data breach if user info exposed

### Personal Impact (Individual)
- **Privacy Invasion**: Complete surveillance
- **Identity Theft**: Personal information exposed
- **Financial Loss**: Unauthorized transactions
- **Reputation**: Private data disclosed
- **Psychological**: Knowing device is compromised

---

## Incident Response Recommendations

### If Compromise Suspected
1. **Immediate Actions**
   - Disconnect device from internet (unplug charging)
   - Do not sync with cloud services
   - Avoid using banking/sensitive apps

2. **Analysis Phase**
   - Connect to isolated network
   - Enable adb debugging
   - Extract device logs
   - Monitor network traffic

3. **Containment**
   - Uninstall suspect app
   - Check for unauthorized apps
   - Disable accessibility services
   - Review installed apps for suspicious entries

4. **Recovery**
   - Factory reset device (if critical data)
   - Restore from clean backup (if available)
   - Change all passwords (on different device)
   - Monitor for identity theft

5. **Prevention**
   - Enable Google Play Protect
   - Regular security updates
   - Permission audits
   - App review before installation

---

## Security Roadmap for Android

### Short-term (Android 16+)
- [ ] Implement per-action accessibility permissions
- [ ] Add accessibility service action logging
- [ ] Pause background services on detection
- [ ] Improve permission UI clarity

### Medium-term (Android 17+)
- [ ] Per-contact/per-message granular permissions
- [ ] Content provider query auditing
- [ ] Real-time permission usage monitoring
- [ ] Accessibility service sandboxing

### Long-term (Android 18+)
- [ ] Zero-trust permission model
- [ ] Machine learning-based anomaly detection
- [ ] Hardware-enforced permission isolation
- [ ] User-controlled permission revocation

---

## Conclusion

AndroSpectra demonstrates that while Android has security mechanisms, their combination through an accessibility service abuse chain can lead to comprehensive device compromise. The key vulnerabilities are:

1. **Accessibility service** with system-wide access
2. **One-time permission grants** never re-verified
3. **Manifest overwhelm** (users don't review)
4. **Content provider design** (all-or-nothing access)
5. **No service suspension** mechanism

Rather than blaming users, improvements should focus on better architecture that doesn't give single permissions such extensive power. The research suggests Android needs a fundamental redesign of the accessibility service and permission model for post-2025 versions.

