# 01 - Proxmox Installation

## Overview

The first phase of this project was deploying Proxmox VE as the virtualization platform for the homelab.

This provides the foundation for hosting virtual machines including TrueNAS SCALE, Docker, and future game server workloads.

---

## Objectives

- Install Proxmox VE
- Configure networking
- Configure package repositories
- Update the system
- Prepare the host for virtual machines

---

## Installation Summary

The installation completed successfully without any issues.

After the initial installation I:

- Logged into the Proxmox web interface
- Configured the no-subscription repository
- Updated all packages
- Rebooted the system
- Verified network connectivity
- Uploaded the installation ISOs for future virtual machines

---

## Repository Configuration

Since this project does not use a Proxmox Enterprise subscription, the default enterprise repository was replaced with the official no-subscription repository.

To simplify the process, I used the **Proxmox VE Post Install** script maintained by the community.

The script automatically:

- Disabled the enterprise repository
- Enabled the official Proxmox no-subscription repository
- Refreshed package sources
- Applied recommended post-installation configuration

This produced the same end result as performing the configuration manually while reducing the chance of mistakes.

---

## Results

After updating and rebooting:

- Proxmox booted successfully
- The web interface remained accessible
- Package updates completed successfully
- The system was ready for virtual machine deployment

No issues were encountered during this phase.

---

## Lessons Learned

During this phase I learned:

- Basic Proxmox administration
- Repository management
- Package updates
- Initial host configuration
- Preparing a virtualization environment

---

## Next Steps

- Deploy TrueNAS SCALE
- Configure ZFS mirror pools
- Deploy the Docker virtual machine
