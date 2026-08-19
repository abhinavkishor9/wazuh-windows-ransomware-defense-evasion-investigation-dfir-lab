# Investigation Timeline

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
