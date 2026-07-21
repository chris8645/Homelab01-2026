# 🌐 Networking and Tailscale

> Network configuration, private remote access, DNS troubleshooting, and LXC TUN passthrough.

---

# 📖 Overview

The homelab network connects Proxmox VE and its containers to the home network through a TP-Link Deco system.

Tailscale provides secure remote access to internal services without exposing administration ports directly to the internet.

The network also includes AdGuard Home for DNS filtering.

---

# 🏗️ Network Architecture

```text
                           Internet
                               │
                         ISP Connection
                               │
                        TP-Link Deco Router
                               │
                          Proxmox VE Host
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
      LXCs                 ZFS Storage          Docker Services
        │
        ├── Vault
        ├── Jellyfin
        ├── Servarr
        ├── Vaultwarden
        ├── Immich
        ├── AdGuard Home
        └── AMP
```

Tailscale creates a private encrypted network between approved devices.

```text
Remote Device
     │
     │ Tailscale
     ▼
Homelab Service
```

---

# 🖥️ Proxmox Network Bridge

Proxmox uses a Linux bridge to connect virtual machines and containers to the physical network.

The main bridge is normally:

```text
vmbr0
```

Example configuration:

```ini
auto vmbr0
iface vmbr0 inet static
    address PROXMOX_IP/24
    gateway ROUTER_IP
    bridge-ports PHYSICAL_INTERFACE
    bridge-stp off
    bridge-fd 0
```

The actual interface name may be similar to:

```text
eno1
enp4s0
eth0
```

Check the current configuration:

```bash
cat /etc/network/interfaces
```

Check available addresses:

```bash
ip -br address
```

Check routing:

```bash
ip route
```

---

# 📍 IP Addressing

Infrastructure services should use predictable addresses.

Recommended options:

* Static IP configured inside the guest
* DHCP reservation configured on the router
* Proxmox LXC network configuration with a fixed address

Example LXC network configuration:

```ini
net0: name=eth0,bridge=vmbr0,ip=192.168.1.50/24,gw=192.168.1.1,type=veth
```

Avoid configuring the same static IP on multiple devices.

---

# 🔐 Tailscale

Tailscale provides private remote access using WireGuard-based encrypted connections.

It is used to access services such as:

* Proxmox
* Vault
* Jellyfin
* Servarr
* Vaultwarden
* Immich
* AMP
* Cockpit
* Portainer

Advantages:

* No public management ports
* No normal router port forwarding
* Encrypted device-to-device traffic
* Stable private IP addresses
* MagicDNS hostnames
* Access control support

---

# 📦 Installing Tailscale

Install Tailscale on Debian or Ubuntu:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Start authentication:

```bash
tailscale up
```

Open the generated login URL and approve the device.

Check status:

```bash
tailscale status
```

Show the IPv4 address:

```bash
tailscale ip -4
```

Check network connectivity:

```bash
tailscale netcheck
```

---

# ⚙️ Tailscale Service

Check the daemon:

```bash
systemctl status tailscaled --no-pager
```

Enable it at boot:

```bash
systemctl enable --now tailscaled
```

Restart it:

```bash
systemctl restart tailscaled
```

Check logs:

```bash
journalctl -u tailscaled -n 100 --no-pager
```

A status similar to this means the service is running but authentication is still required:

```text
Needs login
```

Run:

```bash
tailscale up
```

---

# 🔌 TUN Device for LXC Containers

Tailscale requires access to:

```text
/dev/net/tun
```

LXC containers share the Proxmox host kernel. They cannot load kernel modules themselves.

Check the TUN device on the Proxmox host:

```bash
ls -l /dev/net/tun
```

Expected result:

```text
crw-rw-rw- 1 root root 10, 200 /dev/net/tun
```

---

# 🛠️ Passing TUN into an LXC

Stop the container:

```bash
pct stop CTID
```

Edit the container configuration:

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

Verify inside the container:

```bash
ls -l /dev/net/tun
```

Restart Tailscale:

```bash
systemctl restart tailscaled
tailscale up
```

---

# ⚠️ Common TUN Error

Example error:

```text
CreateTUN("tailscale0") failed
/dev/net/tun does not exist
```

Cause:

The TUN device is available on the Proxmox host but is not passed into the LXC.

The fix must be applied on the Proxmox host, not inside the container.

Running this inside an LXC will normally fail:

```bash
modprobe tun
```

Containers cannot load their own kernel modules.

---

# 🌍 Accessing Services Through Tailscale

Example using a Tailscale IP:

```text
https://100.x.x.x:8006
```

Example services:

```text
Proxmox:
https://TAILSCALE_IP:8006

Jellyfin:
http://TAILSCALE_IP:8096

Portainer:
https://TAILSCALE_IP:9443

Cockpit:
https://TAILSCALE_IP:9090
```

