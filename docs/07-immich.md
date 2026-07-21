# 📸 Immich Photo Management

> Self-hosted photo and video backup using Immich in a dedicated Proxmox LXC.

---

# 📖 Overview

Immich provides private photo and video management similar to cloud photo services.

It is used for:

* Automatic mobile photo backup
* Photo and video browsing
* Albums
* Search
* Facial recognition
* Metadata management
* Multi-user access

Immich runs in a dedicated Proxmox LXC and stores the photo library on Proxmox-managed ZFS storage.

---

# 🏗️ Architecture

```text
Mobile Device
      │
      │ Tailscale or LAN
      ▼
  Immich Server
      │
      ├── Immich application
      ├── PostgreSQL database
      ├── Machine-learning service
      └── Redis
              │
              ▼
       Proxmox ZFS Storage
              │
              ├── Photo Library
              ├── Videos
              └── Backups
```

The application and database use fast local storage.

The photo library is stored on redundant ZFS storage.

---

# 🖥️ Suggested LXC Resources

| Resource         | Suggested Allocation |
| ---------------- | -------------------- |
| CPU              | 4 cores              |
| RAM              | 6 GB to 12 GB        |
| System Disk      | 32 GB or more        |
| Photo Storage    | Proxmox ZFS dataset  |
| Network          | LAN and Tailscale    |
| Operating System | Debian or Ubuntu     |

Machine-learning features can require additional memory and CPU resources.

Resource usage depends on:

* Number of photos
* Number of users
* Video processing
* Thumbnail generation
* Facial recognition
* Smart-search indexing

---

# 📂 Storage Layout

Recommended layout:

```text
Proxmox NVMe
├── Immich application
├── PostgreSQL database
├── Redis
├── Thumbnails
└── Machine-learning cache

ZFS HDD Storage
├── Original photos
├── Original videos
└── Immich backups
```

The database should remain on fast storage when possible.

Original photos and videos require reliable redundant storage.

---

# 🔗 Proxmox Bind Mount

Create a ZFS dataset for the Immich library.

Example:

```bash
zfs create pool8tb/photos
```

Mount it inside the Immich LXC:

```bash
pct set CTID -mp0 /pool8tb/photos,mp=/data/immich
```

Example LXC configuration:

```ini
mp0: /pool8tb/photos,mp=/data/immich
```

Check the configuration:

```bash
pct config CTID
```

Inside the LXC:

```bash
ls -la /data/immich
df -h /data/immich
```

---

# 🔐 LXC Permissions

Unprivileged LXCs remap user and group IDs.

Check the Immich application user:

```bash
id
```

Check numeric ownership on the Proxmox host:

```bash
ls -ln /pool8tb/photos
```

Example mapping:

```text
Container UID 1000
Host UID 101000
```

Example ownership change:

```bash
chown -R 101000:101000 /pool8tb/photos
```

The correct UID and GID must be verified before changing ownership.

Do not apply recursive permission changes to an existing photo library without confirming the mapping.

---

# 🐳 Docker Compose Deployment

Immich can run using Docker Compose inside the LXC.

Recommended project directory:

```text
/opt/immich
```

Example structure:

```text
/opt/immich
├── docker-compose.yml
├── .env
├── postgres/
├── model-cache/
└── backups/
```

The `.env` file may contain database credentials and must not be committed to GitHub.

---

# ⚙️ Environment Configuration

Example values:

```env
UPLOAD_LOCATION=/data/immich/library
DB_DATA_LOCATION=/opt/immich/postgres

TZ=Asia/Bangkok

DB_USERNAME=immich
DB_DATABASE_NAME=immich
DB_PASSWORD=REPLACE_ME
```

Use a strong database password.

Do not publish:

* Database passwords
* API keys
* Authentication tokens
* SMTP credentials
* Private domain configuration

---

# ▶️ Starting Immich

Open the Immich directory:

```bash
cd /opt/immich
```

Validate the Compose configuration:

```bash
docker compose config
```

Pull the images:

```bash
docker compose pull
```

Start the stack:

```bash
docker compose up -d
```

Check services:

```bash
docker compose ps
```

Follow logs:

```bash
docker compose logs -f
```

---

# 🔍 Checking Containers

Show running containers:

```bash
docker ps
```

Expected Immich services may include:

```text
immich-server
immich-machine-learning
immich-postgres
immich-redis
```

Container names depend on the Compose configuration.

Check one service:

