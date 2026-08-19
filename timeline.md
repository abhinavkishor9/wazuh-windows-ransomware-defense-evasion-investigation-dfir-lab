# Investigation Timeline

## Lab 55 – Ransomware Defense Evasion Investigation

| Time | Activity | Evidence | Assessment |
|---|---|---|---|
| 09:48 baseline window | Wazuh / endpoint preparation | Wazuh | Endpoint monitoring established |
| 12:10:55 | Host and user identified | PowerShell | Investigation start |
| 12:11 | Lab directory created | PowerShell | Controlled workspace |
| 12:12 | Five test files created | Filesystem | Harmless test data |
| 12:12 | File baseline captured | PowerShell | Before-state evidence |
| Before 12:20:42 | Defender baseline captured | PowerShell | Pre-existing security state |
| Before 12:20:42 | Existing Defender exclusion observed | PowerShell | Pre-existing configuration |
| 12:20:42 | Defender Event ID 5007 | Defender | `C:\RansomwareDefenseLab` added as exclusion |
| 12:21:37 | Analyst reference time captured | PowerShell | Investigation reference point |
| 12:26:18 | Sysmon Event ID 1 | Sysmon | Phone Link background process |
| 12:28 | Harmless archive created | PowerShell | Controlled data staging |
| 12:27–12:42 | Sysmon Event ID 3 activity | Sysmon | Network telemetry; displayed event related to Zoho Mail |

---

# Investigation Flow

```text
Wazuh Verification
        ↓
Endpoint Identification
        ↓
Workspace Creation
        ↓
Test Data Creation
        ↓
File Baseline
        ↓
Defender Baseline
        ↓
Existing Exclusion Review
        ↓
Controlled Defender Change
        ↓
Event ID 5007
        ↓
Recovery State Inspection
        ↓
Shadow Copy Inspection
        ↓
Harmless Data Staging
        ↓
Sysmon / Wazuh Correlation
        ↓
Pre-Encryption Assessment
```

---

# Initial Baseline

### 12:10:55

Endpoint:

```text
DESKTOP-9MMM37V
```

User:

```text
desktop-9mmm37v\dell
```

---

### 12:11

Investigation workspace created:

```text
C:\RansomwareDefenseLab
```

---

### 12:12

Five test files created:

```text
TestFile1.txt
TestFile2.txt
TestFile3.txt
TestFile4.txt
TestFile5.txt
```

A file baseline was captured.

---

# Defender Baseline

The Defender baseline showed:

```text
AMServiceEnabled          : True
AntivirusEnabled          : True
RealTimeProtectionEnabled : False
BehaviorMonitorEnabled    : False
IoavProtectionEnabled     : False
AntispywareEnabled        : True
```

An existing exclusion was also present:

```text
C:\CompromisedWorkstationLab
```

These are recorded as pre-existing conditions.

---

# Defense-Evasion Timeline

## 12:20:42

Microsoft Defender Event ID 5007 recorded:

```text
HKLM\SOFTWARE\Microsoft\Windows Defender\Exclusions\Paths\C:\RansomwareDefenseLab
```

This is the strongest directly confirmed defense-evasion event in the laboratory.

---

# Recovery Timeline

`reagentc /info` showed:

```text
Windows RE status:
Disabled
```

Assessment:

This was observed system state.

No recovery configuration change was performed in the lab.

---

# Shadow Copy Timeline

`vssadmin list shadows` returned:

```text
No items found that satisfy the query.
```

Assessment:

No Shadow Copies were present at the time of inspection.

No Shadow Copy deletion was performed.

---

# Data-Staging Timeline

At approximately:

```text
12:28
```

the test files were compressed into:

```text
C:\RansomwareDefenseLab\TestArchive.zip
```

The archive was:

```text
1125 bytes
```

Assessment:

Harmless staging simulation.

No encryption was performed.

---

# Sysmon Timeline

## Event ID 1

Observed process:

```text
PhoneExperienceHost.exe
```

Product:

```text
Microsoft Phone Link
```

Observed around:

```text
12:26:18
```

Assessment:

Normal background process activity.

---

## Event ID 3

Observed process:

```text
Zoho Mail - Desktop.exe
```

Protocol:

```text
tcp
```

Initiated:

```text
true
```

Assessment:

Background network activity unrelated to the controlled ransomware-defense simulation.

---

# Evidence Correlation

```text
Pre-existing Defender State
          ↓
Harmless Test Data
          ↓
Controlled Defender Exclusion
          ↓
Defender Event 5007
          ↓
Recovery State Review
          ↓
Shadow Copy Review
          ↓
Harmless Archive
          ↓
Sysmon / Wazuh Review
```

---

# Final Timeline Assessment

The timeline establishes a controlled ransomware defense-evasion scenario centered on a Microsoft Defender exclusion.

The evidence does not establish:

```text
File Encryption
Shadow Copy Deletion
Recovery Environment Modification
External C2
Actual Ransomware Execution
```

The most reliable incident indicator is the Defender Event ID 5007 showing:

```text
C:\RansomwareDefenseLab
```

being added as a Defender exclusion.

---

# Final Analyst Conclusion

The investigation demonstrates a useful **pre-encryption detection model**.

The workstation showed a controlled security-configuration change and harmless data-staging activity, while recovery state and network/process telemetry were reviewed for additional indicators.

The correct forensic conclusion is:

**Controlled ransomware defense-evasion simulation; no actual ransomware impact or recovery-destruction activity was demonstrated.**
