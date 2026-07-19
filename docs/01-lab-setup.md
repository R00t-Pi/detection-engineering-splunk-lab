# Lab Design & Environment Overview

**Status:** Complete
**Lab:** Splunk Detection Engineering Lab
**Updated:** 2026-07-15

Before building anything, the network and VM layout got fixed so nothing downstream would need to change later. This doc covers *why* the environment is shaped the way it is; the actual build steps for each machine are in their own docs, linked at the bottom.

## Hypervisor and network

**VMware Workstation Pro**, two VMs on a shared host-only network.

Host-only instead of relying purely on NAT (VMnet8) for inter-VM traffic: NAT's DHCP range isn't something this lab needs to depend on, and internet access between the lab machines themselves was never a requirement — only outbound access for updates and downloads. A dedicated host-only segment keeps SOC-to-endpoint traffic isolated and predictable, with static IPs that won't shift under Splunk's configuration.

Each VM keeps its NAT adapter alongside the host-only one, specifically for internet access during setup (package installs, Sysmon/Splunk downloads) — not removed, just not the network the lab actually runs on.

## IP plan

| Device | Hostname | IP |
|---|---|---|
| SOC Server | SOC01 | 192.168.56.10 |
| Windows Endpoint | WIN10-01 | 192.168.56.20 |

Network: `192.168.56.0/24`

## VM specifications — as built

| Setting | Windows 10 x64 | Ubuntu 64-bit |
|---|---|---|
| Operating System | Windows 10 (64-bit) | Ubuntu Linux (64-bit) |
| Memory | 8 GB | 8 GB |
| Processors | 4 vCPUs | 4 vCPUs |
| Hard Disk | 80 GB, NVMe | 90 GB, SCSI |
| Network Adapter 1 | NAT | NAT |
| Network Adapter 2 | Host-only | Host-only |

The disk types differ (NVMe on the Windows VM, SCSI on Ubuntu) because that's what VMware defaulted to for each guest OS during creation — not a deliberate performance choice. Either works fine for a lab at this scale; the difference would only start to matter under sustained heavy indexing, which a single-node lab environment doesn't produce.

This build ran on a host laptop with 32 GB of RAM, so 8 GB per VM (16 GB total) left plenty of headroom for the host OS itself.

## Running this on less RAM

32 GB isn't a requirement — it's just what was available here. If you're building this on a machine with less, here's what actually still works:

**16 GB host RAM** — drop each VM to 4 GB RAM / 2 vCPUs. This is genuinely usable: Splunk Enterprise runs acceptably at this size for the data volumes a single-endpoint home lab generates (Splunk's own free/trial guidance doesn't require anywhere near its production-tier specs for light indexing), and Windows 10 is functional at 4 GB for a headless-ish lab VM with GUI-heavy apps closed when not needed. Expect Splunk searches and Windows GPO operations to feel noticeably slower than at 8 GB, but nothing here will fail outright.

**8 GB host RAM** — this is genuinely tight for two full VMs running simultaneously. If it's the only option, 2 GB RAM / 2 vCPU per VM is the practical floor — Splunk Enterprise will start and index, but expect real slowdowns during search and GPO application, and don't expect to comfortably run both VMs plus anything else on the host at the same time. Below this, consider running the two VMs on separate physical machines instead of squeezing both onto one host, or substituting Splunk's lighter footprint options if a full Enterprise instance becomes unworkable.

**CPU cores** — 2 vCPUs per VM is a reasonable floor regardless of RAM tier. Splunk's indexing and search concurrency benefit from more cores, but a single-endpoint lab isn't generating enough search load for core count to be the bottleneck before RAM is.

**Disk** — 40–50 GB per VM is enough for a lab that isn't retaining months of data. The larger sizes used here (80–90 GB) give room for the 1 GB Sysmon and 1 GB Security log allocations (see `04-windows-setup.md`) plus Splunk's indexed data without needing to manage disk pressure during testing.

## Naming convention

Hostnames describe role and number, not defaults — `SOC01` and `WIN10-01` instead of `UbuntuDesktop` or `DESKTOP-XXXXXXXX`. Keeps things unambiguous in Splunk searches and leaves room to expand cleanly later (`WIN10-02`, `WIN-SRV01`, `KALI01`, if the lab grows).

## Where each build step actually lives

- [`02-ubuntu-soc-setup.md`](02-ubuntu-soc-setup.md) — SOC01 OS-level setup: hostname, static IP, base tooling, time sync
- [`03-splunk-installation.md`](03-splunk-installation.md) — Splunk Enterprise install, indexes, firewall
- [`04-windows-setup.md`](04-windows-setup.md) — WIN10-01 OS-level setup: hostname, static IP, event log sizing, time sync
- [`05-windows-telemetry.md`](05-windows-telemetry.md) — audit policy, PowerShell logging, Sysmon, Universal Forwarder
- [`06-atomic-red-team.md`](06-atomic-red-team.md) — Atomic Red Team install and testing methodology
- [`07-sigma-cli-setup.md`](07-sigma-cli-setup.md) — Sigma CLI install, used to convert detection rules to SPL
- [`08-splunk-ta-windows.md`](08-splunk-ta-windows.md) — Splunk Add-on for Microsoft Windows, normalizes Windows event fields for Sigma/SPL
