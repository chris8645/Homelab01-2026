# 🎮 AMP Game Servers

> Self-hosted game-server management using CubeCoders AMP in a dedicated Proxmox LXC.

---

# 📖 Overview

CubeCoders AMP is used to create, configure, monitor, and manage game-server instances from a web interface.

AMP runs in a dedicated Proxmox LXC and is accessed privately through the local network or Tailscale.

The first game server being deployed is Minecraft.

Future game servers may include:

* Palworld
* Satisfactory
* Project Zomboid
* Valheim
* Additional Minecraft instances

---

# 🏗️ Architecture

```text
Players
   │
   │ LAN or approved remote access
   ▼
AMP LXC
   │
   ├── AMP Web Interface
   ├── Minecraft Instance
   ├── Future Game Servers
   ├── Instance Backups
   └── Tailscale
```

AMP separates each game server into its own managed instance.

This allows individual control of:

* CPU allocation
* Memory limits
* Game ports
* Server versions
* Startup parameters
* Backups
* User permissions
* Console access
* Scheduled tasks

---

# 🖥️ Suggested LXC Resources

Initial allocation:

| Resource         | Suggested Allocation          |
| ---------------- | ----------------------------- |
| CPU              | 4 to 6 cores                  |
| RAM              | 16 GB to 32 GB                |
| System Disk      | 64 GB or more                 |
| Game Storage     | NVMe or dedicated ZFS dataset |
| Network          | LAN and Tailscale             |
| Operating System | Debian or Ubuntu              |

The final allocation depends on:

* Number of game servers
* Number of players
* Minecraft modpacks
* World size
* View distance
* Plugins
* Server software
* Concurrent instances

The LXC does not need all available memory assigned immediately.

Resources can be increased later from Proxmox.

---

# 📂 Storage Layout

Recommended layout:

```text
AMP LXC
├── AMP application
├── Instance configurations
├── Game-server files
├── Minecraft worlds
├── Plugins
├── Mods
├── Logs
└── Temporary files

Backup Storage
├── AMP configuration backups
├── Minecraft world backups
└── Instance archives
```

Application files should use fast storage.

Important world backups should be copied outside the AMP LXC.

---

# 🔍 Checking the AMP LXC

From the Proxmox host:

```bash
pct list
```

Check the AMP container configuration:

```bash
pct config CTID
```

Check its status:

```bash
pct status CTID
```

Enter the container:

```bash
pct enter CTID
```

Check system resources:

```bash
free -h
df -h
nproc
```

---

# 🌐 Network Configuration

The AMP LXC should use a stable IP address.

Recommended options:

* Static IP configured in Proxmox
* DHCP reservation configured in the router
* Static configuration inside the LXC

Example LXC network configuration:

```ini
net0: name=eth0,bridge=vmbr0,ip=192.168.1.60/24,gw=192.168.1.1,type=veth
```

Replace the example address with the actual AMP address.

---

# 🔐 Tailscale Remote Access

Tailscale provides private remote access to the AMP management interface.

Install Tailscale:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Authenticate:

```bash
tailscale up
```

Check the service:

```bash
systemctl status tailscaled --no-pager
```

Check the connection:

```bash
tailscale status
```

Show the Tailscale IPv4 address:

```bash
tailscale ip -4
```

---

# ⚠️ Tailscale Needs Login

A status similar to this means the daemon is running but the device is not authenticated:

```text
Status: "Needs login"
```

Run:

```bash
tailscale up
```

Open the generated authentication link and approve the AMP device.

Verify again:

```bash
tailscale status
```

---

# 🔌 LXC TUN Passthrough

Tailscale requires:

```text
/dev/net/tun
```

On the Proxmox host:

```bash
ls -l /dev/net/tun
```

Stop the AMP LXC:

```bash
pct stop CTID
```

Edit its configuration:

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

# 🔑 AMP Licence

AMP requires a CubeCoders licence key.

The licence key is linked to the purchased AMP licence tier.

The key is required before creating or activating managed instances.

Do not publish the licence key in:

* GitHub
* Screenshots
* Documentation
* Public logs
* Docker Compose files
* Shell history examples

Use a placeholder in documentation:

```text
AMP_LICENCE_KEY=REPLACE_ME
```

---

# 📦 AMP Licence Planning

The required licence depends on:

* Number of AMP installations
* Number of managed instances
* Required management features
* Commercial or personal use
* Number of administrators

For a personal Minecraft server, the licence only needs to support the planned number of instances and administrators.

A more advanced licence may be useful later if the homelab expands to several game servers or additional AMP installations.

Licence pricing and upgrade conditions should always be checked directly through the CubeCoders account before purchasing.

