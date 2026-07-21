# 💾 Proxmox ZFS Storage

> Storage configuration using ZFS directly on the Proxmox VE host.

---

# 📖 Overview

Proxmox VE manages both virtualization and physical storage in this homelab.

A separate TrueNAS virtual machine is not used.

The physical HDDs are managed directly by Proxmox using ZFS mirror pools. ZFS datasets are mounted into Linux containers using Proxmox bind mounts.

This design reduces complexity while providing:

* Disk redundancy
* Data integrity
* Snapshots
* Compression
* Direct container access
* Centralized storage management

---

# 🖥️ Hardware Layout

| Storage       | Purpose                                 |
| ------------- | --------------------------------------- |
| 2 TB NVMe SSD | Proxmox VE, LXC disks, application data |
| 2 × 8 TB HDD  | Main ZFS mirror                         |
| 2 × 4 TB HDD  | Secondary ZFS mirror                    |

The two-disk mirror layout allows one disk in each mirror to fail without immediately losing the pool.

ZFS redundancy is not a replacement for backups.

---

# 🏗️ Storage Architecture

```text
Proxmox VE
│
├── 2 TB NVMe SSD
│   ├── Proxmox operating system
│   ├── LXC system disks
│   ├── Docker configurations
│   └── Application databases
│
├── 2 × 8 TB HDD
│   └── ZFS Mirror
│       ├── Media
│       ├── Photos
│       └── Downloads
│
└── 2 × 4 TB HDD
    └── ZFS Mirror
        ├── Documents
        └── Backups
```

---

# 🧱 ZFS Mirror Pools

A ZFS mirror stores the same data on two disks.

Advantages:

* Protection against one disk failure
* Fast read performance
* Easy disk replacement
* Data checksumming
* Snapshot support

Disadvantages:

* Only 50% of the raw capacity is usable
* Accidental deletion affects both disks
* A mirror does not protect against theft, fire, or malware

---

# 🔍 Checking Available Disks

Before creating a pool, identify the physical disks:

```bash
lsblk -o NAME,SIZE,MODEL,SERIAL
```

Additional disk information:

```bash
ls -l /dev/disk/by-id/
```

Using `/dev/disk/by-id/` is safer than using names such as `/dev/sda`, because device names may change after a reboot.

Example:

```text
/dev/disk/by-id/ata-WDC_DISK_1
/dev/disk/by-id/ata-WDC_DISK_2
```

Always verify disk serial numbers before creating or deleting a pool.

---

# 🛠️ Creating a ZFS Mirror

Example command:

```bash
zpool create -f pool8tb mirror \
  /dev/disk/by-id/DISK_1 \
  /dev/disk/by-id/DISK_2
```

For the second mirror:

```bash
zpool create -f pool4tb mirror \
  /dev/disk/by-id/DISK_3 \
  /dev/disk/by-id/DISK_4
```

Replace the disk placeholders with the correct values.

The `-f` option can overwrite existing disk metadata. It must only be used after verifying the selected disks.

---

# 📋 Checking ZFS Pools

List the pools:

```bash
zpool list
```

Check their health:

```bash
zpool status
```

Example:

```text
pool: pool8tb
state: ONLINE
config:

        NAME        STATE
        mirror-0    ONLINE
          disk1     ONLINE
          disk2     ONLINE
```

The expected state is:

```text
ONLINE
```

---

# 📂 Creating Datasets

Datasets provide separate storage areas inside a pool.

Example datasets:

```bash
zfs create pool8tb/media
zfs create pool8tb/photos
zfs create pool8tb/downloads

zfs create pool4tb/documents
zfs create pool4tb/backups
```

Check them:

```bash
zfs list
```

Example result:

```text
pool8tb
pool8tb/media
pool8tb/photos
pool8tb/downloads
pool4tb
pool4tb/documents
pool4tb/backups
```

---

# 🗂️ Dataset Structure

```text
pool8tb/
├── media/
│   ├── movies/
│   ├── shows/
│   ├── anime/
│   └── music/
├── photos/
└── downloads/
    └── qbittorrent/
        ├── completed/
        ├── incomplete/
        └── torrents/

pool4tb/
├── documents/
└── backups/
```

