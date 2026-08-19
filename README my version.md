# wazuh-windows-ransomware-defense-evasion-investigation-dfir-lab
## Overview

Concept

Ransomware defense evasion is behavior intended to reduce the endpoint's ability to detect, prevent, recover from, or investigate ransomware activity.

A simplified attack chain can look like:

Initial Execution
      ↓
Privilege / Administrative Activity
      ↓
Defense Evasion
      ↓
Recovery Impairment
      ↓
Encryption

For this lab, we will stop at the defense-evasion stage.

We will not encrypt files, destroy backups, or disable real security controls.

Examples of ransomware defense evasion

An attacker may attempt to:

Modify Microsoft Defender configuration
Stop or interfere with security services
Modify firewall settings
Remove recovery-related configuration
Delete or interfere with Volume Shadow Copies
Clear logs
Disable backup-related mechanisms
Alter security-related policies

The important DFIR question is:

What changed on the endpoint immediately before the suspected ransomware activity?

In this controlled lab, harmless files were created inside `C:\RansomwareDefenseLab`, a Defender exclusion was added for that directory, Windows recovery configuration and Shadow Copy state were inspected, and the test files were compressed into a harmless archive.

Sysmon, Microsoft Defender Operational logs, Windows process telemetry, and Wazuh were reviewed to investigate the activity.

The lab did not encrypt files, delete Shadow Copies, disable Windows Recovery Environment, or perform destructive ransomware actions.

---

# Lab Objectives

- Establish the workstation’s recovery and security-control baseline before investigating ransomware-related activity.
- Determine whether security controls changed during the investigation window.
- Identify whether recovery resources were available at the time of assessment.
- Examine whether files were being staged or prepared for potential impact.
- Distinguish pre-existing defensive weaknesses from newly generated activity.
- Correlate configuration changes with their timestamps and surrounding process activity.
- Separate normal background network/process events from incident-relevant evidence.
- Determine whether the observed activity indicates pre-encryption behavior or only a controlled simulation.
- Document evidence limitations and avoid attributing destructive behavior that was not actually observed.

---

# Lab Environment

| Component          | Value                                      |
| ------------------ | ------------------------------------------ |
| Host OS            | Windows 11 Pro                             |
| SIEM               | Wazuh 4.12                                 |
| Endpoint Agent     | Wazuh Agent                                |
| Endpoint Name      | DESKTOP-9MMM37V                            |
| Agent ID            | 001                                        |
| Investigation Type | Ransomware Defense Evasion Investigation   |
| Lab Directory      | C:\RansomwareDefenseLab                    |
| Test Port          | 9091                                       |
| Primary Telemetry  | Defender / Sysmon / Windows / Wazuh        |

---

# Tools Used

- PowerShell
- Microsoft Defender Antivirus
- `Get-MpComputerStatus`
- `Get-MpPreference`
- `Add-MpPreference`
- `Remove-MpPreference`
- `Compress-Archive`
- `vssadmin`
- `reagentc`
- Windows Event Viewer
- Windows Defender Event ID 5007
- Sysmon Event ID 1
- Sysmon Event ID 3
- Wazuh Discover
- Wazuh Agent

---

# Investigation Scenario

A Windows workstation is being assessed for possible ransomware preparation activity after a security-control configuration change is observed.

The analyst needs to determine:

- What security setting changed?
- Was the change new or already present?
- What was the state of Windows recovery mechanisms and Shadow Copies?
- Was any data being staged?
- What processes and network activity occurred around the same period?
- Which evidence is actually related to the suspected ransomware activity?
- Does the combined evidence support a pre-encryption ransomware hypothesis?

The investigation focuses on defense-evasion indicators, recovery-state analysis, data staging, and evidence correlation before any destructive encryption occurs.

---

# Investigation Workflow

```text
Endpoint Baseline
        ↓
Create Controlled Test Data
        ↓
Review Defender State
        ↓
Modify Defender Configuration
        ↓
Review Event ID 5007
        ↓
Inspect Recovery State
        ↓
Inspect Shadow Copy State
        ↓
Stage Harmless Archive
        ↓
Review Sysmon / Wazuh
        ↓
Build Timeline
        ↓
Assess Pre-Encryption Risk
```

---

