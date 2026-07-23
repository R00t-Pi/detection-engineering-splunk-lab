# Threat Hypothesis -- T1003.001 (LSASS Memory)

**MITRE ATT&CK:** T1003.001 -- OS Credential Dumping: LSASS Memory
**Stage:** Credential Access
**Atomic Test:** T1003.001-1 -- Dump LSASS.exe Memory using ProcDump

## Real-World Context

Once an attacker has a foothold and has looked around (see [T1083](../T1083-powershell-directory-enumeration/README.md)),
credentials are usually the next target -- they're what turns "access to one machine" into
"access to the network." The Local Security Authority Subsystem Service (`lsass.exe`) is a
favorite target because it holds authentication material in memory for as long as a user
session is active: plaintext passwords in some configurations, NTLM hashes, and Kerberos
tickets. Dump that process's memory, pull it offline, and run it through a credential-parsing
tool, and you can walk away with everything needed for lateral movement or privilege
escalation -- without ever touching a password file.

This technique shows up constantly in ransomware operations and APT campaigns for exactly that
reason. It's also a good technique to study from a detection standpoint because there isn't
just one way to do it -- Atomic Red Team alone ships close to a dozen different
implementations for T1003.001, from Sysinternals ProcDump to native DLL exports to direct
syscalls designed to dodge EDR hooks.

## Objective

Simulate LSASS memory dumping using Atomic Red Team, and use the resulting telemetry to build
Sigma detection rules that hold up across multiple log sources -- not just "did a suspicious
process run," but "did a process actually reach into LSASS's memory."

This lab focuses on **Test #1 -- Dump LSASS.exe Memory using ProcDump**, since it's one of the
most commonly observed real-world implementations and generates the richest telemetry to build
against. Sysinternals ProcDump is a legitimate, signed Microsoft tool -- which is exactly why
it's attractive to attackers: it's less likely to be flagged just for existing on disk than a
custom credential-dumping binary would be.

## Detection Objectives

For this technique we want to answer four questions:

1. **Did Windows generate the expected telemetry?**
   Expected log sources: Sysmon (Process Create, Process Access, File Create), Windows Security
   Log, PowerShell Operational Log.

2. **Did Splunk ingest the telemetry correctly?**
   Expected indexes: `sysmon`, `wineventlog`, `powershell`.

3. **Can we identify the execution?**
   Evidence should include: Process Image, Parent Process, Command Line, Target Process,
   Granted Access, User, Host, Timestamp.

4. **Can we create a reliable detection?**
   Deliverables: SPL Hunt, Sigma Rule(s), Detection Logic, Validation, Tuning Notes.

## Workflow

This technique follows the standard nine-step detection engineering workflow used across every
technique in this lab (see [`docs/00-methodology.md`](../../docs/00-methodology.md)):

`Atomic Test → Observe Telemetry → Ingest & Validate → Investigate → SPL Hunt → Sigma Rule → Test & Tune → Document → Commit`

One extra wrinkle this time: the first execution attempt was **blocked by Microsoft Defender**
before any of the interesting telemetry could be generated. How that was handled -- and why --
is documented in [`02-atomic-test.md`](02-atomic-test.md).

Answers to each objective, and the corresponding evidence, are recorded in
[`03-telemetry-validation.md`](03-telemetry-validation.md), [`04-splunk-validation.md`](04-splunk-validation.md),
and [`05-evidence.md`](05-evidence.md).
