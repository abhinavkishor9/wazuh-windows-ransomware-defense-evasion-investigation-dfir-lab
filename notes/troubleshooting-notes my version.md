# Troubleshooting Notes

## Issue 1 – Defender Protection Features Were Already Disabled

### Observation

The baseline showed:

```text
RealTimeProtectionEnabled : False
BehaviorMonitorEnabled    : False
IoavProtectionEnabled     : False
```

### Interpretation

These values existed before the laboratory Defender exclusion was added.

### Resolution

Treat them as pre-existing endpoint state.

Do not attribute those disabled protections to the current lab.

---

## Issue 2 – Existing Defender Exclusion Was Already Present

### Observation

Before the lab modification:

```text
C:\CompromisedWorkstationLab
```

was already present in the Defender exclusion list.

### Resolution

Record it as a pre-existing exclusion.

The new controlled change was:

```text
C:\RansomwareDefenseLab
```

---

## Issue 3 – Event ID 5007 Appeared Before the Recorded Get-Date Output

### Observation

The Defender Event ID 5007 shown in the evidence was logged at:

```text
19-08-2026 12:20:42
```

while the recorded `Get-Date` output showed:

```text
19 August 2026 12:21:37
```

### Interpretation

The two timestamps are not necessarily contradictory. Event Viewer timestamps and console collection time represent different observations.

### Resolution

Use the Defender event's own timestamp when documenting the configuration change and use `Get-Date` only as a separate analyst reference point.

---

## Issue 4 – Windows Recovery Environment Was Disabled

### Observation

`reagentc /info` showed:

```text
Windows RE status: Disabled
```

### Resolution

Do not claim that the lab disabled Windows RE.

The command was used only to inspect the existing state.

Document it as:

```text
Observed pre-existing recovery state
```

---

## Issue 5 – No Shadow Copies Were Found

### Observation

`vssadmin list shadows` returned:

```text
No items found that satisfy the query.
```

### Resolution

Do not interpret this as proof that ransomware deleted Shadow Copies.

The lab did not execute a Shadow Copy deletion command.

Document it as:

```text
No Shadow Copies were present when inspected.
```

---

## Issue 6 – Sysmon Event ID 3 Showed Zoho Mail

### Observation

The captured network event belonged to:

```text
Zoho Mail - Desktop.exe
```

### Interpretation

This was unrelated endpoint network activity.

### Resolution

Exclude it from the ransomware activity chain unless additional evidence connects it to the investigation.

---

## Issue 7 – Sysmon Event ID 1 Showed Phone Link

### Observation

The captured Sysmon Event ID 1 showed:

```text
PhoneExperienceHost.exe
```

### Interpretation

This is normal background process activity.

### Resolution

Do not include the event as direct evidence of ransomware execution.

---

## Issue 8 – Harmless Archive Could Look Like Staging

### Observation

`TestArchive.zip` was created from the test files.

### Interpretation

Archive creation can resemble data staging in a real incident.

### Resolution

Use context.

In this lab:

- Files were harmless.
- The archive was intentionally created.
- No data was exfiltrated.
- No encryption was performed.

Therefore, it was controlled staging simulation.

---

## Issue 9 – Wazuh Search May Return Large Volumes of Data

### Problem

Sysmon generated many events on the workstation.

### Resolution

Start with:

```text
agent.name: DESKTOP-9MMM37V
```

Then narrow the time range.

Useful terms include:

```text
5007
```

```text
RansomwareDefenseLab
```

```text
powershell.exe
```

```text
vssadmin
```

```text
reagentc
```

Use only fields that actually exist in the Wazuh event.

---

## Issue 10 – Ransomware Was Not Actually Executed

### Observation

The lab did not encrypt files or destroy recovery data.

### Resolution

Do not describe the workstation as infected with ransomware.

The correct terminology is:

```text
Controlled ransomware defense-evasion simulation
```

This distinction is important for accurate DFIR documentation.

---

