# Telemetry Validation -- T1003.001

After the Atomic Red Team simulation completed successfully (see [`02-atomic-test.md`](02-atomic-test.md)),
the resulting telemetry was collected from three sources: **Sysmon Operational**, the
**Windows Security log**, and the **PowerShell Operational log**. Execution took place at
**17:29:32 on 20 July 2026**.

---

## 1. Sysmon Operational Log

**Navigation:**
`Event Viewer → Applications and Services Logs → Microsoft → Windows → Sysmon → Operational`

Sysmon recorded five events that, read together, tell the complete story of the credential
dumping attack -- from the first process launched down to the actual memory access against
LSASS. The log was filtered to the execution window (17:29:17--17:40:42) to isolate the
relevant activity.

### Event ID 1 -- Process Create (`cmd.exe`)

![Event 1 general view -- cmd.exe](screenshots/06-eventid-1-cmd-general.png)

| Field | Value |
|---|---|
| Image | `C:\Windows\System32\cmd.exe` |
| ParentImage | `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe` |
| ParentCommandLine | `"...\powershell.exe" -ExecutionPolicy Bypass` |
| CommandLine | `"cmd.exe" /c "...\ExternalPayloads\procdump.exe" -accepteula -ma lsass.exe C:\Windows\Temp\lsass_dump.dmp` |
| User | WIN10-01\Windowss10Pro |
| IntegrityLevel | High |

This is the first link in the chain: PowerShell (orchestrating the Atomic Red Team test)
launches `cmd.exe`, and the command line handed to it already contains the full ProcDump
invocation. The `IntegrityLevel: High` confirms this ran with the elevated privileges needed to
touch LSASS later in the chain.

### Event ID 1 -- Process Create (`procdump.exe`)

![Event 1 general view -- procdump.exe](screenshots/07-eventid-1-procdump-general.png)

| Field | Value |
|---|---|
| Image | `C:\AtomicRedTeam\ExternalPayloads\procdump.exe` |
| ParentImage | `C:\Windows\System32\cmd.exe` |
| CommandLine | `procdump.exe -accepteula -ma lsass.exe C:\Windows\Temp\lsass_dump.dmp` |
| Company | Sysinternals - www.sysinternals.com |
| IntegrityLevel | High |

`cmd.exe` spawns ProcDump directly. The `-accepteula` flag auto-accepts the Sysinternals
license (so no GUI prompt interrupts the run), and `-ma` requests a full memory dump rather
than a lighter mini-dump.

### Event ID 10 -- Process Access (`procdump.exe → lsass.exe`)

![Event 10 general view -- procdump.exe to lsass.exe](screenshots/08-eventid-10-procdump-lsass-general.png)

| Field | Value |
|---|---|
| SourceImage | `C:\AtomicRedTeam\ExternalPayloads\procdump.exe` |
| TargetImage | `C:\Windows\system32\lsass.exe` |
| GrantedAccess | `0x1FFFFF` |
| SourceUser | WIN10-01\Windowss10Pro |
| TargetUser | NT AUTHORITY\SYSTEM |

This is the event that actually matters most for detection: it's not just "ProcDump ran," it's
**ProcDump reaching into LSASS's memory**. `GrantedAccess: 0x1FFFFF` is effectively
`PROCESS_ALL_ACCESS` -- full read/write/query access to a SYSTEM-owned process, requested by a
process running as a regular (if elevated) user. That gap -- a non-SYSTEM process getting
full access to a SYSTEM process -- is exactly the kind of thing that doesn't happen during
normal operation.

### Event ID 1 -- Process Create (`procdump64.exe`)

![Event 1 general view -- procdump64.exe](screenshots/09-eventid-1-procdump64-general.png)

| Field | Value |
|---|---|
| Image | `C:\AtomicRedTeam\ExternalPayloads\procdump64.exe` |
| ParentImage | `C:\AtomicRedTeam\ExternalPayloads\procdump.exe` |
| CommandLine | `procdump.exe -accepteula -ma lsass.exe C:\Windows\Temp\lsass_dump.dmp` |
| IntegrityLevel | High |

Worth calling out on its own, since it's easy to miss the first time: the 32-bit `procdump.exe`
launcher re-executes itself as the 64-bit `procdump64.exe` to actually do the dumping (LSASS on
a 64-bit system needs a 64-bit dumping tool). That means the attack chain has **two separate
LSASS access attempts** to account for, not one -- the first from `procdump.exe`, and a second
from `procdump64.exe` after it re-launches. Both need to be covered by a detection built on
process access, which is exactly why Sigma Rule 2 in [`05-evidence.md`](05-evidence.md) doesn't
hard-code a single binary name.

### Event ID 10 -- Process Access (`procdump64.exe → lsass.exe`, x2)

Sysmon recorded **two** separate process-access events from `procdump64.exe` against
`lsass.exe`, a few milliseconds apart -- consistent with ProcDump making more than one API call
against the target process while building the dump (Sysmon's Process Access rule doesn't
de-duplicate rapid repeated accesses from the same source/target pair).

