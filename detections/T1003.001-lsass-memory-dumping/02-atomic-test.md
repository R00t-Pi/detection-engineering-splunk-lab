# Atomic Test Execution -- T1003.001-1

## Step 1 -- Review Available Tests

Before picking a specific implementation, the available Atomic Red Team tests for T1003.001
were listed to see what was on offer:

```powershell
Invoke-AtomicTest T1003.001 -ShowDetailsBrief
```

![List of available Atomic Tests](screenshots/01-list-available-atomic-tests.png)

The output confirmed multiple implementations of LSASS memory dumping -- ProcDump, comsvcs.dll,
direct syscalls with API unhooking, NanoDump, Mimikatz, and a few others. **Test #1 -- Dump
LSASS.exe Memory using ProcDump** was selected for this lab, since it's one of the most commonly
observed real-world implementations.

## Step 2 -- Verify Prerequisites

```powershell
Invoke-AtomicTest T1003.001 -TestNumbers 1 -CheckPrereqs
```

The check confirmed:
- Administrative privileges were available
- Sysinternals ProcDump was already present in the Atomic Red Team `ExternalPayloads` directory

Since the dependency was already installed from earlier lab setup, `-GetPrereqs` was run anyway
just to confirm nothing extra was needed -- consistent with the repeatable, auditable workflow
used across every technique in this lab, whether or not a given test strictly requires it:

```powershell
Invoke-AtomicTest T1003.001 -TestNumbers 1 -GetPrereqs
```

![Prerequisite check and GetPrereqs output](screenshots/02-prereq-check-and-getprereqs.png)

Both confirmed ProcDump was already present and ready to go.

## Step 3 -- Initial Execution Attempt (Blocked)

```powershell
Invoke-AtomicTest T1003.001 -TestNumbers 1
```

This first attempt **failed with an "Access is denied" error** before ProcDump could dump the
LSASS process. Digging into why, Microsoft Defender's Real-time Protection had detected and
removed the payload:

![Microsoft Defender blocking the ProcDump execution](screenshots/04-defender-blocked-hacktool-dumplsass.png)

Defender flagged it specifically as **`HackTool:Win32/DumpLsass.E`**, matching on the exact
command line ProcDump was invoked with. This is expected, and honestly a good sign -- modern
Windows systems are supposed to catch this. Credential-dumping tools like ProcDump are commonly
identified and blocked by endpoint protection products precisely because this command-line
pattern is so well known.

## Step 4 -- Lab Configuration Adjustment

The point of this lab isn't to prove Defender works (it clearly does) -- it's to collect the
complete telemetry an analyst would need to build a detection, which means the simulation has
to actually run end-to-end at least once. To allow that, Microsoft Defender Real-time
Protection was **temporarily disabled**, strictly within this isolated lab VM:

![Microsoft Defender Real-time Protection disabled](screenshots/03-defender-realtime-protection-disabled.png)

**This is a lab-only decision, not a production recommendation.** Endpoint protection should
stay enabled in any real environment -- disabling it here was purely to generate telemetry for
detection engineering purposes, and protection was re-enabled immediately after evidence
collection and detection validation were complete.

## Step 5 -- Execute

With Defender's real-time protection temporarily out of the way, the test was run again:

```powershell
Invoke-AtomicTest T1003.001 -TestNumbers 1
```

This time it completed successfully. ProcDump attached to the LSASS process and wrote a full
memory dump to `C:\Windows\Temp\lsass_dump.dmp`:

![Atomic test execution succeeding, 55 MB dump written](screenshots/05-atomic-test-execution-success.png)

**Execution record:**

| Field | Value |
|---|---|
| Command | `Invoke-AtomicTest T1003.001 -TestNumbers 1` |
| Start time | 17:29:17 on 20 July 2026 |
| Dump written | 17:29:32, `C:\Windows\Temp\lsass_dump.dmp`, 55 MB in 0.3 seconds |
| Reported exit code | `-2` |
| Host | WIN10-01 |

Worth a quick note on that exit code: `-2` looks like a failure at a glance, but ProcDump's own
console output confirms the dump completed and the file was written -- this is just ProcDump's
own exit-code convention (it uses non-zero/negative codes for a range of conditions, "dump
count reached" being one of them), not a sign anything went wrong.

Windows generated telemetry across Sysmon, the Security log, and the PowerShell Operational log
within seconds of this run. See [`03-telemetry-validation.md`](03-telemetry-validation.md) and
[`04-splunk-validation.md`](04-splunk-validation.md).
