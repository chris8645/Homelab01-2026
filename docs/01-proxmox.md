# 01 - Proxmox VE Installation & Initial Configuration

## Overview

The first phase of the homelab focused on deploying Proxmox VE as the virtualization platform that will host all virtual machines and services.

The goal was to prepare a stable virtualization environment before deploying TrueNAS SCALE, Docker, AI services, and game servers.

---

## Objectives

- Install Proxmox VE
- Configure networking
- Configure package repositories
- Update the system
- Prepare the host for virtual machines

---

## Hardware

| Component | Model |
|----------|-------|
| CPU | AMD Ryzen 5 5600X |
| RAM | 64GB DDR4 |
| Boot Drive | 2TB NVMe SSD |
| Storage | 2 × 8TB SATA + 2 × 4TB SATA |

---

## BIOS Configuration

Before installing Proxmox, the following BIOS settings were configured:

- Enabled AMD SVM (Virtualization)
- Enabled AMD IOMMU
- Enabled DOCP/XMP memory profile
- Updated motherboard BIOS to the latest stable version
- Configured the NVMe SSD as the primary boot device

These settings ensure virtualization support and prepare the system for future hardware passthrough.

---

## Installation

- Installed Proxmox VE
- Configured networking
- Logged into the web interface
- Verified network connectivity

---

## Repository Configuration

Since this homelab does not use a Proxmox Enterprise subscription, the default enterprise repository was replaced with the official no-subscription repository.

To simplify the process, I used the **Proxmox VE Post Install Script**, which:

- Disabled the enterprise repository
- Enabled the official no-subscription repository
- Refreshed package sources
- Applied recommended post-installation configuration

Afterward, the system was fully updated.

---

## Updates

The initial package update completed successfully without any issues.

After rebooting:

- Proxmox booted normally.
- The web interface remained accessible.
- Package repositories functioned correctly.
- The host was ready for virtual machine deployment.

---

## Virtual Machines

The first virtual machine deployed was:

| VM | Purpose |
|----|---------|
| TrueNAS SCALE | Storage Server |

---

## Lessons Learned

During this phase I learned:

- Basic Proxmox administration
- BIOS virtualization settings
- Repository management
- Package updates
- Initial host configuration
- Preparing a virtualization platform

---

## Remote Access

To manage the homelab remotely during the initial deployment, I installed **Tailscale** directly on the Proxmox host.

This allowed secure remote administration of the hypervisor without exposing the Proxmox web interface directly to the internet.

---

## Temporary Access to TrueNAS

During the initial TrueNAS deployment, I needed to access the TrueNAS web interface before the storage pools could be created.

Since Tailscale had not yet been installed inside the TrueNAS VM, I temporarily configured a port forwarding rule on my router to access the TrueNAS web interface remotely.

This was a temporary solution used only during the initial setup.

Once the storage pools are configured and the system is fully operational, the port forwarding rule will be removed and replaced with Tailscale running directly inside the TrueNAS VM.

This approach keeps all management traffic within the private Tailscale network and avoids exposing administrative services to the public internet.

## Next Steps

- Deploy TrueNAS SCALE
- Configure ZFS mirror pools
- Deploy Docker VM

## Troubleshooting

### Virtualization Disabled

**Problem**

The TrueNAS virtual machine failed to start due to virtualization not being enabled.

**Solution**

Enabled AMD SVM virtualization in the motherboard BIOS.

---

### Repository Configuration

**Problem**

The default Proxmox Enterprise repository requires a subscription.

**Solution**

Used the Proxmox VE Post Install Script to disable the enterprise repository and enable the official no-subscription repository.

Verified that package updates completed successfully afterward.
