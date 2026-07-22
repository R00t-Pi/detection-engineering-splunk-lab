# T1059.001 -- Command and Scripting Interpreter: PowerShell

| | |
|---|---|
| **MITRE ATT&CK** | [T1059.001](https://attack.mitre.org/techniques/T1059/001/) -- Command and Scripting Interpreter: PowerShell |
| **Tactic** | Execution |
| **Atomic Test** | T1059.001-17 -- PowerShell Command Execution |
| **Status** | ✅ Validated -- Tuned |
| **Sigma Rule** | [`sigma/T1059.001_PowerShell_Encoded_Command.yml`](sigma/T1059.001_PowerShell_Encoded_Command.yml) |
| **Sigma CLI Setup** | [`docs/07-sigma-cli-setup.md`](../../docs/07-sigma-cli-setup.md) (one-time environment setup, referenced not repeated) |
| **Log Sources** | Sysmon (EventID 1), Windows Security (EventCode 4688), PowerShell Operational (4103/4104) |
| **Part of Campaign** | [Campaign 01 -- Initial Access to Impact](../../campaigns/campaign-01-initial-access-to-impact/README.md), Stage 1 (Execution) |

## Contents

Files are numbered in the order you should read them — GitHub lists them in that same order.
The **Layer** column shows which part of the technique each artifact belongs to.

| # | File | Layer | Purpose |
|---|---|---|---|
| 1 | [`01-hypothesis.md`](01-hypothesis.md) | 🎯 Threat / Objective | Why this technique, what we expect to see, what "done" looks like |
| 2 | [`02-atomic-test.md`](02-atomic-test.md) | ⚔️ Attack Execution | Atomic Red Team test review, prerequisite checks, execution record |
| 3 | [`03-telemetry-validation.md`](03-telemetry-validation.md) | 🔍 Detection (Windows) | Raw Windows Event Log / Sysmon evidence (Event IDs 4104, 4103, 4688) |
| 4 | [`04-splunk-validation.md`](04-splunk-validation.md) | 🔍 Detection (Splunk) | SPL hunt queries used to confirm ingestion, with evidence |
| 5 | [`05-evidence.md`](05-evidence.md) | 🛡️ Sigma / Rule | Sigma rule design, Sigma → SPL conversion, tuning, final verification |
| -- | [`sigma/`](sigma/) | 🛡️ Sigma / Rule | The Sigma detection rule itself (platform-independent) |
| -- | [`spl/`](spl/) | 🔍 Detection (Splunk) | Generated (raw) and environment-tuned SPL, as standalone files |
| -- | [`screenshots/`](screenshots/) | ⚔️ / 🔍 Evidence | Technique-specific evidence screenshots (01--19), in write-up order. General Sigma CLI installation evidence lives in [`docs/screenshots/sigma-cli-setup/`](../../docs/screenshots/sigma-cli-setup/) instead, since that's environment setup, not evidence for this technique. |

## Summary

PowerShell execution generates the expected telemetry across the PowerShell Operational log,
Windows Security log, and Sysmon, and is ingested correctly into the `powershell`, `wineventlog`,
and `sysmon` Splunk indexes. A high-confidence Sigma rule was built for the specific pattern of
`cmd.exe` spawning an obfuscated (`-EncodedCommand`) PowerShell invocation, converted to SPL using
the official Sigma CLI, tuned for this lab's index naming, and validated end-to-end against the
Atomic Red Team simulation.

See [`05-evidence.md`](05-evidence.md) for the full detection engineering narrative, or jump straight to
the final tuned query: [`spl/tuned_splunk.spl`](spl/tuned_splunk.spl).
