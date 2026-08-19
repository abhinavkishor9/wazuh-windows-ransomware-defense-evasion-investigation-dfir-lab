# Investigation Notes

## Analyst Methodology

1. Verify Wazuh endpoint status.
2. Identify the host and active user.
3. Establish the investigation time.
4. Create controlled test workspace.
5. Create harmless test files.
6. Capture file baseline.
7. Capture Defender baseline.
8. Capture existing Defender exclusions.
9. Add a controlled Defender exclusion.
10. Correlate Defender Event ID 5007.
11. Inspect Windows Recovery Environment.
12. Inspect Shadow Copy availability.
13. Create harmless test archive.
14. Review Sysmon process activity.
15. Review Sysmon network activity.
16. Correlate evidence in Wazuh.
17. Separate relevant from unrelated activity.
18. Assess pre-encryption ransomware risk.
19. Restore the Defender configuration.
20. Clean up laboratory artifacts.

---

## Investigation Scenario

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

## Evidence Collected

### Evidence 1 – Wazuh Agent

Collected:

- Agent ID: `001`
- Agent Name: `DESKTOP-9MMM37V`
- Status: `Active`
- Operating System: Windows 11 Pro
- Client Version: `Wazuh v4.12.0`

Finding:

Confirmed that Wazuh was actively monitoring the endpoint.

---

### Evidence 2 – Endpoint Identity

Commands:

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

Finding:

Established the host, user, and investigation start time.

---

### Evidence 3 – Investigation Workspace

Created:

```text
C:\RansomwareDefenseLab
```

Observed creation time:

```text
19-08-2026 12:11
```

Finding:

Established the controlled staging directory for the investigation.

---

### Evidence 4 – Test Files

Created:

```text
TestFile1.txt
TestFile2.txt
TestFile3.txt
TestFile4.txt
TestFile5.txt
```

Each file was approximately 29 bytes.

Finding:

Created harmless test data that could be used to demonstrate data staging without performing ransomware encryption.

---

### Evidence 5 – File Baseline

Command:

```powershell
Get-ChildItem "C:\RansomwareDefenseLab" |
Select-Object Name, Length, CreationTime, LastWriteTime |
Export-Csv "C:\RansomwareDefenseLab\file-baseline.csv" -NoTypeInformation
```

Finding:

Preserved the original test-file state.

---

### Evidence 6 – Defender Baseline

Command:

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

Finding:

Several Defender protection features were already disabled before the controlled Defender exclusion was added.

This is a pre-existing security condition.

---

### Evidence 7 – Existing Defender Exclusion

Command:

```powershell
Get-MpPreference |
Select-Object ExclusionPath, ExclusionProcess, ExclusionExtension
```

Observed:

```text
C:\CompromisedWorkstationLab
```

Finding:

A Defender exclusion already existed before the ransomware-defense lab modification.

---

### Evidence 8 – Controlled Defender Change

Command:

```powershell
Add-MpPreference -ExclusionPath "C:\RansomwareDefenseLab"
```

Observed resulting exclusions:

```text
C:\CompromisedWorkstationLab
C:\RansomwareDefenseLab
```

Finding:

A new Defender exclusion was successfully added for the controlled laboratory directory.

---

### Evidence 9 – Defender Event ID 5007

Observed:

```text
Event ID: 5007
Logged: 19-08-2026 12:20:42
```

Configuration:

```text
HKLM\SOFTWARE\Microsoft\Windows Defender\Exclusions\Paths\C:\RansomwareDefenseLab
```

Finding:

Directly confirmed the Defender configuration change.

---

### Evidence 10 – Recovery Environment

Command:

```cmd
reagentc /info
```

Observed:

```text
Windows RE status: Disabled
```

Finding:

Windows Recovery Environment was disabled at the time of investigation.

No command was used in the laboratory to disable it.

Therefore, this was recorded as endpoint state rather than a simulated ransomware modification.

---

### Evidence 11 – Shadow Copies

Command:

```cmd
vssadmin list shadows
```

Observed:

```text
No items found that satisfy the query.
```

Finding:

No Volume Shadow Copies were present.

No Shadow Copies were deleted during the lab.

---

### Evidence 12 – Harmless Archive

Command:

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

Finding:

Harmless test files were staged into an archive.

This demonstrated data staging without performing encryption.

---

### Evidence 13 – Sysmon Event ID 1

Observed process:

```text
C:\Program Files\WindowsApps\Microsoft.YourPhone_1.26042.99.0_x64__8wekyb3d8bbwe\PhoneExperienceHost.exe
```

Product:

```text
Microsoft Phone Link
```

Observed timestamp:

```text
19-08-2026 12:26:18
```

Finding:

Valid Sysmon process telemetry was observed, but the process was unrelated background activity.

---

### Evidence 14 – Sysmon Event ID 3

Observed network process:

```text
C:\Users\Dell\AppData\Local\Programs\Zoho Mail - Desktop\Zoho Mail - Desktop.exe
```

User:

```text
DESKTOP-9MMM37V\Dell
```

Protocol:

```text
tcp
```

Initiated:

```text
true
```

Finding:

Valid Sysmon network telemetry was available, but the displayed connection belonged to Zoho Mail Desktop and was treated as unrelated background traffic.

---

## Evidence Correlation

| Evidence | Observation | Relevance |
|---|---|---|
| Wazuh Agent | Active | Confirms endpoint monitoring |
| Workspace | Created at 12:11 | Establishes lab start |
| Test Files | Five text files | Controlled data |
| Defender Baseline | Multiple protections already False | Pre-existing state |
| Existing Exclusion | `C:\CompromisedWorkstationLab` | Pre-existing configuration |
| New Exclusion | `C:\RansomwareDefenseLab` | Controlled security-tool change |
| Event 5007 | Logged 12:20:42 | Direct Defender change evidence |
| Windows RE | Disabled | Existing endpoint state |
| Shadow Copies | None | Recovery state |
| Test Archive | 1125 bytes | Harmless staging |
| Sysmon 1 | Phone Link | Unrelated process |
| Sysmon 3 | Zoho Mail | Unrelated network activity |

---