Some applications require HTTPS or use different ports.

Always confirm the actual service port before connecting.

---

# 🏷️ MagicDNS

MagicDNS allows devices to use hostnames instead of Tailscale IP addresses.

Examples:

```text
https://proxmox:8006
http://jellyfin:8096
\\vault\Shared
```

MagicDNS must be enabled in the Tailscale administration console.

Check the current tailnet names:

```bash
tailscale status
```

---

# 🔒 Tailscale Serve

Tailscale Serve can provide private HTTPS access to a local application.

Example for Vaultwarden:

```bash
tailscale serve reset
tailscale serve --https=443 http://127.0.0.1:8000
```

Check the configuration:

```bash
tailscale serve status
```

Architecture:

```text
Browser
   │
   │ HTTPS
   ▼
Tailscale Serve
   │
   │ Local HTTP
   ▼
Application
```

This is useful for applications that require a secure browser context.

---

# 🧭 DNS Configuration

DNS converts hostnames into IP addresses.

Example:

```text
archive.ubuntu.com → IP address
```

A working internet connection can still appear broken when DNS fails.

---

# 🔍 Testing Connectivity

Test direct IP connectivity:

```bash
ping -c 3 1.1.1.1
```

Test hostname resolution:

```bash
ping -c 3 archive.ubuntu.com
```

Check the resolver:

```bash
cat /etc/resolv.conf
```

Check a hostname:

```bash
getent hosts archive.ubuntu.com
```

Interpretation:

| Result                        | Likely Problem                        |
| ----------------------------- | ------------------------------------- |
| IP ping works, hostname fails | DNS issue                             |
| IP ping fails                 | Routing or network issue              |
| Both work                     | Connectivity is operational           |
| Local works, Tailscale fails  | Binding, firewall, or Tailscale issue |

---

# 🧩 Tailscale DNS

When Tailscale DNS is enabled, `/etc/resolv.conf` may contain:

```text
nameserver 100.100.100.100
```

This is the Tailscale MagicDNS resolver.

A Tailscale-managed file may look similar to:

```text
search example-tailnet.ts.net
nameserver 100.100.100.100
```

If Tailscale DNS cannot reach its upstream resolver, external domains may fail.

Example error:

```text
Temporary failure resolving 'archive.ubuntu.com'
```

---

# 🛠️ Disabling Tailscale DNS

To prevent Tailscale from changing DNS on one device:

```bash
tailscale set --accept-dns=false
```

Another valid DNS server must already be configured.

Check again:

```bash
cat /etc/resolv.conf
getent hosts archive.ubuntu.com
```

Do not disable Tailscale DNS when MagicDNS hostnames are required unless another DNS design is in place.

---

# 🌐 Public DNS Resolvers

Common public resolvers include:

| Provider   | Primary | Secondary       |
| ---------- | ------- | --------------- |
| Cloudflare | 1.1.1.1 | 1.0.0.1         |
| Google     | 8.8.8.8 | 8.8.4.4         |
| Quad9      | 9.9.9.9 | 149.112.112.112 |

Using a public resolver does not open access to the server.

The DNS provider can see domain-resolution requests, so the provider should be selected deliberately.

---

# 🛡️ AdGuard Home

AdGuard Home provides DNS filtering for trusted devices.

Desired flow:

```text
Client
   │
   ▼
AdGuard Home
   │
   ▼
Upstream DNS
   │
   ▼
Internet
```

Benefits:

* Network-wide ad blocking
* Tracker filtering
* DNS query visibility
* Custom blocklists
* Local DNS records

---

# 🔍 Testing AdGuard Home

Test from a client:

```bash
nslookup example.com ADGUARD_IP
```

Using `dig`:

```bash
dig @ADGUARD_IP example.com
```

Check whether AdGuard is listening on port 53:

```bash
ss -tulpn | grep ':53'
```

Check its network configuration:

```bash
ip -br address
ip route
```

---

# 🔁 Avoiding DNS Loops

A DNS loop can happen when:

```text
AdGuard Home → Router → AdGuard Home
```

This causes failed or repeated DNS queries.

A clearer design is:

```text
Clients
   │
AdGuard Home
   │
Public Upstream DNS
```

Example upstream servers:

```text
1.1.1.1
1.0.0.1
```

Local DNS requirements may require a different setup.

---

# 📡 TP-Link Deco Guest Network

The Deco guest network isolates guest devices from the main LAN.

This means guest devices may be unable to reach:

* Proxmox
* AdGuard Home
* Vault
* Jellyfin
* Other internal services

Even if AdGuard Home is entered as the DNS server, guest isolation may prevent communication.

---