```bash
docker logs --tail 100 immich-server
```

Check the database:

```bash
docker logs --tail 100 immich-postgres
```

---

# 🌐 Accessing Immich

Immich normally listens on port:

```text
2283
```

Access through the local network:

```text
http://IMMICH_IP:2283
```

Access through Tailscale:

```text
http://IMMICH_TAILSCALE_IP:2283
```

A private HTTPS proxy can be added later if required.

Do not expose the Immich administration interface publicly without appropriate security controls.

---

# 📱 Tailscale Installation

Install Tailscale inside the Immich LXC:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Authenticate:

```bash
tailscale up
```

Check:

```bash
tailscale status
tailscale ip -4
```

The LXC requires `/dev/net/tun`.

---

# 🔌 TUN Passthrough

On the Proxmox host:

```bash
ls -l /dev/net/tun
```

Stop the Immich LXC:

```bash
pct stop CTID
```

Edit:

```bash
nano /etc/pve/lxc/CTID.conf
```

Add:

```ini
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

Start the container:

```bash
pct start CTID
```

Verify inside the LXC:

```bash
ls -l /dev/net/tun
systemctl restart tailscaled
tailscale up
```

---

# ⚠️ Tailscale Daemon Error

Possible error:

```text
Failed to connect to local Tailscale daemon
```

or:

```text
503 Service Unavailable: no backend
```

Check:

```bash
systemctl status tailscaled --no-pager
journalctl -u tailscaled -n 100 --no-pager
ls -l /dev/net/tun
```

The most common cause inside an LXC is missing TUN passthrough.

---

# 🚫 GitHub HTTP 429 Installation Error

A previous installation attempt using a Proxmox community script failed with:

```text
curl: (22) The requested URL returned error: 429
```

HTTP 429 means:

```text
Too Many Requests
```

The public IP address was temporarily rate-limited by GitHub or its content-delivery network.

---

# 🔍 Checking the GitHub Response

Test the script URL:

```bash
curl -I https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/immich.sh
```

Possible result:

```text
HTTP/2 429
```

A successful header request may show:

```text
HTTP/2 200
```

The complete installer may still fail because it downloads several additional files.

---

# ⚠️ Why the Installer Failed

The community installer did not download only one file.

It also retrieved supporting scripts during installation.

This caused repeated requests to GitHub and triggered rate limiting.

Possible causes include:

* Shared public IP address
* Carrier-grade NAT
* VPN exit address
* Repeated installer attempts
* Other devices using the same public address
* Temporary GitHub CDN throttling

---

# 🛠️ Handling HTTP 429

Recommended actions:

1. Stop repeated installation attempts.
2. Wait for the rate limit to clear.
3. Verify the public IP address.
4. Try a different network when appropriate.
5. Use the official Docker Compose installation.
6. Avoid automatically retrying the same script in a loop.

Check the public IP:

```bash
curl https://ifconfig.me
```

Do not install a permanent VPN directly on the Proxmox host only to bypass a temporary download issue.

Changing Proxmox host routing can interrupt access to all virtual machines and containers.

---

# 🧹 Removing a Failed Installation

List containers:

```bash
pct list
```

Confirm the failed Immich CT ID.

Stop it:

```bash
pct stop CTID
```

Remove it:

```bash
pct destroy CTID --purge
```

Verify it no longer exists:

```bash
pct list
```

Check the configuration directory:

```bash
ls /etc/pve/lxc
```

---

# 💽 Checking Leftover Storage

Check Proxmox storage:

```bash
pvesm list local-lvm | grep CTID
```

Check logical volumes:

```bash
lvs
```

The failed container disk should no longer exist after a successful purge.

Do not manually remove an LVM volume unless its ownership and CT ID are confirmed.

---

# 🧹 Removing Installer Files

Possible installer files:

```text
/root/immich.sh
/tmp/immich-*.log
```

Remove them:

```bash
rm -f /root/immich.sh
rm -f /tmp/immich-*.log
```

Search for remaining files:

```bash
find / -iname "*immich*" 2>/dev/null
```

Community-script template files are not the Immich application itself.

Deleting shared community-script files may affect future helper-script use and is normally unnecessary.

---

# 📱 Mobile Application

Immich provides mobile applications for photo backup.

Configure the server URL:

```text
http://IMMICH_TAILSCALE_IP:2283
```

or the configured HTTPS hostname.

Recommended mobile settings:

* Background backup enabled
* Wi-Fi-only upload if mobile data is limited
* Battery optimization disabled for Immich
* Selected albums configured
* Duplicate handling verified
* Backup status reviewed regularly

---

# 👥 User Accounts

The first registered account normally becomes the administrator.

The administrator can:

* Create users
* Configure storage limits
* Manage system settings
* Review jobs
* Control machine-learning features

Disable unnecessary public registration after the required users are created.

Use strong unique passwords.

---

# 🧠 Machine Learning

Immich machine-learning features may include:

* Facial recognition
* Smart search
* Object recognition
* CLIP-based search
* Duplicate detection

Initial processing can use significant resources.

Monitor:

```bash
docker stats
```

Check logs:

```bash
docker logs -f immich-machine-learning
```

Large libraries may require several hours or days to finish initial processing.

---

# ⚙️ Background Jobs

Immich uses background jobs for:

* Thumbnail generation
* Metadata extraction
* Video transcoding
* Facial recognition
* Smart search
* Duplicate detection

Review job status through the Immich administration interface.

If jobs become stuck:

```bash
docker compose restart immich-server
```

Check logs before restarting the entire stack.

---

# 🎞️ Video Processing

Video uploads may require:

* Metadata extraction
* Thumbnail generation
* Transcoding
* Preview generation

Video processing can consume significant CPU resources.

Monitor:

```bash
docker stats
```

Check server logs:

```bash
docker logs --tail 200 immich-server
```

---

# 💾 Backup Requirements

A complete Immich backup must include:

```text
PostgreSQL database
Original photo and video library
Required application configuration
Environment file
Optional machine-learning configuration
```

Backing up only the image library is not sufficient.

The database contains:

* Users
* Albums
* Metadata
* Asset relationships
* Facial-recognition information
* Sharing configuration

---

# 🗃️ PostgreSQL Backup

Create a database dump:

```bash
docker exec immich-postgres \
  pg_dump -U immich immich \
  > /opt/immich/backups/immich-database.sql
