# Splunk Detection Engineering Lab

A hands-on detection engineering lab built with Atomic Red Team, Windows telemetry (Sysmon,
Security, PowerShell logs), Splunk, Sigma, and MITRE ATT&CK. See
[`docs/00-methodology.md`](docs/00-methodology.md) for the full methodology.

## Repository Structure

```
splunk-detection-engineering-lab/
├── README.md
├── CHANGELOG.md
├── LICENSE
├── .gitignore
│
├── docs/
│   ├── 00-methodology.md          # attack lifecycle + detection engineering workflow
│   ├── 01-lab-setup.md
│   ├── 02-ubuntu-soc-setup.md
│   ├── 03-splunk-installation.md
│   ├── 04-windows-setup.md
│   ├── 05-windows-telemetry.md
│   ├── 06-atomic-red-team.md
│   ├── 07-sigma-cli-setup.md      # one-time SOC01 setup: Sigma CLI, Splunk backend, splunk_windows pipeline
│   └── screenshots/
│       └── sigma-cli-setup/       # evidence for 07-sigma-cli-setup.md (install/plugin/pipeline verification)
│
├── architecture/
│   └── architecture-v2.png
│
├── configs/
│   ├── splunk/          # inputs.conf, outputs.conf, indexes.conf
│   └── sysmon/          # sysmonconfig.xml
│
├── campaigns/                                     # the realistic, ordered attack chain
│   └── campaign-01-initial-access-to-impact/
│       ├── README.md         # threat hypothesis + stage-by-stage index
│       └── timeline.md       # correlated cross-technique timeline
│
├── detections/                                    # single source of truth, one folder per technique
│   ├── README.md                                  # index of every technique folder (mirrors coverage/attack-matrix.md)
│   ├── T1059.001-encoded-powershell-via-cmd/
│   │   ├── README.md                    # index for this technique -- MITRE ID, tactic, status, at a glance
│   │   ├── 01-hypothesis.md             # threat hypothesis + detection objectives    [objective]
│   │   ├── 02-atomic-test.md            # Atomic Red Team review + execution steps    [attack execution]
│   │   ├── 03-telemetry-validation.md   # Windows Event Log / Sysmon evidence         [detection]
│   │   ├── 04-splunk-validation.md      # SPL hunt queries + ingestion evidence       [detection]
│   │   ├── 05-evidence.md               # Sigma design, Sigma→SPL conversion, tuning  [sigma]
│   │   ├── sigma/
│   │   │   └── T1059.001_PowerShell_Encoded_Command.yml                              [sigma]
│   │   ├── spl/
│   │   │   ├── generated_splunk.spl                                                  [detection]
│   │   │   └── tuned_splunk.spl                                                      [detection]
│   │   └── screenshots/
│   │       └── 01-atomic-test-execution.png ... 19-tuned-spl-detection-result.png
│   │           # (Sigma CLI install/plugin/pipeline evidence lives in docs/screenshots/sigma-cli-setup/ instead --
│   │           #  it's one-time environment setup, not evidence specific to this technique)
│   ├── T1082-<descriptive-detection-name>/        # (add as executed -- same shape: README + 01..05 + sigma/ + spl/ + screenshots/)
│   ├── T1003-<descriptive-detection-name>/        # (add as executed)
│   └── ...
│
├── coverage/
│   └── attack-matrix.md          # status of every technique, across campaigns + hardening
│
├── hardening/                                     # broad T-code sweep + AD hardening + alerting
│   ├── README.md
│   ├── ad-hardening-notes.md
│   └── alerting-baseline.md
│
└── scripts/                                       # (populate if/when reproducibility automation is written)
```