---

# 🌐 AMP Web Interface

AMP is primarily managed through its web interface.

Access it using the AMP LXC address and the port configured during installation.

Example:

```text
http://AMP_LAN_IP:AMP_PORT
```

Through Tailscale:

```text
http://AMP_TAILSCALE_IP:AMP_PORT
```

With MagicDNS:

```text
http://amp:AMP_PORT
```

The exact port should be confirmed from the AMP installation output or AMP configuration.

---

# 👤 AMP Administrator Account

During the initial setup, create an administrator account.

Use:

* A unique username
* A strong password
* Multi-factor authentication if supported
* A separate non-administrator account for normal use

Do not reuse the Proxmox root password.

Do not publish administrator credentials.

---

# 📦 Creating a Game Instance

AMP instances should normally be created through the web interface.

General procedure:

1. Open the AMP administration interface.
2. Open the instance manager.
3. Select Create Instance.
4. Select the game or application template.
5. Enter an instance name.
6. Assign CPU and memory limits.
7. Select the game port.
8. Select the storage location.
9. Enter the AMP licence information if requested.
10. Create the instance.
11. Start the instance.
12. Open the instance management page.

Example instance name:

```text
Minecraft-01
```

Use clear names when several servers are planned.

---

# 🧱 Minecraft Instance

The first AMP instance is used for Minecraft.

Important configuration choices include:

| Setting         | Example                                    |
| --------------- | ------------------------------------------ |
| Instance Name   | Minecraft-01                               |
| Game Port       | 25565                                      |
| Maximum Players | Based on available resources               |
| Memory          | 6 GB to 12 GB                              |
| Java Version    | Based on Minecraft version                 |
| Server Type     | Vanilla, Paper, Fabric, Forge, or NeoForge |
| World Name      | Dedicated server world                     |
| Online Mode     | Enabled                                    |
| Whitelist       | Recommended                                |
| Backups         | Enabled                                    |

The correct Java version depends on the selected Minecraft version.

---

# ☕ Java Version

Minecraft versions require compatible Java versions.

Check the installed Java version:

```bash
java -version
```

List Java installations:

```bash
update-alternatives --list java
```

AMP may install or manage a compatible Java runtime automatically, depending on the instance template.

Do not manually replace Java unless the selected Minecraft server version requires it.

---

# 🧩 Minecraft Server Types

Common options include:

## Vanilla

Official Minecraft server.

Advantages:

* Simple
* Official
* Good compatibility

Disadvantages:

* Limited performance optimization
* No plugin support

## Paper

Performance-focused server based on Spigot.

Advantages:

* Better performance
* Plugin support
* Large community

Disadvantages:

* Some gameplay behavior may differ slightly from Vanilla

## Fabric

Lightweight mod loader.

Advantages:

* Good performance
* Many optimization mods
* Suitable for lightweight modpacks

Disadvantages:

* Requires Fabric-compatible mods

## Forge or NeoForge

Mod loaders commonly used by larger modpacks.

Advantages:

* Large mod ecosystem
* Supports complex modpacks

Disadvantages:

* Higher memory usage
* Longer startup times
* More complicated troubleshooting

---

# 📦 Modpacks and CurseForge

Modpack installation depends on:

* Minecraft version
* Mod loader
* AMP application template
* Available AMP integrations
* Modpack distribution method
* CurseForge download restrictions

Before installing a modpack:

1. Confirm the Minecraft version.
2. Confirm whether it uses Forge, NeoForge, or Fabric.
3. Confirm the required Java version.
4. Read the modpack server installation instructions.
5. Confirm the required RAM.
6. Back up the current server.
7. Stop the instance before replacing files.

Some modpacks provide a separate server pack.

The server pack should be used instead of copying the complete client installation.

---

# ⚙️ Minecraft Memory

Suggested starting allocations:

| Server Type        | Suggested RAM          |
| ------------------ | ---------------------- |
| Vanilla            | 4 GB to 6 GB           |
| Paper              | 4 GB to 8 GB           |
| Lightweight Fabric | 6 GB to 8 GB           |
| Medium Modpack     | 8 GB to 12 GB          |
| Large Modpack      | 12 GB to 16 GB or more |

Do not allocate all LXC memory to one Minecraft instance.

The operating system and AMP also require memory.

Example:

```text
AMP LXC RAM: 32 GB
Minecraft Instance: 10 GB
Remaining Capacity: AMP, OS, cache, and future instances
```

---

# 🌍 Minecraft Network Port

The default Java Edition port is:

```text
25565/TCP
```

Check whether the server is listening:

```bash
ss -tulpn | grep 25565
```

