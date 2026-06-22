# kvm_wipe_disk

Detects and wipes Dell BOSS and/or Dell DC NVMe disks using the configured method.
Designed to run on a Live ISO boot (CentOS Stream minimal live) via iDRAC virtual media.
Part of RHIS KVM landing zone bare-metal wipe workflow.

## Purpose

1. Detects Dell BOSS devices (OS disk) and Dell DC NVMe devices (data disks)
2. Validates at least one disk was detected
3. Wipes each disk using the selected method with per-disk logging
4. Executes post-wipe action (poweroff/reboot/none)

## Variables

| Variable | Default | Description |
|---|---|---|
| `kvm_wipe_disk_method` | `"blkdiscard"` | Wipe method: `blkdiscard`, `zero`, `shred`, `nvme-format` |
| `kvm_wipe_disk_boss_devices` | `true` | Target Dell BOSS devices |
| `kvm_wipe_disk_nvme_devices` | `true` | Target Dell DC NVMe devices |
| `kvm_wipe_disk_post_action` | `"poweroff"` | Post-wipe: `poweroff`, `reboot`, `none` |
| `kvm_wipe_disk_log_path` | `"/tmp/wipe"` | Directory for per-disk wipe logs |

## Wipe Methods

| Method | Speed | Security | Notes |
|---|---|---|---|
| `blkdiscard` | Near-instant | Not DoD | SSD TRIM/UNMAP (default for NVMe) |
| `zero` | Fast | Not DoD | Full zero-write via dd |
| `shred` | Slow | DoD 3-pass | 3-pass overwrite |
| `nvme-format` | Fast | Hardware | NVMe controller secure erase |

## Dependencies

Requires `util-linux` (blkdiscard), `coreutils` (dd, shred), or `nvme-cli` (nvme format)
depending on selected method. The Live ISO environment provides these.