# Investigation Steps

## Step 1 – Verify Wazuh Agent

The Wazuh manager was checked using:

```bash
sudo /var/ossec/bin/agent_control -i 001
```

Observed:

- Agent ID: `001`
- Agent Name: `DESKTOP-9MMM37V`
- Status: `Active`
- Operating System: Windows 11 Pro
- Wazuh Version: `4.12.0`

Syscheck activity was also recorded for the endpoint before the laboratory activity.

---

## Step 2 – Identify the Endpoint

Commands used:

```powershell
hostname
```

```powershell
whoami
```

```powershell
Get-Date
```

Observed:

```text
DESKTOP-9MMM37V
desktop-9mmm37v\dell
19 August 2026 12:10:55
```

---

## Step 3 – Create the Investigation Workspace

```powershell
New-Item -Path "C:\RansomwareDefenseLab" -ItemType Directory -Force
```

The directory was created at approximately:

```text
19-08-2026 12:11
```

---

## Step 4 – Create Harmless Test Data

Five harmless test files were created:

```text
TestFile1.txt
TestFile2.txt
TestFile3.txt
TestFile4.txt
TestFile5.txt
```

Each file contained benign ransomware-investigation test text.

The files were created at approximately:

```text
19-08-2026 12:12
```

---

## Step 5 – Capture File Baseline

```powershell
Get-ChildItem "C:\RansomwareDefenseLab" |
Select-Object Name, Length, CreationTime, LastWriteTime |
Export-Csv "C:\RansomwareDefenseLab\file-baseline.csv" -NoTypeInformation
```

This preserved the initial file state for later comparison.

---

## Step 6 – Capture Defender Baseline

```powershell
Get-MpComputerStatus |
Select-Object `
AMServiceEnabled,
AntivirusEnabled,
RealTimeProtectionEnabled,
BehaviorMonitorEnabled,
IoavProtectionEnabled,
AntispywareEnabled
```

Observed:

```text
AMServiceEnabled          : True
AntivirusEnabled          : True
RealTimeProtectionEnabled : False
BehaviorMonitorEnabled    : False
IoavProtectionEnabled     : False
AntispywareEnabled        : True
```

Important:

The disabled Real-Time Protection, Behavior Monitoring, and IOAV Protection states were already present in the baseline and therefore were not attributed to the later laboratory Defender exclusion change.

---

## Step 7 – Capture Existing Defender Exclusions

Before modification:

```powershell
Get-MpPreference |
Select-Object ExclusionPath, ExclusionProcess, ExclusionExtension
```

Observed existing exclusion:

```text
C:\CompromisedWorkstationLab
```

The exclusion baseline was saved to:

```text
C:\RansomwareDefenseLab\exclusions-before.txt
```

---

## Step 8 – Add Controlled Defender Exclusion

The laboratory directory was added as a Defender exclusion:

```powershell
Add-MpPreference -ExclusionPath "C:\RansomwareDefenseLab"
```

The resulting configuration showed:

```text
C:\CompromisedWorkstationLab
C:\RansomwareDefenseLab
```

This was a controlled and reversible configuration change.

---

## Step 9 – Record the Change Time

```powershell
Get-Date
```

Observed:

```text
19 August 2026 12:21:37
```

This provided a reference point for event correlation.

---

## Step 10 – Review Defender Event ID 5007

Windows Defender Operational logs showed Event ID:

```text
5007
```

The important event was logged at:

```text
19-08-2026 12:20:42
```

The configuration change identified:

```text
HKLM\SOFTWARE\Microsoft\Windows Defender\Exclusions\Paths\C:\RansomwareDefenseLab
```

This directly confirms the Defender exclusion change.

---

## Step 11 – Run Harmless Administrative Commands

Commands used:

```powershell
Get-Process | Select-Object -First 10
```

and:

```powershell
Get-Service | Where-Object {$_.Status -eq "Running"} |
Select-Object -First 10
```

These commands generated normal endpoint process/service activity around the investigation window.

Examples observed included:

- Acrobat
- AdobeCollabSync
- ApplicationFrameHost
- Base Filtering Engine
- BitLocker Drive Encryption Service
- Windows Audio

These were treated as normal endpoint activity rather than ransomware indicators.

---

## Step 12 – Inspect Windows Recovery Configuration

Command used:

```cmd
reagentc /info
```

Observed:

```text
Windows RE status: Disabled
```

No change to Windows Recovery Environment was performed during the lab.

Therefore:

```text
Windows RE Disabled
```

is recorded as an endpoint state, not as evidence that the laboratory disabled it.

---

## Step 13 – Inspect Shadow Copies

Command used:

```cmd
vssadmin list shadows
```

Observed:

```text
No items found that satisfy the query.
```

No Shadow Copies were deleted during the lab.

Therefore the result establishes the observed recovery state but does not prove ransomware-related Shadow Copy deletion.

---

## Step 14 – Create Harmless Test Archive

```powershell
Compress-Archive `
    -Path "C:\RansomwareDefenseLab\*.txt" `
    -DestinationPath "C:\RansomwareDefenseLab\TestArchive.zip"
