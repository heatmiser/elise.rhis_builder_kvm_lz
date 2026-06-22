# kvm_bare_metal_provision

Generates per-host kickstart configurations and FAT32 OEMDRV USB images for bare-metal
RHEL installation via Dell iDRAC virtual media. Part of RHIS KVM landing zone bare-metal
provisioning (Phase 1).

## Purpose

Runs on `localhost` (Ansible controller). For each target hypervisor host:
1. Renders `ks.cfg.j2` kickstart template to `<workspace>/ks-<hostname>.cfg`
2. Creates a 10MB FAT32 image with OEMDRV label
3. Copies kickstart into the image via `mcopy`

The resulting `.img` files are uploaded to the HTTP server and mounted via iDRAC
virtual media by the wrapper playbook (`bare_metal_deploy_prep.yml`).

## Variables

| Variable | Default | Description |
|---|---|---|
| `kvm_bare_metal_provision_local_workspace` | `"/tmp/kickstart"` | Controller workspace dir |
| `kvm_bare_metal_provision_img_filename` | `"oemdrv.img"` | OEMDRV image filename |

## Required Input (from `host` dict or `host_vars`)

The `host` dict must contain: `hostname`, `domain`, `mac`, `ipv4_address`,
`ipv4_netmask`, `ipv4_gateway`, `name_servers`, `root_enc_pass`, `username`,
`user_enc_pass`, `user_sudoer_policy`, `ssh_pub_key`, `org`, `activation_key`,
`filesystem`, `boot_mb`, `boot_efi_mb`, `lv_root_mb`, `lv_home_mb`, `lv_tmp_mb`,
`lv_var_tmp_mb`, `lv_var_log_mb`, `lv_var_log_audit_mb`, `lv_var_mb`.

## Kickstart Behavior

- Auto-detects Dell BOSS device for OS disk; falls back to `host.boot_disk`/`host.root_disk`
- Compliance-focused LVM layout (DISA-STIG, CIS, PCI-DSS)
- Filesystem type defaults to `xfs`; override via `host.filesystem: "ext4"`
- Lab used `ext4` on all hypervisors and VMs

## Dependencies

Requires `dosfstools` and `mtools` packages on the Ansible controller.
