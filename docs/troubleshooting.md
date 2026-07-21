# 🛠️ Homelab Troubleshooting Guide

> Common problems, diagnostic commands, verified fixes, and lessons learned from the Homelab01-2026 project.

---

# 📖 Overview

This document records problems encountered while building and maintaining the homelab.

The general troubleshooting process is:

1. Identify which layer is failing.
2. Confirm the service status.
3. Check network connectivity.
4. Check DNS resolution.
5. Check ports and firewall rules.
6. Review logs.
7. Verify storage and permissions.
8. Make one change at a time.
9. Test again.
10. Document the final solution.

---

# 🧭 Troubleshooting Order

Use this order before changing configuration files:

```text
Power and hardware
        │
        ▼
Proxmox host
        │
        ▼
VM or LXC status
        │
        ▼
Network interface and routing
        │
        ▼
DNS resolution
        │
        ▼
Application service
        │
        ▼
Firewall and ports
        │
        ▼
Storage mounts and permissions
        │
        ▼
Application configuration
```

Avoid changing several unrelated settings at the same time.

---

# 🖥️ Proxmox Host Checks

Check system information:

```bash
hostname
uptime
free -h
df -h
```

Check Proxmox services:

```bash
systemctl status pveproxy --no-pager
systemctl status pvedaemon --no-pager
systemctl status pvestatd --no-pager
```

Restart the Proxmox web interface if necessary:

```bash
systemctl restart pveproxy
```

Check recent errors:

```bash
journalctl -p err -n 100 --no-pager
```

Check kernel messages:

```bash
dmesg -T | tail -n 100
```

---

# 📦 LXC Troubleshooting

List containers:

```bash
pct list
```

Check a container:

```bash
pct status CTID
pct config CTID
```

Start it:

```bash
pct start CTID
```

Stop it:

```bash
pct stop CTID
```

Reboot it:

```bash
pct reboot CTID
```

Enter it:

```bash
pct enter CTID
```

If `pct reboot` does not work:

```bash
pct stop CTID
pct start CTID
```

---

# ⚠️ LXC Fails to Start

Check the startup error:

```bash
pct start CTID --debug
```

Review the configuration:

```bash
cat /etc/pve/lxc/CTID.conf
```

Common causes include:

* Invalid bind mount
* Missing host directory
* Incorrect device passthrough
* Missing `/dev/net/tun`
* Incorrect GPU device major number
* Storage unavailable
* Invalid configuration syntax

Temporarily comment out the most recently added configuration line and test again.

---

# 🌐 Network Troubleshooting

Check interfaces:

```bash
ip -br address
```

Check routing:

```bash
ip route
```

Check the default gateway:

```bash
ip route | grep default
```

Test the gateway:

```bash
ping -c 3 ROUTER_IP
```

Test internet connectivity without DNS:

```bash
ping -c 3 1.1.1.1
```

Test DNS:

```bash
ping -c 3 example.com
```

Interpretation:

| Test                        | Result                 | Likely Issue                         |
| --------------------------- | ---------------------- | ------------------------------------ |
| Router ping fails           | No local connectivity  | Bridge, subnet, gateway or isolation |
| Router works, 1.1.1.1 fails | No internet route      | Router or gateway                    |
| 1.1.1.1 works, domain fails | DNS problem            | Resolver configuration               |
| All tests work              | Network is operational | Check application layer              |

---

# ⚠️ Destination Host Unreachable

Example:

```text
Destination Host Unreachable
```

Check:

```bash
ip -br address
ip route
ip neigh
```

Possible causes:

* Wrong subnet mask
* Wrong gateway
* Duplicate IP address
* Disconnected bridge
* Guest-network isolation
* Incorrect VLAN
* Router or Deco mode issue
* Physical link problem

Confirm that the container address is in the same subnet as the gateway.

Example:

```text
Container: 192.168.1.53/24
Gateway:   192.168.1.1
```

---

# 🌍 DNS Troubleshooting

Check the resolver:

```bash
cat /etc/resolv.conf
```

Test a hostname:

```bash
getent hosts example.com
```

Test a specific DNS server:

```bash
nslookup example.com 1.1.1.1
```

Test AdGuard Home:

```bash
nslookup example.com ADGUARD_IP
```

Using `dig`:

```bash
dig @ADGUARD_IP example.com
```