# ⚠️ Guest Network Limitations

Possible symptoms:

* Guest devices cannot ping the Proxmox host
* Guest devices cannot reach AdGuard Home
* Internal DNS fails
* Main-network devices work correctly
* Tailscale services remain accessible

Possible solutions:

* Use public DNS for guest clients
* Move trusted devices to the main network
* Use a router with VLAN and firewall support
* Create explicit inter-network firewall rules
* Keep the guest network fully isolated

Forwarding public DNS port 53 to AdGuard Home is not recommended.

---

# 🔥 Firewall Checks

Check UFW:

```bash
ufw status
```

Allow access only through Tailscale when needed.

Example SMB access through Tailscale:

```bash
ufw allow in on tailscale0 to any port 445 proto tcp
```

Example Cockpit access:

```bash
ufw allow in on tailscale0 to any port 9090 proto tcp
```

Example Jellyfin access:

```bash
ufw allow in on tailscale0 to any port 8096 proto tcp
```

If UFW is inactive, these rules are stored but not enforced.

---

# 🔎 Service Binding

A service must listen on an accessible interface.

Check listening ports:

```bash
ss -tulpn
```

Check a specific port:

```bash
ss -tulpn | grep 8096
```

Examples:

```text
127.0.0.1:8096
```

Only accessible locally.

```text
0.0.0.0:8096
```

Accessible through all IPv4 interfaces unless blocked by a firewall.

```text
100.x.x.x:8096
```

Accessible through the Tailscale interface.

---

# 🧪 Testing a Service

Test locally:

```bash
curl http://127.0.0.1:PORT
```

Test using the LAN address:

```bash
curl http://LAN_IP:PORT
```

Test using the Tailscale address:

```bash
curl http://TAILSCALE_IP:PORT
```

This helps identify whether the issue is:

* Application failure
* Incorrect port
* Interface binding
* Firewall rule
* Tailscale configuration
* HTTP versus HTTPS mismatch

---

# 🔐 Services Not Exposed Publicly

The following services should not be forwarded directly through the router:

| Service             | Default Port |
| ------------------- | ------------ |
| Proxmox             | 8006         |
| Samba               | 445          |
| Cockpit             | 9090         |
| Portainer           | 9443         |
| Vaultwarden backend | 8000         |
| Sonarr              | 8989         |
| Radarr              | 7878         |
| Prowlarr            | 9696         |

Use Tailscale or another authenticated reverse proxy instead.

---

# 🛠️ Useful Commands

```bash
# Show interfaces
ip -br address

# Show routing
ip route

# Show DNS configuration
cat /etc/resolv.conf

# Test internet connectivity
ping -c 3 1.1.1.1

# Test DNS
getent hosts example.com

# Show listening ports
ss -tulpn

# Check Tailscale
tailscale status

# Show Tailscale IP
tailscale ip -4

# Check Tailscale connectivity
tailscale netcheck

# Restart Tailscale
systemctl restart tailscaled

# Check Tailscale logs
journalctl -u tailscaled -n 100 --no-pager

# Check LXC configuration
pct config CTID

# Check TUN on Proxmox
ls -l /dev/net/tun
```

---

# ⚠️ Important Notes

* Do not expose Proxmox directly to the internet.
* Do not expose Samba ports publicly.
* Use Tailscale for administration.
* Verify whether a service uses HTTP or HTTPS.
* Tailscale DNS can override system DNS.
* LXC containers require TUN passthrough.
* Guest-network isolation can block internal DNS.
* Use fixed addresses or DHCP reservations for infrastructure services.
* Document important ports and hostnames.
* Keep authentication keys outside GitHub.

---

# ✅ Current Status

| Task                       | Status             |
| -------------------------- | ------------------ |
| Proxmox bridge             | ✅ Complete         |
| LAN connectivity           | ✅ Complete         |
| Tailscale on Proxmox       | ✅ Complete         |
| Tailscale on selected LXCs | ✅ Complete         |
| TUN passthrough            | ✅ Complete         |
| MagicDNS                   | ✅ Complete         |
| Vault remote access        | ✅ Complete         |
| Jellyfin remote access     | ✅ Complete         |
| Vaultwarden HTTPS          | ✅ Complete         |
| AdGuard Home               | ✅ Installed        |
| AdGuard Home integration   | 🟡 In Progress     |
| Deco guest-network DNS     | 🟡 Troubleshooting |
| Tailscale ACLs             | ⏳ Planned          |

---

# 🚀 Future Improvements

* Document all static addresses
* Configure Tailscale ACLs
* Add internal DNS records
* Complete AdGuard Home integration
* Improve guest-network separation
* Add VLAN support
* Add network monitoring
* Add automated availability checks
* Document firewall rules
* Create a full network diagram
