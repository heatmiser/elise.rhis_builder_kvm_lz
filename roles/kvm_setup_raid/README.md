# kvm_setup_raid

Creates mdadm RAID 10 on Dell DC NVMe disks, partitions the array into dual LVM volume
groups, and defines libvirt filesystem and block storage pools. Part of RHIS KVM landing
zone host configuration (Phase 2).

## Purpose

- Creates `/dev/<kvm_setup_raid_name>` mdadm RAID 10 array
- Partitions array using percentage-based allocation (filesystem + block + overhead)
- Creates LVM PVs, VGs, and LVs for each pool
- Defines, starts, and autostarts libvirt `dir` (filesystem) and `logical` (block) pools
- Starts `mdmonitor` for array health monitoring

## Requirements

- `host.additional_disks` populated (by `kvm_detect_nvme_raid` or `host_vars`)
- `libvirtd` running (start it with `kvm_setup_libvirt` before or after)
- Cross-role vars defined by `kvm_setup_libvirt` (or wrapper `group_vars`): see below

## Variables

| Variable | Default | Description |
|---|---|---|
| `kvm_setup_raid_level` | `10` | mdadm RAID level |
| `kvm_setup_raid_name` | `"md0"` | RAID device name (`/dev/<name>`) |
| `kvm_setup_raid_chunk_size` | `512` | Chunk size in KB |
| `kvm_setup_raid_metadata_version` | `"1.2"` | mdadm metadata version |
| `kvm_setup_raid_force_recreate` | `false` | Destroy and recreate existing array |
| `kvm_setup_raid_lvm_vg_filesystem_name` | `"vg_nvme_fs"` | Filesystem pool VG name |
| `kvm_setup_raid_lvm_vg_block_name` | `"vg_nvme_block"` | Block pool VG name |
| `kvm_setup_raid_lvm_vg_filesystem_percent` | `15` | % of RAID for filesystem pool |
| `kvm_setup_raid_lvm_vg_block_percent` | `80` | % of RAID for block pool |
| `kvm_setup_raid_lvm_overhead_percent` | `5` | % reserved for LVM overhead |
| `kvm_setup_raid_lvm_snapshot_reserve_percent` | `15` | % of each VG reserved for snapshots |

## Cross-Role Dependencies (must be defined by `kvm_setup_libvirt` or wrapper `group_vars`)

| Variable | Example | Description |
|---|---|---|
| `kvm_setup_libvirt_pool_filesystem_path` | `/mnt/md0/libvirt/filesystem` | Filesystem pool mount path |
| `kvm_setup_libvirt_pool_filesystem_name` | `nvme_pool_fs` | Filesystem pool libvirt name |
| `kvm_setup_libvirt_pool_filesystem_type` | `dir` | Filesystem pool libvirt type |
| `kvm_setup_libvirt_pool_block_name` | `nvme_pool_block` | Block pool libvirt name |
| `kvm_setup_libvirt_pool_block_type` | `logical` | Block pool libvirt type |

## Lab-Proven Overrides (flightpath.test, 2 hypervisors)

```yaml
kvm_setup_raid_lvm_vg_filesystem_percent: 45   # lab had filesystem-mode VMs
kvm_setup_raid_lvm_vg_block_percent: 50        # adjusted to balance with filesystem
kvm_setup_raid_lvm_overhead_percent: 5         # unchanged
```

## Dependencies

- Run `kvm_detect_nvme_raid` first to populate `host.additional_disks`
- Run `kvm_setup_libvirt` to ensure `libvirtd` is running (can run concurrently)