If IP connectivity works but domain names fail, the problem is DNS.

---

# ⚠️ Temporary Failure Resolving

Example:

```text
Temporary failure resolving 'archive.ubuntu.com'
```

Test:

```bash
ping -c 3 1.1.1.1
getent hosts archive.ubuntu.com
cat /etc/resolv.conf
```

Possible causes:

* Invalid DNS server
* AdGuard Home unavailable
* Tailscale DNS problem
* DNS loop
* Guest-network isolation
* Router DNS misconfiguration

---

# 🔁 DNS Loop

Problematic flow:

```text
AdGuard Home
     │
     ▼
Router
     │
     ▼
AdGuard Home
```

Recommended flow:

```text
Clients
   │
   ▼
AdGuard Home
   │
   ▼
Public Upstream DNS
```

Example upstream resolvers:

```text
1.1.1.1
1.0.0.1
9.9.9.9
```

Do not configure the router as AdGuard Home's upstream if the router forwards DNS back to AdGuard Home.

---

# 📡 TP-Link Deco Guest Network

The guest network isolates devices from the main LAN.

Possible symptoms:

* Guest devices cannot ping Proxmox
* Guest devices cannot reach AdGuard Home
* DNS times out
* Internal services are unavailable
* Main-network devices work normally

This is normally caused by guest isolation, not Proxmox.

Possible options:

* Use public DNS on the guest network
* Keep trusted devices on the main network
* Use VLAN-capable networking equipment
* Run a separate DNS resolver for guests

Do not expose DNS port 53 publicly to bypass guest isolation.

---

# 🔐 Tailscale Troubleshooting

Check the daemon:

```bash
systemctl status tailscaled --no-pager
```

Check the connection:

```bash
tailscale status
```

Check the Tailscale address:

```bash
tailscale ip -4
```

Check connection quality:

```bash
tailscale netcheck
```

Review logs:

```bash
journalctl -u tailscaled -n 100 --no-pager
```

Restart Tailscale:

```bash
systemctl restart tailscaled
```

---

# ⚠️ Tailscale Shows “Needs Login”

Example:

```text
Status: "Needs login"
```

Run:

```bash
tailscale up
```

Open the generated authentication URL.

Then confirm:

```bash
tailscale status
```

---

# ⚠️ Tailscale Cannot Create TUN Device

Example:

```text
CreateTUN("tailscale0") failed
/dev/net/tun does not exist
```

Check on the Proxmox host:

```bash
ls -l /dev/net/tun
```

Add to the LXC configuration:

```ini
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

Restart the container:

```bash
pct stop CTID
pct start CTID
```

Verify inside the container:

```bash
ls -l /dev/net/tun
systemctl restart tailscaled
tailscale up
```

Do not run `modprobe tun` inside the LXC.

Kernel modules must be managed on the Proxmox host.

---

# ⚠️ Tailscale DNS Breaks Internet Access

Check:

```bash
cat /etc/resolv.conf
```

Tailscale DNS may show:

```text
nameserver 100.100.100.100
```

Test:

```bash
getent hosts example.com
tailscale status
```

Disable Tailscale DNS on the affected device:

```bash
tailscale set --accept-dns=false
```

Only do this after configuring another working resolver.

---

# 🔒 Tailscale Serve Troubleshooting

Check the configuration:

```bash
tailscale serve status
```

Reset it:

```bash
tailscale serve reset
```

Example HTTPS proxy:

```bash
tailscale serve --https=443 http://127.0.0.1:8000
```

Test the backend first:

```bash
curl http://127.0.0.1:8000
```

If the backend does not work locally, Tailscale Serve cannot fix it.

---

# 🔌 Port Troubleshooting

Check all listening services:

```bash
ss -tulpn
```

Check one port:

```bash
ss -tulpn | grep PORT
```

Examples:

```bash
ss -tulpn | grep 8096
ss -tulpn | grep 2283
ss -tulpn | grep 8000
ss -tulpn | grep 25565
```

Interpretation:

```text
127.0.0.1:PORT
```

Accessible only locally.

```text
0.0.0.0:PORT
```

Accessible through all IPv4 interfaces unless blocked.

---

# 🔥 Firewall Troubleshooting

Check UFW:

```bash
ufw status verbose
```

Check Proxmox firewall:

```bash
pve-firewall status
```

Temporarily disabling a firewall may help identify the problem, but it should not be used as a permanent fix.

Review:

* Proxmox datacenter firewall
* Proxmox node firewall
* VM or LXC firewall
* UFW
* Router firewall
* Tailscale ACLs

---

# 💾 ZFS Troubleshooting

Check pools:

```bash
zpool list
```

Check pool health:

```bash
zpool status
```

Check datasets:

```bash
zfs list
```

Check mounted datasets:

```bash
mount | grep zfs
```

Check disk usage:

```bash
zfs list -o name,used,available,mountpoint
```

Expected pool state:

```text
ONLINE
```

---

# ⚠️ ZFS Pool Is Degraded

Check:

```bash
zpool status
```

Possible states:

```text
DEGRADED
FAULTED
UNAVAIL
```

Do not remove the remaining healthy mirror disk.

Identify the failed disk using:

```bash
lsblk -o NAME,SIZE,MODEL,SERIAL
ls -l /dev/disk/by-id/
```

Replace the failed device:

```bash
zpool replace POOL_NAME OLD_DISK NEW_DISK
```

Monitor resilvering:

```bash
watch -n 5 zpool status POOL_NAME
```

---

# ⚠️ ZFS Dataset Missing Inside an LXC

Check the LXC configuration:

```bash
pct config CTID
```

Example expected mount:

```ini
mp0: /pool/media,mp=/data/media
```

Check the host directory:

```bash
ls -la /pool/media
```

Inside the LXC:

```bash
mountpoint /data/media
df -h /data/media
```

If the mount is missing, stop applications before correcting it.

Applications may otherwise write data to the LXC system disk.

---

# 🔐 Permission Troubleshooting

Check numeric ownership:

```bash
ls -ln /data
```

Check the application user:

```bash
id
```

For a named service user:

```bash
id jellyfin
```

Unprivileged LXC mapping example:

```text
Container UID 0    → Host UID 100000
Container UID 1000 → Host UID 101000
```

Do not run recursive `chown` commands until the correct mapping is verified.

---

# 🐳 Docker Troubleshooting

Check Docker:

```bash
systemctl status docker --no-pager
```

List containers:

```bash
docker ps -a
```

Check Compose services:

```bash
docker compose ps
```

Validate configuration:

```bash
docker compose config
```

Check logs:

```bash
docker compose logs --tail 200
```

Check one container:

```bash
docker logs --tail 200 CONTAINER_NAME
```

Restart one container:

```bash
docker restart CONTAINER_NAME
```

---

# ⚠️ Docker Container Keeps Restarting

Check:

```bash
docker ps -a
docker logs --tail 200 CONTAINER_NAME
```

Possible causes:

* Invalid environment variable
* Missing directory
* Incorrect permissions
* Port conflict
* Database unavailable
* Missing secret
* Invalid Compose syntax
* Required dependency unhealthy

Inspect the container:

```bash
docker inspect CONTAINER_NAME
```

---

# ⚠️ Docker Port Already in Use

Example:

```text
Bind for 0.0.0.0:8080 failed: port is already allocated
```

Find the process:

```bash
ss -tulpn | grep 8080
```

Check Docker ports:

```bash
docker ps --format "table {{.Names}}\t{{.Ports}}"
```

Change one service to a different host port.

---

# ⚠️ Docker Cannot Access `/dev/net/tun`

Check inside the LXC:

```bash
ls -l /dev/net/tun
```

Check the Docker Compose service:

```yaml
devices:
  - /dev/net/tun:/dev/net/tun
```

For Gluetun:

```yaml
cap_add:
  - NET_ADMIN
```

The TUN device must first be passed from Proxmox into the LXC.

---

# 🛡️ Gluetun Troubleshooting

Check logs:

```bash
docker logs --tail 200 gluetun
```

Check health:

```bash
docker inspect gluetun --format '{{json .State.Health}}'
```

Test the VPN address:

```bash
docker run --rm \
  --network=container:gluetun \
  alpine:3.20 \
  sh -c "apk add --no-cache wget >/dev/null && wget -qO- https://ipinfo.io"