![Event 10 general view -- procdump64.exe to lsass.exe (1)](screenshots/10-eventid-10-procdump64-lsass-general-1.png)
![Event 10 general view -- procdump64.exe to lsass.exe (2)](screenshots/11-eventid-10-procdump64-lsass-general-2.png)

| Field | Value (both events) |
|---|---|
| SourceImage | `C:\AtomicRedTeam\ExternalPayloads\procdump64.exe` |
| TargetImage | `C:\Windows\system32\lsass.exe` |
| GrantedAccess | `0x1FFFFF` |
| SourceUser | WIN10-01\Windowss10Pro |
| TargetUser | NT AUTHORITY\SYSTEM |

Same full-access pattern as the earlier `procdump.exe` access event, just from the re-launched
64-bit binary. Between the two ProcDump processes, that's three separate `GrantedAccess:
0x1FFFFF` events against LSASS in this single test run -- a strong, repeatable signal.

### A quick note on the `RuleName` field

Every **Event ID 1** (Process Create) event above carries a Sysmon `RuleName` of
`technique_id=T1083,technique_name=File and Directory Discovery` -- clearly stale, left over
from this lab's earlier T1083 work, and not actually describing what's happening here. The
**Event ID 10** (Process Access) events, by contrast, correctly show
`technique_id=T1003,technique_name=Credential Dumping`. This lines up with a fairly common
Sysmon config pattern: a broad, generic "log all process creation" rule for Event ID 1 (which
picked up an old label and never got updated), alongside specific, technique-tagged rules for
Event ID 10 (which did get configured correctly for this scenario). It's a good reminder not to
trust a `RuleName` tag at face value just because it's present -- verify what the event is
actually describing, especially for Process Create events in this lab's config.

---

## 2. Windows Security Log (Event ID 4688)

**Navigation:** `Event Viewer → Windows Logs → Security`

![Event 4688 general view](screenshots/15-eventid-4688-processcreation-general.png)

| Field | Value |
|---|---|
| SubjectUserName | Windowss10Pro |
| NewProcessName | `C:\AtomicRedTeam\ExternalPayloads\procdump.exe` |
| ParentProcessName | `C:\Windows\System32\cmd.exe` |
| CommandLine | `...\ExternalPayloads\procdump.exe -accepteula -ma lsass.exe C:\Windows\Temp\lsass_dump.dmp` |
| TokenElevationType | %%1937 |
| MandatoryLabel | S-1-16-12288 (High Mandatory Level) |

This corroborates the Sysmon Event ID 1 (`procdump.exe`) evidence above, using Windows' native
auditing instead of Sysmon -- useful in environments where Sysmon isn't deployed. The
`TokenElevationType` and `MandatoryLabel` both confirm the process ran with the elevated
privileges required to access LSASS.

---

## 3. PowerShell Operational Log

**Navigation:**
`Event Viewer → Applications and Services Logs → Microsoft → Windows → PowerShell → Operational`

### Event ID 4104 -- Script Block Logging (Attack Execution)

![Event 4104 general view](screenshots/12-eventid-4104-scriptblock-general.png)

| Field | Value |
|---|---|
| ScriptBlockText | `Invoke-AtomicTest T1003.001 -TestNumbers 1` |
| Host Application | `powershell.exe -ExecutionPolicy Bypass` |

Confirms the exact command used to kick off the simulation.

### Event ID 4103 -- Atomic Test Metadata

![Event 4103 general view -- metadata](screenshots/13-eventid-4103-atomictest-metadata-general.png)

This event's `Payload` field is large -- it's PowerShell's Module Logging capturing the entire
parsed YAML definition for every T1003.001 Atomic Test (all ~15 of them, including the ProcDump
one actually used here), since `Invoke-AtomicTest` loads the whole technique file before
picking the specific test number. Buried in there is the confirmation that matters:

| Field | Value |
|---|---|
| Attack Technique | T1003.001 |
| Atomic Test | Dump LSASS.exe Memory using ProcDump |
| Expected Output | `C:\Windows\Temp\lsass_dump.dmp` |

### Event ID 4103 -- Successful Test Completion

![Event 4103 general view -- completion](screenshots/14-eventid-4103-test-completion-general.png)

| Field | Value |
|---|---|
| Payload | `Done executing test: T1003.001-1 Dump LSASS.exe Memory using ProcDump` |
| Host Application | `powershell.exe -ExecutionPolicy Bypass` |

Confirms `Invoke-AtomicTest` itself reported a clean finish, independent of whatever ProcDump's
own exit code said.

---

## Conclusion

All expected log sources -- Sysmon (Process Create, Process Access), Windows Security (4688),
and PowerShell Operational (4104, 4103) -- captured the full attack chain consistently:
PowerShell → `cmd.exe` → `procdump.exe` → LSASS access → `procdump64.exe` → LSASS access (x2).
This confirms Detection Objective #1 from [`01-hypothesis.md`](01-hypothesis.md).

Next: [`04-splunk-validation.md`](04-splunk-validation.md) confirms ingestion into Splunk and
traces the same chain end-to-end with SPL.