```

Observed:

```text
TestArchive.zip
Length: 1125 bytes
```

The archive was created as a harmless staging simulation.

No files were encrypted.

---

## Step 15 – Review Sysmon Event ID 1

Sysmon Event ID 1 was reviewed around the investigation window.

The captured process event showed:

```text
Image:
C:\Program Files\WindowsApps\Microsoft.YourPhone_1.26042.99.0_x64__8wekyb3d8bbwe\PhoneExperienceHost.exe

FileVersion:
1.26042.99.0

Description:
Microsoft Phone Link

Product:
Microsoft Phone Link
```

This was treated as normal endpoint background activity rather than direct evidence of ransomware behavior.

---

## Step 16 – Review Sysmon Event ID 3

The displayed Sysmon Event ID 3 showed:

```text
Image:
C:\Users\Dell\AppData\Local\Programs\Zoho Mail - Desktop\Zoho Mail - Desktop.exe

User:
DESKTOP-9MMM37V\Dell

Protocol:
tcp

Initiated:
true
```

The displayed network event was associated with Zoho Mail Desktop.

It was therefore treated as unrelated background network activity.

---

## Step 17 – Correlate Wazuh

Wazuh Discover was reviewed for:

```text
agent.name: DESKTOP-9MMM37V
```

Additional investigation searches can include:

```text
agent.name: DESKTOP-9MMM37V AND 5007
```

```text
agent.name: DESKTOP-9MMM37V AND RansomwareDefenseLab
```

```text
agent.name: DESKTOP-9MMM37V AND powershell.exe
```

Where available:

```text
agent.name: DESKTOP-9MMM37V AND vssadmin
```

and:

```text
agent.name: DESKTOP-9MMM37V AND reagentc
```

---

# Evidence Correlation

| Evidence | Source | Observation | Interpretation |
|---|---|---|---|
| Wazuh Agent | Wazuh Manager | Agent `001` active | Endpoint visibility |
| Endpoint Identity | PowerShell | `DESKTOP-9MMM37V` | Host attribution |
| Test Files | Filesystem | Five `.txt` files | Controlled test data |
| File Baseline | PowerShell | CSV created | Before-state evidence |
| Defender Baseline | PowerShell | Several protections already False | Pre-existing state |
| Existing Exclusion | Defender | `C:\CompromisedWorkstationLab` | Pre-existing configuration |
| New Exclusion | Defender | `C:\RansomwareDefenseLab` | Controlled configuration change |
| Event 5007 | Defender | Exclusion path added | Direct change evidence |
| Windows RE | `reagentc` | Disabled | Observed state, not lab modification |
| Shadow Copies | `vssadmin` | None found | Observed recovery state |
| Archive | PowerShell | `TestArchive.zip` | Harmless data staging |
| Sysmon 1 | Event Viewer | Phone Link process | Background activity |
| Sysmon 3 | Event Viewer | Zoho Mail network connection | Background activity |
| Wazuh | Discover | Endpoint telemetry | Centralized visibility |

---

# MITRE ATT&CK Context

The controlled Defender configuration change is relevant to:

- `T1562.001 – Impair Defenses: Disable or Modify Tools`

Other ransomware investigations may also involve:

- `T1490 – Inhibit System Recovery`
- `T1486 – Data Encrypted for Impact`

Those techniques were not actually performed in this laboratory.

---