```

The address should not match the home public IP.

---

# ⚠️ qBittorrent Has No Internet

Check Gluetun first:

```bash
docker logs --tail 100 gluetun
```

Check qBittorrent network mode:

```bash
docker inspect qbittorrent --format '{{.HostConfig.NetworkMode}}'
```

Expected:

```text
container:gluetun
```

Possible causes:

* VPN authentication failure
* Invalid WireGuard key
* Gluetun unhealthy
* DNS failure inside Gluetun
* Missing TUN device
* VPN endpoint unavailable

---

# 🎬 Jellyfin Troubleshooting

Check the service:

```bash
systemctl status jellyfin --no-pager
```

Check logs:

```bash
journalctl -u jellyfin -n 100 --no-pager
```

Check the port:

```bash
ss -tulpn | grep 8096
```

Test locally:

```bash
curl http://127.0.0.1:8096
```

Check media mounts:

```bash
df -h /data/media
ls -la /data/media
```

---

# ⚠️ Jellyfin Cannot See Media

Check:

```bash
mountpoint /data/media
ls -ln /data/media
id jellyfin
```

Possible causes:

* Missing bind mount
* Incorrect library path
* Permission problem
* Dataset unavailable
* Wrong case in folder name

Jellyfin library paths must match the paths inside the LXC.

---

# ⚡ NVIDIA GPU Troubleshooting

On the Proxmox host:

```bash
nvidia-smi
lsmod | grep nvidia
ls -l /dev/nvidia*
```

Inside the Jellyfin LXC:

```bash
nvidia-smi
ls -l /dev/nvidia*
```

Check Jellyfin FFmpeg:

```bash
/usr/lib/jellyfin-ffmpeg/ffmpeg -hwaccels
/usr/lib/jellyfin-ffmpeg/ffmpeg -encoders | grep nvenc
```

---

# ⚠️ `/dev/nvidia-uvm` Is Missing

On the Proxmox host:

```bash
modprobe nvidia_uvm
nvidia-modprobe -u -c=0
```

Check:

```bash
ls -l /dev/nvidia*
```

Load the module at boot:

```bash
echo nvidia_uvm >> /etc/modules
```

Do not run these commands inside the LXC.

---

# ⚠️ NVIDIA Driver and Library Mismatch

Example error:

```text
Failed to initialize NVML
Driver/library version mismatch
```

Check the Proxmox host:

```bash
nvidia-smi
cat /proc/driver/nvidia/version
```

Check inside the LXC:

```bash
nvidia-smi
dpkg -l | grep nvidia
```

The LXC userspace libraries must be compatible with the host NVIDIA driver.

Do not install a separate NVIDIA kernel driver inside the LXC.

---

# ⚠️ FFmpeg Exit Code 187

Possible causes:

* Missing NVIDIA UVM device
* Incorrect device passthrough
* Library mismatch
* Missing NVENC libraries
* Permission problem
* Unsupported acceleration setting

Check:

```bash
ls -l /dev/nvidia*
nvidia-smi
/usr/lib/jellyfin-ffmpeg/ffmpeg -hwaccels
```

---

# 🔐 Vaultwarden Troubleshooting

Check Vaultwarden:

```bash
systemctl status vaultwarden --no-pager
```

Check logs:

```bash
journalctl -u vaultwarden -n 100 --no-pager
```

Test locally:

```bash
curl http://127.0.0.1:8000
```

Check the port:

```bash
ss -tlnp | grep 8000
```

Check Tailscale Serve:

```bash
tailscale serve status
```

---

# ⚠️ Vaultwarden Fails After TLS Change

Incorrect:

```env
ROCKET_TLS=
```

Possible error:

```text
invalid type: found string "", expected struct TlsConfig
```

Fix:

Remove the entire `ROCKET_TLS` line.

Use Vaultwarden HTTP internally:

```text
http://127.0.0.1:8000
```

Use Tailscale Serve for external HTTPS:

```bash
tailscale serve --https=443 http://127.0.0.1:8000
```

---

# ⚠️ Vaultwarden Shows Insecure Context

Do not use:

```text
http://TAILSCALE_IP:8000
```

Use:

```text
https://vaultwarden.example-tailnet.ts.net
```

The web vault requires HTTPS for browser cryptography.

---

# 📸 Immich Troubleshooting

Check containers:

```bash
docker compose ps
```

Check logs:

```bash
docker compose logs --tail 200
```

Check the Immich port:

```bash
ss -tulpn | grep 2283
```

Test locally:

```bash
curl http://127.0.0.1:2283
```

Check photo storage:

```bash
mountpoint /data/immich
df -h /data/immich
```

---

# ⚠️ Immich Installer Returns HTTP 429

Example:

```text
curl: (22) The requested URL returned error: 429
```

Meaning:

```text
Too Many Requests
```

Possible causes:

* Repeated script attempts
* Shared public IP
* Carrier-grade NAT
* VPN exit IP
* Temporary GitHub rate limiting

Recommended actions:

* Stop retrying repeatedly
* Wait before trying again
* Use the official Docker Compose deployment
* Try another network if appropriate
* Avoid changing Proxmox routing only to bypass the limit

---

# ⚠️ Failed Immich Installation Cleanup

List containers:

```bash
pct list
```

Stop the failed container:

```bash
pct stop CTID
```

Delete it:

```bash
pct destroy CTID --purge
```

Check leftover storage:

```bash
pvesm list local-lvm | grep CTID
lvs
```

Do not manually remove storage until the CT ID is confirmed.

---

# ⚠️ Immich Uploads Fill the LXC Disk

Check:

```bash
df -h /
df -h /data/immich
mountpoint /data/immich
```

If `/data/immich` is not mounted, Immich may create a normal local directory and store uploads on the LXC system disk.

Stop Immich before repairing the bind mount.

---

# 🌐 AdGuard Home Troubleshooting

Check the service:

```bash
systemctl status AdGuardHome --no-pager
```

Check DNS port 53:

```bash
ss -tulpn | grep ':53'
```

Test locally:

```bash
nslookup example.com 127.0.0.1
```

Test remotely:

```bash
nslookup example.com ADGUARD_IP
```

Check logs:

```bash
journalctl -u AdGuardHome -n 100 --no-pager
```

---

# ⚠️ Clients Do Not Use AdGuard Home

Check the client DNS configuration.

Linux:

```bash
cat /etc/resolv.conf
```

Windows:

```cmd
ipconfig /all
```

Possible causes:

* Router still provides public DNS
* Browser secure DNS enabled
* Android Private DNS enabled
* Apple Private Relay enabled
* VPN application overrides DNS
* DHCP lease not renewed
* Guest-network isolation

---

# ⚠️ AdGuard Works Locally but Not from Clients

Check:

```bash
ss -tulpn | grep ':53'
ufw status
ip -br address
```

Possible causes:

* AdGuard listening only on localhost
* Firewall blocking port 53
* Wrong DNS address in DHCP
* Incorrect subnet
* Guest isolation
* Duplicate IP address

---

# 🎮 AMP Troubleshooting

Check the AMP LXC:

```bash
pct status CTID
pct config CTID
```

Inside the LXC:

```bash
free -h
df -h
ip -br address
ss -tulpn
```

Check Tailscale:

```bash
systemctl status tailscaled --no-pager
tailscale status
```

Check AMP processes:

```bash
ps aux | grep -i amp
```

---

# ⚠️ AMP Interface Cannot Be Reached

Possible causes:

* Wrong IP address
* Wrong port
* AMP service stopped
* Firewall blocking access
* HTTP and HTTPS mismatch
* Tailscale not authenticated

Check:

```bash
ip -br address
ss -tulpn
tailscale status
```

---

# ⚠️ Minecraft Server Does Not Start

Check the AMP console.

Possible causes:

* Incorrect Java version
* Insufficient RAM
* Port conflict
* Invalid mod
* Missing dependency
* Damaged world
* Incorrect server version
* Licence activation problem

Check Java:

```bash
java -version
```

Check the game port:

```bash
ss -tulpn | grep 25565
```

---

# ⚠️ Minecraft Players Cannot Connect

Verify:

* Server is running
* Correct IP is used
* Correct port is used
* Player has Tailscale access when required
* Firewall allows the port
* Player is whitelisted
* Minecraft versions match
* Modpacks match

Test the port:

```bash
nc -zv AMP_IP 25565
```

---

# 💽 Disk Space Troubleshooting

Check filesystems:

```bash
df -h
```

Check directories:

```bash
du -sh /*
```

For a specific directory:

```bash
du -sh /var/lib/*
du -sh /docker/*
du -sh /data/*
```

Check Docker usage:

```bash
docker system df
```

Remove unused Docker images carefully:

```bash
docker image prune
```

Do not run aggressive Docker cleanup commands without checking what will be deleted.

---

# 📋 Log Troubleshooting

System logs:

```bash
journalctl -n 100 --no-pager
```

Errors only:

```bash
journalctl -p err -n 100 --no-pager
```

Service logs:

```bash
journalctl -u SERVICE_NAME -n 100 --no-pager
```

Follow logs:

```bash
journalctl -u SERVICE_NAME -f
```

Docker logs:

```bash
docker logs --tail 100 CONTAINER_NAME
```

Kernel logs:

```bash
dmesg -T | tail -n 100
```

---

# 🔄 Safe Change Procedure

Before changing a working service:

1. Record the current configuration.
2. Create a backup.
3. Take a snapshot when appropriate.
4. Change one setting.
5. Restart only the affected service.
6. Review logs.
7. Test locally.
8. Test through the LAN.
9. Test through Tailscale.
10. Roll back if necessary.

---

# 🧪 Local-to-Remote Testing Method

Test the application in this order:

```text
1. localhost
2. LAN address
3. Tailscale address
4. MagicDNS hostname
5. External client
```

Example:

```bash
curl http://127.0.0.1:PORT
curl http://LAN_IP:PORT
curl http://TAILSCALE_IP:PORT
```

This identifies where the connection fails.

---

# 📋 Useful Diagnostic Bundle

Run these commands when requesting help:

```bash
hostname
date
uptime
ip -br address
ip route
cat /etc/resolv.conf
ping -c 3 1.1.1.1
getent hosts example.com
ss -tulpn
df -h
free -h
systemctl --failed
journalctl -p err -n 50 --no-pager
```

For an LXC, also include:

```bash
pct status CTID
pct config CTID
```

For Docker:

```bash
docker ps -a
docker compose ps
docker compose logs --tail 100
```

Remove passwords, keys, tokens, and private information before sharing output.

---

# 🔐 Information to Remove from Logs

Never publish:

```text
Passwords
API keys
Tailscale authentication keys
WireGuard private keys
VPN configuration files
AMP licence keys
Database passwords
SMTP passwords
Session cookies
Private certificates
Public IP addresses when privacy is required
Personal file names
Vaultwarden database content
Immich personal metadata
```

Use placeholders such as:

```text
REDACTED
TAILSCALE_IP
LAN_IP
DATABASE_PASSWORD
```

---

# 📚 Lessons Learned

* Test IP connectivity before blaming DNS.
* Test DNS before reinstalling an application.
* LXC containers share the Proxmox kernel.
* Kernel modules must be loaded on the Proxmox host.
* `/dev/net/tun` must be passed into LXCs.
* ZFS bind mounts must be verified before applications start.
* Missing mounts can fill LXC system disks.
* Unprivileged LXCs remap user and group IDs.
* Tailscale DNS can override normal DNS settings.
* Guest networks may block access to internal DNS.
* NVIDIA userspace libraries must match the host driver.
* Vaultwarden requires HTTPS for browser cryptography.
* HTTP 429 errors do not necessarily mean an installer is broken.
* Docker services should use consistent paths.
* ZFS mirrors provide redundancy, not backups.
* A backup is incomplete until restoration has been tested.

---

# ✅ Current Troubleshooting Status

| Area                         | Status       |
| ---------------------------- | ------------ |
| Proxmox diagnostics          | ✅ Documented |
| LXC troubleshooting          | ✅ Documented |
| Network troubleshooting      | ✅ Documented |
| DNS troubleshooting          | ✅ Documented |
| Tailscale troubleshooting    | ✅ Documented |
| ZFS troubleshooting          | ✅ Documented |
| Docker troubleshooting       | ✅ Documented |
| Jellyfin GPU troubleshooting | ✅ Documented |
| Vaultwarden troubleshooting  | ✅ Documented |
| Immich troubleshooting       | ✅ Documented |
| AdGuard troubleshooting      | ✅ Documented |
| AMP troubleshooting          | ✅ Documented |
| Automated alerting           | ⏳ Planned    |
| Centralized logging          | ⏳ Planned    |

---

# 🚀 Future Improvements

* Add screenshots of common errors
* Add real network diagrams
* Add automatic service-health scripts
* Add Uptime Kuma alerts
* Add centralized logging
* Add Grafana troubleshooting dashboards
* Add automated mount checks
* Add disk-health notifications
* Add backup-failure alerts
* Add recovery checklists for each service
