# Threat Hypothesis -- T1059.001 (PowerShell)

**MITRE ATT&CK:** T1059.001 -- Command and Scripting Interpreter: PowerShell
**Stage:** Execution
**Atomic Test:** T1059.001-17 -- PowerShell Command Execution

## Objective

Validate that PowerShell execution generates the expected endpoint telemetry and can be
reliably detected using Windows Event Logs, Sysmon, and Splunk.

This serves as the baseline for future PowerShell-based attack techniques such as encoded
commands, download cradles, AMSI bypasses, and malicious script execution.

## Real-World Context

After obtaining initial access through methods such as social engineering, malicious
attachments, or exploitation of vulnerable services, attackers frequently leverage PowerShell
because it is a trusted Windows administration utility. Rather than immediately deploying
malware, PowerShell is often used to execute commands directly in memory, download additional
payloads, perform reconnaissance, and automate post-compromise activities while blending with
legitimate administrative operations.

This simulation validates that the logging pipeline captures PowerShell activity and that
corresponding detections can be developed and tested against realistic telemetry.

## Detection Objectives

For this technique we want to answer four questions:

1. **Did Windows generate the expected telemetry?**
   Expected log sources: PowerShell Operational Log, Security Log, Sysmon.

2. **Did Splunk ingest the telemetry correctly?**
   Expected indexes: `powershell`, `sysmon`, `wineventlog`.

3. **Can we identify the execution?**
   Evidence should include: Process Image, Parent Process, Command Line, User, Host,
   Timestamp, Script Block (if applicable).

4. **Can we create a reliable detection?**
   Deliverables: SPL Hunt, Sigma Rule, Detection Logic, Validation, Tuning Notes.

## Workflow

This technique follows the standard nine-step detection engineering workflow used across every
technique in this lab (see [`docs/00-methodology.md`](../../docs/00-methodology.md)):

`Atomic Test → Observe Telemetry → Ingest & Validate → Investigate → SPL Hunt → Sigma Rule → Test & Tune → Document → Commit`

Answers to each objective, and the corresponding evidence, are recorded in
[`telemetry-validation.md`](03-telemetry-validation.md), [`splunk-validation.md`](04-splunk-validation.md),
and [`evidence.md`](05-evidence.md).
