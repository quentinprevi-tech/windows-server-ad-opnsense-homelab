# Backup and Recovery

This document describes the backup and recovery strategy used in the lab.

## Goal

The goal is to protect important lab states before and after major configuration changes.

This makes it possible to roll back quickly if a configuration mistake breaks Active Directory, OPNsense, the DMZ or the Windows client.

## Snapshots vs Backups

### Snapshots

Proxmox snapshots were used as fast rollback points.

Snapshots are useful before changes such as:

- Firewall rule changes
- Active Directory configuration changes
- GPO creation or modification
- Nginx configuration changes
- DMZ firewall hardening

A snapshot is fast to create and fast to revert, but it still depends on the current storage.

### Backups

vzdump backups were used to create real restorable Proxmox backup archives.

Backups were stored on the backup-nvme storage.

Backups are better for recovery if a VM is deleted, corrupted, or needs to be restored later.

## Backed Up VMs

The following VMs were protected:

- VM 300: OPNsense firewall
- VM 310: Windows 11 domain client
- VM 320: SRV-AD01 domain controller
- VM 330: Debian/Nginx DMZ web server

## Important Snapshots

The following snapshots were created after important milestones:

- ad-lab-opnsense-ok
  - VM: 300
  - Purpose: OPNsense interfaces and firewall rules validated

- ad-lab-win11-domain-ok
  - VM: 310
  - Purpose: Windows 11 joined to homelab.local and GPO validated

- ad-lab-srv-ad01-ok
  - VM: 320
  - Purpose: Active Directory, DNS, GPO and file share validated

- dmz-web01-nginx-ok
  - VM: 330
  - Purpose: Debian DMZ web server with Nginx validated

- dmz-web01-nginx-nat-firewall-ok
  - VM: 330
  - Purpose: DMZ Nginx, internal DNS, WAN NAT publication and firewall segmentation validated

## Manual Snapshot Examples

Example commands used for snapshots:

- qm snapshot 300 "ad-lab-opnsense-ok"
- qm snapshot 310 "ad-lab-win11-domain-ok"
- qm snapshot 320 "ad-lab-srv-ad01-ok"
- qm snapshot 330 "dmz-web01-nginx-ok"

## Manual Backup Examples

Example vzdump commands used for real backups:

- vzdump 300 310 320 --storage backup-nvme --mode snapshot --compresz zstd
- vzdump 330 --storage backup-nvme --mode snapshot --compress zstd

The storage backup-nvme was verified as active and had enough available space.

## Backup Verification

Backups were verified by checking the dump directory.

Example:

- ls -lh /mnt/backup-nvme/dump/

Expected result:

- vzdump-qemu-300-*.vma.zst
- vzdump-qemu-310-*.vma.zst
- vzdump-qemu-320-*.vma.zst
- vzdump-qemu-330-*.vma.zst

## Recovery Reasoning

The recovery strategy is:

1. Use a snapshot first if a recent configuration change needs to be reverted.
2. Use a vzdump backup if a VM needs to be restored from a real backup archive.
3. Create a new snapshot after every major validated milestone.
4. Create a vzdump backup after a stable lab state is reached.

## Design Reasoning

This approach shows basic operational discipline.

Instead of making changes directly and hoping that they work, each major configuration step is validated and protected with a rollback point.

This is a common admin practice when managing infrastructure.
