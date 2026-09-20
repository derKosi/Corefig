# Platform Support Matrix

Corefig targets the local-GUI gap on headless Windows hosts. The relevant
platform landscape as of 2026:

## Hyper-V Server (free standalone product)

| Version | Status | Notes |
|---------|--------|-------|
| 2008 R2 / 2012 / 2012 R2 / 2016 | Out of support | Original design targets of Corefig/CoreConfig |
| **2019** | **Last standalone version. Extended support ends 2029-01-09** | Microsoft confirmed no free standalone successor |

There is **no free standalone Hyper-V Server after 2019**. Microsoft's
successor in the hyperconverged segment is Azure Local (formerly Azure
Stack HCI), a subscription product — out of scope for Corefig.

## Hyper-V as a Windows Server role

| Version | Support ends | Hyper-V highlights |
|---------|--------------|--------------------|
| Server 2012 / 2012 R2 | ended 2023-10-10 | original Corefig era |
| Server 2016 | 2027-01-12 | |
| Server 2019 | 2029-01-09 | |
| Server 2022 | 2031-10-14 | nested virt on AMD, updated RSC |
| Server 2025 | ~2034 | Gen2 default, GPU partitioning, hypervisor-enforced paging |

Server Core installations of these versions remain a valid Corefig target
(the GUI-less SKU is exactly what Corefig was built for).

## How Corefig handles the version spread

The Hyper-V module (`hyperV.ps1`) uses a provider cascade added in 2026:

1. **Hyper-V PowerShell cmdlets** (`Get-VM`, `Start-VM`, `Stop-VM`, ...) —
   used whenever the module is present (2016+, typically also installed on
   standalone Hyper-V Server 2019).
2. **WMI `root\virtualization\v2`** — fallback for older/trimmed hosts.
3. **WMI `root\virtualization`** — 2012-era fallback.

Known limitation: the WMI fallbacks use `Get-WmiObject`, which exists in
Windows PowerShell 5.1 (the interpreter Corefig launches) but not in
PowerShell 7+. Migrating the fallbacks to `Get-CimInstance` is planned, but
deliberately deferred until it can be verified on real 2012/2012 R2 hosts —
 Corefig's contract is to keep legacy hosts working, not to modernize at
their expense.

## What Corefig does NOT target

- Azure Local / Azure Stack HCI (subscription-managed, has its own tooling)
- Windows client Hyper-V (Windows 10/11 Pro) — different management surface
- Linux/KVM hosts (out of scope by design)
