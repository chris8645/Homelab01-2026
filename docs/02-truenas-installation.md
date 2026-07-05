# 02 - TrueNAS SCALE Installation

## Overview

This phase focused on deploying TrueNAS SCALE as the dedicated storage server for the homelab. TrueNAS will manage the ZFS storage pools and provide centralized storage for Docker applications, media, photos, backups, and future services.

---

## VM Configuration

| Setting | Value |
|---------|-------|
| vCPUs | 4 |
| Memory | 16 GB |
| Boot Disk | 64 GB VirtIO |
| Network | VirtIO (vmbr0) |

---

## Installation

- Created a new TrueNAS SCALE virtual machine in Proxmox.
- Installed TrueNAS SCALE on a dedicated 64 GB virtual disk.
- Configured the administrative account.
- Verified access to the TrueNAS web interface.

---

## Storage Configuration

Initially, the TrueNAS VM only detected the 64 GB virtual installation disk.

To make the physical storage drives available, the SATA drives were passed through from Proxmox to the TrueNAS virtual machine.

---

## Issue Encountered

### Duplicate Disk Serial Numbers

After passing the drives through, TrueNAS detected all four drives, but they reported duplicate or missing serial numbers. As a result, TrueNAS would not allow a storage pool to be created because it could not uniquely identify each disk.

---

## Resolution

After researching the issue, I manually edited the Proxmox VM configuration file to assign the correct serial numbers to each passthrough disk.

Once unique serial numbers were configured:

- All four drives were correctly identified.
- TrueNAS displayed the proper serial numbers.
- The storage pools could be created normally.

---

## Lessons Learned

This issue highlighted the importance of exposing unique hardware identifiers when passing physical disks to virtual machines. Storage platforms such as TrueNAS rely on disk serial numbers to safely identify and manage devices.