```

The container, database, and user names must match the actual Compose configuration.

Compress the backup:

```bash
gzip /opt/immich/backups/immich-database.sql
```

Protect the backup because it may contain personal metadata.

---

# 📂 Photo Library Backup

Example:

```bash
rsync -aHAX \
  /data/immich/library/ \
  /backup/immich-library/
```

Recommended backup destinations:

* Separate ZFS pool
* External disk
* Another server
* Encrypted cloud backup
* Off-site storage

The backup must not exist only on the same mirror as the primary library.

---

# 🔄 Restore Process

A general restore process includes:

1. Install a compatible Immich version.
2. Stop Immich services.
3. Restore the photo library.
4. Restore the PostgreSQL database.
5. Restore the environment configuration.
6. Correct ownership and permissions.
7. Start the services.
8. Verify users, albums, and photos.
9. Run a library scan if required.

The exact restore commands depend on the installed version and Compose configuration.

Test restoration before relying on the backup strategy.

---

# 🔐 Forgotten Password

Use the administration or password-reset procedure supported by the installed Immich version.

Before making manual database changes:

* Back up PostgreSQL
* Confirm the installed Immich version
* Confirm the current database schema
* Check the official Immich documentation
* Avoid manually replacing password hashes unless necessary

Direct database modification can damage authentication data when the schema has changed.

---

# 🔄 Updating Immich

Before an update:

1. Read the release notes.
2. Back up PostgreSQL.
3. Back up the environment file.
4. Confirm the photo-library mount.
5. Record the current version.
6. Check for breaking changes.

Pull new images:

```bash
cd /opt/immich
docker compose pull
```

Apply the update:

```bash
docker compose up -d
```

Check:

```bash
docker compose ps
docker compose logs --tail 100
```

Do not use automatic unattended updates for major Immich releases without reviewing changes.

---

# 🔍 Checking Storage

Check the photo dataset:

```bash
df -h /data/immich
```

Check directory usage:

```bash
du -sh /data/immich/*
```

Check the container system disk:

```bash
df -h /
```

If the ZFS bind mount fails, Immich may write photos to the LXC system disk.

Always verify the mount before starting Immich.

---

# 🔎 Mount Verification

Check whether the path is mounted:

```bash
mountpoint /data/immich
```

Expected result:

```text
/data/immich is a mountpoint
```

Check the Proxmox LXC configuration:

```bash
pct config CTID
```

The expected bind mount should be listed.

---

# 📊 Monitoring

Useful commands:

```bash
docker compose ps
docker stats
df -h
du -sh /data/immich
```

Services to monitor:

* Immich server
* PostgreSQL
* Redis
* Machine-learning container
* Photo-storage usage
* LXC disk usage
* Backup age
* Failed upload jobs

Uptime Kuma can monitor the Immich web interface.

Prometheus and Grafana can later provide container and system metrics.

---

# 🔍 Troubleshooting Checklist

Check containers:

```bash
docker compose ps
```

Check logs:

```bash
docker compose logs --tail 200
```

Check the server port:

```bash
ss -tulpn | grep 2283
```

Test locally:

```bash
curl http://127.0.0.1:2283
```

Check storage:

```bash
df -h /data/immich
mountpoint /data/immich
```

Check permissions:

```bash
ls -ln /data/immich
```

Check Tailscale:

```bash
tailscale status
```

---

# ⚠️ Common Problems

## Uploads fail

Possible causes:

* Photo-storage mount missing
* Incorrect permissions
* Storage full
* Database unavailable
* Mobile application disconnected
* Reverse-proxy upload limit

Check:

```bash
docker compose logs --tail 200 immich-server
df -h
ls -ln /data/immich
```

---

## Immich cannot connect to PostgreSQL

Check:

```bash
docker compose ps
docker logs immich-postgres
docker logs immich-server
```

Possible causes:

* Incorrect database password
* PostgreSQL container unhealthy
* Changed environment file
* Database migration failure

---

## Machine learning is unavailable

Check:

```bash
docker logs immich-machine-learning
```

Possible causes:

* Insufficient RAM
* Model download failure
* DNS problem
* Storage permissions
* Incompatible image version

---

## Mobile backup stops

Check:

* Tailscale is connected
* Immich application is allowed to run in the background
* Battery optimization is disabled
* Server URL is correct
* Storage is available
* User session is still valid

---

## Storage fills unexpectedly

Check:

```bash
df -h
du -sh /data/immich/*
du -sh /opt/immich/*
```

Possible causes:

* Missing ZFS bind mount
* Large thumbnail cache
* Video transcoding
* Duplicate uploads
* Database logs
* Backup files stored locally

---

# 🛠️ Useful Commands

```bash
# Check containers
docker compose ps

# Start Immich
docker compose up -d

# Stop Immich
docker compose down

# Pull updates
docker compose pull

# Follow logs
docker compose logs -f

# Check resources
docker stats

# Check Immich port
ss -tulpn | grep 2283

# Check storage
df -h /data/immich

# Verify mount
mountpoint /data/immich

# Check Tailscale
tailscale status

# Show Tailscale IP
tailscale ip -4

# Check LXC configuration
pct config CTID

# Create a PostgreSQL backup
docker exec immich-postgres \
  pg_dump -U immich immich \
  > immich-database.sql
```

---

# ⚠️ Important Notes

* Original photos require a separate backup.
* The PostgreSQL database must also be backed up.
* Do not store the only backup on the same ZFS pool.
* Verify the photo bind mount before starting Immich.
* Do not publish the `.env` file.
* Do not publish personal photos or database dumps.
* Review release notes before updates.
* Avoid automatic major-version updates.
* Test the restore procedure.
* Monitor storage usage regularly.
* Use Tailscale for private remote access.
* Disable public registration when it is not required.

---

# ✅ Current Status

| Task                 | Status         |
| -------------------- | -------------- |
| Immich LXC           | ✅ Complete     |
| Docker Engine        | ✅ Complete     |
| Immich services      | ✅ Running      |
| PostgreSQL           | ✅ Running      |
| Redis                | ✅ Running      |
| Machine learning     | ✅ Running      |
| ZFS photo storage    | ✅ Mounted      |
| Tailscale access     | ✅ Working      |
| Mobile photo backup  | ✅ Working      |
| User accounts        | ✅ Configured   |
| Database backup      | 🟡 In Progress |
| Photo-library backup | 🟡 In Progress |
| Automated backups    | ⏳ Planned      |
| Restore testing      | ⏳ Planned      |
| Monitoring           | ⏳ Planned      |

---

# 🚀 Future Improvements

* Automate PostgreSQL backups
* Add external photo-library backups
* Test a complete restore
* Configure Uptime Kuma monitoring
* Add storage alerts
* Add failed-upload notifications
* Monitor machine-learning jobs
* Add HTTPS hostname access
* Document mobile-device configuration
* Create a disaster-recovery procedure