Create the media folders:

```bash
mkdir -p /pool8tb/media/{movies,shows,anime,music}
```

Create the download folders:

```bash
mkdir -p /pool8tb/downloads/qbittorrent/{completed,incomplete,torrents}
```

The actual paths depend on the pool mount points.

---

# 🗜️ Compression

ZFS compression reduces storage usage without requiring application changes.

Enable LZ4 compression:

```bash
zfs set compression=lz4 pool8tb
zfs set compression=lz4 pool4tb
```

Check the configuration:

```bash
zfs get compression
```

LZ4 is lightweight and suitable for most homelab workloads.

Media files are already compressed, so they may not become significantly smaller. Documents, databases, and configuration files may benefit more.

---

# ⏱️ Access Time

Updating file access time can create unnecessary disk writes.

Disable access-time updates:

```bash
zfs set atime=off pool8tb
zfs set atime=off pool4tb
```

Verify:

```bash
zfs get atime
```

This is useful for media storage and download datasets.

---

# 🔗 LXC Bind Mounts

Proxmox can mount a host directory directly inside an LXC.

Example:

```bash
pct set 201 -mp0 /pool8tb/media,mp=/data/media
```

Add downloads:

```bash
pct set 201 -mp1 /pool8tb/downloads,mp=/data/downloads
```

Example LXC configuration:

```ini
mp0: /pool8tb/media,mp=/data/media
mp1: /pool8tb/downloads,mp=/data/downloads
```

Check the container configuration:

```bash
pct config 201
```

Restart the container if needed:

```bash
pct stop 201
pct start 201
```

Verify inside the LXC:

```bash
ls -la /data
```

---

# 📺 Media Container Layout

The Jellyfin and Servarr containers should use consistent paths.

Example:

```text
/data
├── media
│   ├── movies
│   ├── shows
│   ├── anime
│   └── music
└── downloads
    └── qbittorrent
        ├── completed
        ├── incomplete
        └── torrents
```

Using consistent paths helps with:

* Sonarr imports
* Radarr imports
* Hardlinks
* Atomic moves
* Permission management
* Troubleshooting

---

# 🔐 Unprivileged LXC Permissions

Unprivileged LXCs remap user and group IDs.

Typical mapping:

```text
Container UID 0     -> Host UID 100000
Container UID 1000  -> Host UID 101000
```

Check the application user inside the LXC:

```bash
id
```

Check numeric ownership on the Proxmox host:

```bash
ls -ln /pool8tb/media
```

Example ownership change for container UID 1000:

```bash
chown -R 101000:101000 /pool8tb/media
```

Example for container root:

```bash
chown -R 100000:100000 /pool8tb/media
```

The correct UID and GID must be confirmed before changing ownership.

Do not run recursive ownership changes on important data without verifying the mapping.

---

# 📁 Vault LXC

The Vault LXC provides network file sharing.

Architecture:

```text
Proxmox ZFS Dataset
        │
        │ Bind Mount
        ▼
     Vault LXC
        │
        ├── Samba
        ├── Cockpit
        └── Tailscale
```

Example bind mount:

```bash
pct set 100 -mp0 /pool4tb/documents,mp=/srv/documents
```

Another example:

```bash
pct set 100 -mp1 /pool8tb/media,mp=/srv/media
```

---

# 🌐 Samba File Sharing

Install Samba inside the Vault LXC:

```bash
apt update
apt install samba -y
```

Enable the service:

```bash
systemctl enable --now smbd
```

Check the service:

```bash
systemctl status smbd
```

Example share configuration in `/etc/samba/smb.conf`:

```ini
[Documents]
path = /srv/documents
browseable = yes
read only = no
valid users = username

[Media]
path = /srv/media
browseable = yes
read only = no
valid users = username
```

Create a Samba password:

```bash
smbpasswd -a username
```

Restart Samba:

```bash
systemctl restart smbd
```

---

# 📱 Accessing Shares Through Tailscale

Find the Vault Tailscale address:

