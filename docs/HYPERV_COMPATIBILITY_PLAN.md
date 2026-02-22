# Hyper-V Compatibility Analysis and Update Plan (2012/2012 R2/2016/2019)

## Repository purpose (current state)
Corefig is a PowerShell/WinForms configuration utility originally built for Windows Server 2012 Core and Hyper-V Server 2012. It provides a local GUI for common headless-host tasks including networking, firewall, WinRM, and a basic Hyper-V VM management pane (list/start/stop/screenshot). The current docs explicitly position it as a 2012-era tool.

## Why VM discovery fails on Windows Server/Hyper-V 2016/2019
The Hyper-V module (`hyperV.ps1`) uses legacy WMI namespace/class queries against `root\virtualization` via `Get-WmiObject`:

- `Msvm_ComputerSystem` for VM inventory and lifecycle actions.
- `Msvm_InternalEthernetPort` for network listing.
- `Msvm_VirtualSystemManagementService` for screenshots.

On newer hosts (especially 2016+), Hyper-V WMI providers are primarily exposed in `root\virtualization\v2`. Querying only the older namespace commonly returns no VMs (or partial data), which matches the symptom “Hyper-V Manager shows no VMs to manage” when tooling depends on the wrong provider path.

## Design goals for first modernization phase
1. Keep current behavior for 2012.
2. Add reliable support for 2012 R2, 2016, and 2019.
3. Prefer built-in Hyper-V cmdlets when available; fall back to WMI where needed.
4. Isolate version/provider differences in one compatibility layer.
5. Preserve existing UI and localization as much as possible.

## Compatibility strategy
### 1) Introduce a provider abstraction
Add `Get-HyperVProviderContext` that detects best available access path:

- **Primary:** Hyper-V PowerShell cmdlets (`Get-VM`, `Start-VM`, `Stop-VM`) if module present.
- **Secondary:** WMI namespace fallback order:
  1. `root\virtualization\v2`
  2. `root\virtualization`

Return a context object used by all VM/network/screenshot functions.

### 2) Replace direct script calls with wrapper functions
Refactor `hyperV.ps1` operations to wrappers:

- `Get-CorefigVMInventory`
- `Start-CorefigVM`
- `Stop-CorefigVM`
- `Get-CorefigVMNetworks`
- `Get-CorefigVMThumbnail`

Each wrapper uses provider context; UI code remains mostly unchanged.

### 3) Harden state mapping and filtering
Normalize VM states for all providers:

- Running
- Saved
- Stopped
- Other/Unknown (optional UI bucket)

Ensure host system object filtering remains explicit so host is never listed as a VM.

### 4) Remote-management posture checks (for Hyper-V Manager interoperability)
Add a preflight diagnostic action (read-only) that reports:

- WinRM listener status.
- Required firewall groups/rules relevant to Hyper-V remote management.
- Current PowerShell remoting status.
- Optional DCOM/WMI reachability hints.

Note: this does not auto-change policy by default; offer “fix” actions as explicit user opt-in.

## Suggested implementation phases
### Phase 0 – Baseline and safety net
- Add transcript-style diagnostics logging for Hyper-V provider selection.
- Add unit-like script tests for provider selection and VM state mapping.

### Phase 1 – 2012/2012 R2/2016/2019 compatibility core
- Implement provider context and wrapper functions.
- Migrate existing VM list/start/stop flows to wrappers.
- Keep old WMI code path as fallback.

### Phase 2 – Hyper-V Manager interoperability diagnostics
- Add “Hyper-V Remote Health” screen or button.
- Surface actionable remediation commands (copy/run).

### Phase 3 – Cleanup and extension path
- Reduce duplicate localization strings if new UI labels added.
- Prepare extension points for 2019/2022 validation.

## Acceptance criteria (phase 1)
1. On 2012: VM list/start/stop still works.
2. On 2012 R2: VM list/start/stop works without manual code edits.
3. On 2016: VM list/start/stop works and no empty-list false negative caused by namespace mismatch.
4. On 2019: VM list/start/stop works using cmdlets when available, or WMI fallback as needed.
5. Provider chosen is visible in logs.

## Validation matrix to run before release
- Windows Server 2012 Core + Hyper-V role
- Hyper-V Server 2012
- Windows Server 2012 R2 Core + Hyper-V role
- Hyper-V Server 2016 / Windows Server 2016 Core + Hyper-V role
- Hyper-V Server 2019 / Windows Server 2019 Core + Hyper-V role

For each environment validate:
- Open Hyper-V page -> VM list populates.
- Start stopped VM.
- Stop running VM.
- Refresh updates UI.
- (Optional) Screenshot call behavior.

## Immediate troubleshooting commands for 2016 hosts
Use these commands on the host to confirm current management state while updates are in progress:

```powershell
Get-VM
Get-CimInstance -Namespace root/virtualization/v2 -ClassName Msvm_ComputerSystem |
  Where-Object { $_.Caption -ne 'Hosting Computer System' } |
  Select-Object ElementName, EnabledState
winrm enumerate winrm/config/listener
Get-NetFirewallRule -DisplayGroup 'Windows Remote Management'
```

If `Get-VM` works but Corefig Hyper-V page is empty, the issue is almost certainly provider/namespace compatibility in current script logic.
