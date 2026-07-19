# detections

One folder per technique. Each folder is self-contained: threat hypothesis, the Atomic Red
Team execution that generated the telemetry, the Windows/Splunk evidence that telemetry was
captured correctly, and the Sigma rule (plus its SPL conversion) that detects it.

**Folder naming:** `<MITRE-Technique-ID>-<what-the-detection-actually-catches>`, e.g.
`T1059.001-encoded-powershell-via-cmd`. The T-code makes it greppable against ATT&CK; the
suffix describes the specific behavior detected, not just the tactic — a T-code can cover
several distinct detections, and the suffix is what tells them apart at a glance.

Every folder has the same shape, numbered in reading order (README, then 01 through 05,
plus `sigma/`, `spl/`, `screenshots/`) — see any existing technique folder for the pattern.

For current status across all techniques — including ones not started yet — see
`../coverage/attack-matrix.md`.