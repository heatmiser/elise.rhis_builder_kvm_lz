# kvm_remove_vm

Destroys a VM and removes all associated storage volumes and kickstart artifacts.
Symmetric counterpart to `kvm_deploy_vm`. Part of RHIS KVM landing zone VM lifecycle.

## Purpose

1. Checks VM state and destroys if running (forced shutdown)
2. Undefines VM from libvirt
3. Removes filesystem-mode disk images (`.img` files)
4. Removes block-mode logical volumes from the block VG
5. Removes kickstart configuration and OEMDRV USB image files
6. Cleans up temporary XML definition file

All operations are idempotent — safe if VM doesn't exist.

## Cross-Role Dependencies

| Variable | Source Role | Description |
|---|---|---|
| `kvm_setup_libvirt_pool_filesystem_path` | `kvm_setup_libvirt` | Filesystem pool mount path |
| `kvm_setup_raid_lvm_vg_block_name` | `kvm_setup_raid` | Block pool VG name |

## Example Usage

```yaml
- hosts: infra_servers
  tasks:
    - ansible.builtin.include_role:
        name: elise.rhis_builder_kvm_lz.kvm_remove_vm
      vars:
        vm: "{{ item }}"
      loop: "{{ vms | selectattr('target_host', 'equalto', inventory_hostname) | list }}"
```