```bash
tailscale ip -4
```

From Windows:

```text
\\TAILSCALE_IP\Documents
```

With MagicDNS:

```text
\\vault\Documents
```

Samba ports must not be forwarded through the public router.

---

# 📸 ZFS Snapshots

Snapshots record the state of a dataset at a specific moment.

Create a snapshot:

```bash
zfs snapshot pool4tb/documents@before-change
```

List snapshots:

```bash
zfs list -t snapshot
```

Rollback example:

```bash
zfs rollback pool4tb/documents@before-change
```

Rollback removes newer changes. It should be used carefully.

Snapshots are useful before:

* Large file reorganizations
* Application migrations
* Configuration changes
* Mass deletions
* Upgrades

---

# 🧹 ZFS Scrubs

A scrub verifies stored data and repairs errors when redundant copies are available.

Start a scrub:

```bash
zpool scrub pool8tb
```

Check progress:

```bash
zpool status pool8tb
```

A monthly scrub is common for home storage.

Only one scrub per pool is required at a time.

---

# 💽 Disk Health

Check disk information:

```bash
smartctl -a /dev/sdX
```

Run a short test:

```bash
smartctl -t short /dev/sdX
```

Run a long test:

```bash
smartctl -t long /dev/sdX
```

Check test results:

```bash
smartctl -l selftest /dev/sdX
```

Disk health should also be monitored through Proxmox.

---

# 🔄 Replacing a Failed Disk

Check the pool:

```bash
zpool status
```

Replace the failed disk:

```bash
zpool replace pool8tb \
  OLD_DISK_ID \
  NEW_DISK_ID
```

Monitor resilvering:

```bash
watch -n 5 zpool status pool8tb
```

Do not remove the remaining healthy mirror disk during resilvering.

---

# 💾 Backup Strategy

ZFS mirrors protect against disk failure, but they are not backups.

Important data should have at least one additional copy.

Recommended approach:

```text
Primary Data
   |
ZFS Mirror
   |
Local Backup
   |
External or Off-Site Backup
```

Data requiring backup includes:

* Personal photos
* Documents
* Vaultwarden data
* Immich database
* Application configurations
* Minecraft worlds
* Important Docker volumes

A backup is only reliable after a successful restore test.

---

# 🔍 Useful Commands

```bash
# Show pools
zpool list

# Check pool health
zpool status

# Show datasets
zfs list

# Show snapshots
zfs list -t snapshot

# Check compression
zfs get compression

# Check disk usage
zfs list -o name,used,available,mountpoint

# Start scrub
zpool scrub POOL_NAME

# Check physical disks
lsblk -o NAME,SIZE,MODEL,SERIAL

# Check disk identifiers
ls -l /dev/disk/by-id/

# Check LXC mounts
pct config CTID
```

---

# ⚠️ Important Notes

* Always identify disks using serial numbers.
* Do not create a pool before confirming the selected drives.
* Do not treat ZFS redundancy as a backup.
* Use bind mounts for local LXC access.
* Verify UID and GID mappings before changing permissions.
* Keep application databases on the NVMe when possible.
* Keep bulk media on the HDD pools.
* Monitor SMART data and ZFS pool health.
* Test backup restoration regularly.

---

# ✅ Current Status

| Task                    | Status         |
| ----------------------- | -------------- |
| Physical disks detected | ✅ Complete     |
| ZFS mirror pools        | ✅ Complete     |
| Datasets                | ✅ Complete     |
| LXC bind mounts         | ✅ Complete     |
| Vault LXC               | ✅ Complete     |
| Samba sharing           | ✅ Complete     |
| Tailscale file access   | ✅ Complete     |
| Automated snapshots     | 🟡 In Progress |
| Automated backups       | ⏳ Planned      |
| Restore testing         | ⏳ Planned      |

---

# 🚀 Future Improvements

* Automated ZFS snapshots
* Scheduled ZFS scrubs
* SMART email alerts
* Automated external backups
* Off-site backup storage
* Dataset quotas
* Snapshot retention policies
* Backup restoration testing
* Storage monitoring dashboards
