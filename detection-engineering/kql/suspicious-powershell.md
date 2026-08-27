# Suspicious PowerShell Activity — KQL Hunting Example

## Objective
Identify PowerShell execution that warrants investigation based on command-line behavior rather than treating every PowerShell process as malicious.

## Data Source
Microsoft Defender XDR `DeviceProcessEvents`.

## Hunt Query

```kusto
DeviceProcessEvents
| where Timestamp > ago(24h)
| where FileName in~ ("powershell.exe", "pwsh.exe")
| where ProcessCommandLine has_any (
    "-enc", "-encodedcommand", "FromBase64String",
    "DownloadString", "Invoke-WebRequest", "IEX"
)
| project Timestamp, DeviceName, AccountName, FileName,
          ProcessCommandLine, InitiatingProcessFileName,
          InitiatingProcessCommandLine
| order by Timestamp desc
```

## Why This Matters
PowerShell is a legitimate administrative tool, so detecting the executable alone creates excessive noise. This hunt narrows attention to command-line patterns commonly associated with encoded execution, remote retrieval, or dynamic command execution.

## Investigation Workflow
1. Determine whether the account and device normally execute administrative PowerShell.
2. Review the full command line and parent process.
3. Check surrounding process, network, file, and identity telemetry.
4. Determine whether the script or destination is approved and expected.
5. Escalate when behavior is unexplained, obfuscated, externally retrieved, or associated with other suspicious signals.

## False Positives
Administrative scripts, software-management platforms, security products, and deployment tooling can legitimately use some of these patterns. Production detections should be baselined and tuned rather than blindly alerting on every match.

## Detection Engineering Principle
Start with a threat hypothesis, identify the telemetry that can prove or disprove it, and preserve enough context for an analyst to make a decision.
