# kvm_setup_libvirt

Note that this initial implementation is based on a specific lab implmentaiton and will
be updated to provide a more generic implementation that can be applied to a wider
variety of host hardware configurations.

Installs libvirt, configures a NetworkManager bridge on the second NIC, defines the
libvirt bridge network, configures firewall, and enables IP forwarding. Part of RHIS KVM
landing zone host configuration (Phase 2).

## Purpose

- Installs `@virtualization-host-environment` package group
- Starts and enables `libvirtd`
- Auto-selects the second active NIC for bridge (excludes primary/kickstart NIC by MAC)
- Creates `br1` (default) bridge with IPv4/IPv6 disabled (pure L2 bridge for VM traffic)
- Defines and autostarts libvirt bridge network
- Opens libvirt firewall services
- Enables `net.ipv4.ip_forward` via sysctl

## Key Design Decisions

**Bridge NIC selection**: The primary NIC (identified by `host.mac`) is excluded.
The next active NIC becomes the bridge uplink. Existing NM connections on that interface
are deleted before bridging to avoid DHCP conflicts.

**Storage pool creation**: Pools are created by `kvm_setup_raid`. This role defines the
pool configuration variables (`kvm_setup_libvirt_pool_*`) used by `kvm_setup_raid` and
all downstream roles. Set these in wrapper `group_vars` to override.

## Variables

| Variable | Default | Description |
|---|---|---|
| `kvm_setup_libvirt_bridge_name` | `"br1"` | Bridge interface and NM connection name |
| `kvm_setup_libvirt_network_name` | `"libvirt_net1"` | Libvirt network name |
| `kvm_setup_libvirt_network_mode` | `"bridge"` | Libvirt network forward mode |
| `kvm_setup_libvirt_network_mtu` | `1500` | Bridge MTU |
| `kvm_setup_libvirt_pool_filesystem_path` | `/mnt/{{ kvm_setup_raid_name }}/libvirt/filesystem` | Filesystem pool mount path |
| `kvm_setup_libvirt_pool_filesystem_name` | `"nvme_pool_fs"` | Filesystem pool libvirt name |
| `kvm_setup_libvirt_pool_filesystem_type` | `"dir"` | Filesystem pool libvirt type |
| `kvm_setup_libvirt_pool_block_name` | `"nvme_pool_block"` | Block pool libvirt name |
| `kvm_setup_libvirt_pool_block_type` | `"logical"` | Block pool libvirt type |

## Internal Facts Set

- `kvm_setup_libvirt_physical_interface` — the selected bridge uplink NIC name

## Dependencies

Should run before or concurrently with `kvm_setup_raid` (both need `libvirtd` running).
`kvm_setup_raid` references `kvm_setup_libvirt_pool_*` variables.

## Example Playbook

```yaml
- hosts: infra_servers
  roles:
    - role: elise.rhis_builder_kvm_lz.kvm_detect_nvme_raid
    - role: elise.rhis_builder_kvm_lz.kvm_setup_libvirt
    - role: elise.rhis_builder_kvm_lz.kvm_setup_raid
```
