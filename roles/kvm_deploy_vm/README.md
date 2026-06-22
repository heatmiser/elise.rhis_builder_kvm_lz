# kvm_deploy_vm

Validates OS disk capacity, creates storage volumes, generates a per-VM kickstart with
OEMDRV USB image, defines the VM in libvirt, starts it, and waits for SSH to be
available. Part of RHIS KVM landing zone VM provisioning (Phase 3).

## Purpose

1. Validates OS disk(s) capacity against filesystem layout (fail-fast)
2. Creates filesystem-mode (qemu-img) and block-mode (LVM) disk volumes
3. Generates VM kickstart (`vm_ks.cfg.j2`) and FAT32 OEMDRV image
4. Templates libvirt XML (`vm_definition.xml.j2`) and defines + starts VM
5. Waits up to 60 minutes for SSH availability
6. Enables VM autostart

## Variables

No role-specific defaults. The VM is fully described by the `vm` dict (from `vm_vars.yml`).

## Cross-Role Dependencies (define in wrapper `group_vars`)

| Variable | Source Role | Description |
|---|---|---|
| `kvm_setup_libvirt_pool_filesystem_path` | `kvm_setup_libvirt` | Filesystem pool mount path |
| `kvm_setup_raid_lvm_vg_block_name` | `kvm_setup_raid` | Block pool VG name |
| `kvm_setup_libvirt_network_name` | `kvm_setup_libvirt` | Libvirt network name |
| `kvm_setup_libvirt_bridge_name` | `kvm_setup_libvirt` | Libvirt bridge name |
| `kvm_distribute_iso_filename` | `kvm_distribute_iso` | RHEL ISO filename |

## Required Input

`vm` dict from `vm_vars.yml` — see project `vars/vm_vars.yml` for full schema.

## Templates

- `vm_ks.cfg.j2` — VM kickstart with compliance LVM layout, `reboot --eject`
- `vm_definition.xml.j2` — Libvirt XML with virtio-scsi, hugepages, CD-ROM boot order

## Dependencies

Run after `kvm_setup_raid`, `kvm_setup_libvirt`, `kvm_distribute_iso`.
