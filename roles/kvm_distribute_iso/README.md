# kvm_distribute_iso

Distributes the RHEL installation ISO to the hypervisor's filesystem storage pool.
Part of RHIS KVM landing zone host configuration (Phase 2).

## Purpose

Creates `<pool_path>/iso/` directory, downloads or copies the RHEL ISO, and sets the
correct `virt_image_t` SELinux context for libvirt access.

## Variables

| Variable | Default | Description |
|---|---|---|
| `kvm_distribute_iso_filename` | `"rhel-9.7-x86_64-dvd.iso"` | RHEL ISO filename |
| `kvm_distribute_iso_liveiso_filename` | `"CentOS-Stream-MIN-Live-Automation.x86_64-10.iso"` | Live ISO filename (wipe workflow) |
| `kvm_distribute_iso_http_server_base_url` | *(required)* | HTTP base URL for ISO downloads |
| `kvm_distribute_iso_use_tmpfs` | `false` | Copy from pre-staged tmpfs instead of HTTP |
| `kvm_distribute_iso_tmpfs_path` | `"/dev/shm"` | Tmpfs staging path when `use_tmpfs: true` |

## Cross-Role Dependencies

- `kvm_setup_libvirt_pool_filesystem_path` — provided by `kvm_setup_libvirt` defaults or wrapper `group_vars`

## Lab Override

```yaml
kvm_distribute_iso_use_tmpfs: true   # ISO pre-staged in /dev/shm for fast re-runs
```

## Dependencies

Run after `kvm_setup_raid` (filesystem pool must be mounted).
