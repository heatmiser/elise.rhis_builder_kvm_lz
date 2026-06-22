# elise.rhis_builder_kvm_lz

Ansible collection providing KVM landing zone automation for the [Red Hat Infrastructure Standard (RHIS)](https://github.com/parmstro/rhis-architecture).

## Description

Automates bare-metal RHEL 9 deployment on Dell PowerEdge servers and KVM VM provisioning via kickstart. Extracted from [`rhis-builder-kvm-lz`](https://github.com/parmstro/rhis-builder-kvm-lz) as a reusable collection.

## Roles

| Role | Purpose |
|---|---|
| `kvm_detect_nvme_raid` | Auto-detect Dell DC NVMe devices and verify RAID 10 capability |
| `kvm_setup_raid` | Create mdadm RAID 10, partition for dual LVM pools, define libvirt pools |
| `kvm_setup_libvirt` | Configure libvirt bridge network and firewall |
| `kvm_host_performance` | Apply IBM KVM performance best practices (tuned, hugepages, KVM params, IOMMU) |
| `kvm_distribute_iso` | Distribute RHEL installation ISO to hypervisor filesystem pool |
| `kvm_bare_metal_provision` | Generate kickstart and OEMDRV USB image for bare-metal install |
| `kvm_deploy_vm` | Validate OS disk, create volumes, generate VM kickstart, define and boot VM |
| `kvm_remove_vm` | Destroy VM, remove storage volumes and kickstart artifacts |
| `kvm_wipe_disk` | Detect and wipe Dell BOSS/NVMe disks via blkdiscard, zero, shred, or nvme-format |

## Requirements

- Ansible >= 2.16
- RHEL 9 target hosts
- Dell PowerEdge with iDRAC 9+ (for bare-metal phases)
- Python 3.9+ on controller

## Dependencies

```yaml
collections:
  - dellemc.openmanage >= 9.0.0
  - community.libvirt >= 1.0.0
  - community.general >= 9.0.0
  - ansible.posix >= 1.5.0
  - ansible.utils >= 2.10.0
```

Install: `ansible-galaxy collection install -r collections/requirements.yml`

## License

GPL-3.0-only
