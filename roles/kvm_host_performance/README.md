# kvm_host_performance

Applies IBM KVM hypervisor performance best practices. Part of RHIS KVM landing zone
host configuration (Phase 2). Merges tuning and Intel IOMMU tasks from the original
`rhis-builder-kvm-lz` project.

## Purpose

- Sets `tuned` profile to `virtual-host`
- Configures `/etc/modprobe.d/kvm.conf` with `hpage` and `halt_poll_ns` parameters
- Reserves huge pages (`vm.nr_hugepages` via sysctl)
- Configures `kernel.sched_migration_cost_ns` (gracefully skipped if unavailable)
- Disables cpuset cgroup controller in `/etc/libvirt/qemu.conf` (cgroups v1 only)
- Optionally configures Intel IOMMU if hardware supports it and `enable_iommu: true`

## Variables

| Variable | Default | Description |
|---|---|---|
| `kvm_host_performance_tuned_profile` | `"virtual-host"` | tuned profile name |
| `kvm_host_performance_hugepages_percent` | `70` | % of RAM to reserve for huge pages |
| `kvm_host_performance_hugepages_count` | *(unset)* | Override auto-calc; number of 2MB pages |
| `kvm_host_performance_kvm_halt_poll_ns` | `50000` | KVM idle vCPU polling (ns) |
| `kvm_host_performance_kvm_hpage` | `1` | KVM hpage module param (0=off, 1=on) |
| `kvm_host_performance_sched_migration_cost_ns` | `500000` | CPU scheduler migration cost (ns) |
| `kvm_host_performance_enable_iommu` | `false` | Enable Intel IOMMU if hardware supports it |

## Handlers

- `KVM module reload required` — triggered by `kvm.conf` changes; reloads kvm module
- `Libvirtd restart required` — triggered by `qemu.conf` changes; restarts libvirtd

**Note:** KVM module reload requires no VMs to be running. During initial host setup this
is safe. On live hosts, quiesce VMs first.

## Intel IOMMU

`intel_iommu.yml` is included when `kvm_host_performance_enable_iommu: true` AND
`/sys/firmware/acpi/tables/DMAR` exists. If `intel_iommu=on` is not in the kernel
cmdline, it sets it via `grubby` and reboots (900s timeout for Dell POST).

**Warning:** Enabling IOMMU passthrough disables KVM memory over-commitment.

## Lab Notes

Lab (flightpath.test) ran with IOMMU disabled. All defaults matched lab usage.

## Dependencies

Requires `libvirtd` running (started by `kvm_setup_libvirt`).