Test locally:

```bash
nc -zv 127.0.0.1 25565
```

Test from another LAN device:

```bash
nc -zv AMP_LAN_IP 25565
```

Replace the example port if AMP assigned a different one.

---

# 🏠 LAN Access

Players on the local network can connect using:

```text
AMP_LAN_IP:25565
```

Example:

```text
192.168.1.60:25565
```

A static IP or DHCP reservation is recommended so the server address does not change.

---

# 📱 Tailscale Player Access

Approved Tailscale users can connect using:

```text
AMP_TAILSCALE_IP:25565
```

or MagicDNS:

```text
amp:25565
```

Every remote player must be connected to the same tailnet unless another networking method is configured.

Tailscale is suitable for private servers with a limited number of trusted users.

---

# 🌐 Public Server Access

A public Minecraft server normally requires:

* Router port forwarding
* Public DNS or a public IP
* Firewall rules
* Strong server security
* Continuous updates
* DDoS considerations
* Player moderation

For this homelab, private access through Tailscale is preferred during initial testing.

Do not expose the AMP administration interface publicly.

---

# 🔥 Firewall

Check UFW:

```bash
ufw status
```

Allow Minecraft from the local network:

```bash
ufw allow from LAN_SUBNET to any port 25565 proto tcp
```

Allow Minecraft through Tailscale:

```bash
ufw allow in on tailscale0 to any port 25565 proto tcp
```

Do not expose the AMP management port publicly.

Firewall rules only apply when the firewall is active.

---

# 👥 Minecraft Security

Recommended settings:

```text
online-mode=true
white-list=true
enable-command-block=false
enable-rcon=false
```

Enable RCON only when it is specifically required and secured.

Recommended practices:

* Use a whitelist
* Keep online mode enabled
* Give operator permissions only to trusted users
* Back up before adding plugins or mods
* Keep the server updated
* Review logs for failed login attempts
* Do not reuse administrator passwords

---

# 📋 Whitelist

Enable the whitelist from the AMP interface or Minecraft console.

Typical Minecraft commands:

```text
whitelist on
whitelist add PLAYER_NAME
whitelist remove PLAYER_NAME
whitelist list
```

Use the exact Minecraft account name.

---

# 👑 Operator Permissions

Grant operator access:

```text
op PLAYER_NAME
```

Remove operator access:

```text
deop PLAYER_NAME
```

Operator access allows powerful administrative commands.

Only trusted users should be operators.

---

# 💾 Backups

Minecraft backups should include:

* World folders
* Server configuration
* Plugin configuration
* Mods
* Whitelist
* Operator list
* Ban lists
* AMP instance configuration

Important world files include:

```text
world/
world_nether/
world_the_end/
```

Modded servers may use different world directories.

---

# ⏱️ Scheduled Backups

AMP can schedule backups through its task system.

Recommended schedule:

```text
Daily incremental or archive backup
Weekly full backup
Backup before updates
Backup before installing mods
Backup before changing server software
```

Backups should be copied outside the AMP LXC.

A backup stored only inside the same LXC can be lost if the container disk fails.

---

# 📂 External Backup Location

Recommended backup flow:

```text
Minecraft Instance
       │
       ▼
AMP Local Backup
       │
       ▼
Proxmox ZFS Backup Dataset
       │
       ▼
External or Off-Site Backup
```

Possible backup destinations:

* Separate ZFS dataset
* External USB disk
* Another server
* Encrypted cloud storage
* Proxmox Backup Server

---

# 🔄 Updating Minecraft

Before updating:

1. Stop the server.
2. Create a complete backup.
3. Record the current version.
4. Check mod and plugin compatibility.
5. Update the server software.
6. Start the server.
7. Review the console.
8. Test player login.
9. Confirm the world loaded correctly.

Do not update a modded server until every required mod supports the new version.

---

# 🔄 Updating AMP

Before updating AMP:

1. Back up important instances.
2. Stop active game servers.
3. Review available update information.
4. Apply the AMP update.
5. Restart AMP if required.
6. Check every instance.
7. Confirm the web interface works.
8. Test the Minecraft server.

Avoid updating AMP immediately before an important multiplayer session.

---

# 📊 Monitoring

AMP provides instance monitoring for:

* CPU usage
* Memory usage
* Server status
* Player count
* Console output
* Scheduled tasks
* Backups

Additional monitoring can later include:

* Uptime Kuma
* Prometheus
* Grafana
* Proxmox host metrics
* Storage alerts
* Backup-age alerts

---

# 🔍 Troubleshooting Checklist

Check the LXC:

```bash
pct status CTID
```

Check network addresses:

```bash
ip -br address
ip route
```

Check Tailscale:

```bash
systemctl status tailscaled --no-pager
tailscale status
tailscale ip -4
```

Check memory:

```bash
free -h
```

Check storage:

```bash
df -h
```

Check listening ports:

```bash
ss -tulpn
```

Check Java:

```bash
java -version
```

Check AMP processes:

```bash
ps aux | grep -i amp
```

---

# ⚠️ Common Problems

## AMP shows “Needs login” for Tailscale

Run:

```bash
tailscale up
```

Open the authentication URL.

Then check:

```bash
tailscale status
```

---

## AMP interface cannot be reached

Check:

```bash
ip -br address
ss -tulpn
```

Possible causes:

* Incorrect IP address
* Incorrect AMP port
* AMP service not running
* Firewall blocking the connection
* Browser using HTTPS when AMP expects HTTP
* Tailscale disconnected

---

## Minecraft server does not start

Check the AMP console.

Possible causes:

* Incorrect Java version
* Insufficient memory
* Corrupted mod
* Missing dependency
* Port already in use
* Invalid configuration
* Failed world upgrade
* Licence activation problem

---

## Minecraft port is already in use

Check:

```bash
ss -tulpn | grep 25565
```

Another instance may already be using the port.

Assign a different port to one instance.

Example:

```text
Minecraft-01: 25565
Minecraft-02: 25566
Minecraft-03: 25567
```

---

## Server runs but players cannot connect

Check:

* Correct server IP
* Correct port
* Server is fully started
* Player is connected to Tailscale when required
* Firewall permits the connection
* Whitelist contains the player
* Minecraft versions match
* Modpacks match between server and client

---

## Modded server crashes

Check the AMP console and Minecraft logs.

Common causes:

* Client-only mod installed on the server
* Missing dependency
* Incorrect mod version
* Incorrect Java version
* Insufficient memory
* Incompatible mods
* Incorrect Forge, NeoForge, or Fabric version

Remove only the most recently added changes after creating a backup.

---

## Server reports out of memory

Increase the instance memory allocation or reduce:

* View distance
* Simulation distance
* Player count
* Number of loaded chunks
* Mod count
* Plugin count

Do not allocate more memory than the LXC can provide.

---

## World is corrupted

Do not repeatedly restart the server.

Procedure:

1. Stop the instance.
2. Copy the damaged world.
3. Preserve the latest logs.
4. Restore the latest known-good backup.
5. Test the restored world.
6. Investigate the cause separately.

---

# 🛠️ Useful Commands

```bash
# Check AMP LXC
pct status CTID
pct config CTID
pct enter CTID

# Check resources
free -h
df -h
nproc

# Check network
ip -br address
ip route
ss -tulpn

# Check Tailscale
systemctl status tailscaled --no-pager
tailscale status
tailscale ip -4
tailscale netcheck

# Check Java
java -version

# Check Minecraft port
ss -tulpn | grep 25565

# Check AMP processes
ps aux | grep -i amp

# Check recent system logs
journalctl -n 100 --no-pager
```

---

# ⚠️ Important Notes

* Do not commit the AMP licence key.
* Do not expose the AMP administration interface publicly.
* Use a static IP or DHCP reservation.
* Use Tailscale for private management.
* Use a Minecraft whitelist.
* Keep online mode enabled.
* Back up worlds before updates.
* Keep backups outside the AMP LXC.
* Confirm Java compatibility before changing versions.
* Test modpacks before inviting players.
* Monitor RAM and storage usage.
* Document every port assigned to an instance.

---

# ✅ Current Status

| Task                     | Status         |
| ------------------------ | -------------- |
| AMP LXC                  | ✅ Complete     |
| AMP installed            | ✅ Complete     |
| AMP web interface        | ✅ Working      |
| AMP licence              | ✅ Activated    |
| Tailscale installed      | ✅ Complete     |
| Tailscale authentication | ✅ Working      |
| Minecraft instance       | 🟡 In Progress |
| Minecraft networking     | 🟡 In Progress |
| Minecraft backups        | ⏳ Planned      |
| Additional game servers  | ⏳ Planned      |
| External backup copy     | ⏳ Planned      |
| Uptime monitoring        | ⏳ Planned      |

---

# 🚀 Future Improvements

* Complete Minecraft deployment
* Configure automatic Minecraft backups
* Copy backups to ZFS storage
* Add Uptime Kuma monitoring
* Add player-count monitoring
* Add resource alerts
* Test restore procedures
* Add a second Minecraft instance
* Test a modded Minecraft server
* Evaluate additional game-server templates
* Document public-access options
* Add Tailscale access-control rules
