# kvm_detect_nvme_raid

Auto-detects Dell DC NVMe SSDs and verifies RAID 10 capability. Part of the RHIS KVM
landing zone host configuration phase (Phase 2).

## Purpose

Scans `/dev/disk/by-id` for Dell DC NVMe devices and populates `host.additional_disks`
if not already set in `host_vars`. Asserts that at least 4 disks are present before
allowing RAID setup to proceed.

## Requirements

- Target host must be a Dell PowerEdge with Dell DC NVMe SSDs
- `host` dict available in scope (from `host_vars`)

## Variables

| Variable | Default | Description |
|---|---|---|
| `kvm_detect_nvme_raid_min_disk_count` | `4` | Minimum disks for RAID 10 |

## Cross-Role References

Display task references `kvm_setup_raid_level` and `kvm_setup_raid_name` (defined in
`kvm_setup_raid` defaults) for informational output.

## Dependencies

None. Should run before `kvm_setup_raid`.

## Example Playbook

```yaml
- hosts: infra_servers
  roles:
    - role: elise.rhis_builder_kvm_lz.kvm_detect_nvme_raid
```
