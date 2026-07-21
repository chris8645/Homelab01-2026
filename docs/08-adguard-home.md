# 🌐 AdGuard Home DNS Filtering

> Network-wide DNS filtering using AdGuard Home in a dedicated Proxmox LXC.

---

# 📖 Overview

AdGuard Home provides DNS filtering for devices connected to the home network.

It is used to:

* Block advertising domains
* Block tracking domains
* Improve DNS visibility
* Create local DNS records
* Apply filtering rules to multiple devices
* Monitor DNS queries
* Control upstream DNS providers

AdGuard Home runs inside a dedicated Proxmox LXC.

---

# 🏗️ Architecture

```text
Client Device
      │
      │ DNS Request
      ▼
AdGuard Home LXC
      │
      │ Filtered DNS Request
      ▼
Upstream DNS Resolver
      │
      ▼
Internet
```

The intended network flow is:

```text
TP-Link Deco DHCP
        │
        ▼
Clients receive AdGuard IP
        │
        ▼
AdGuard Home
        │
        ▼
Cloudflare, Quad9, or another upstream resolver
```

---

# 🖥️ Suggested LXC Resources

| Resource         | Suggested Allocation |
| ---------------- | -------------------- |
| CPU              | 1 or 2 cores         |
| RAM              | 512 MB to 1 GB       |
| Storage          | 8 GB or more         |
| Network          | Static LAN address   |
| Operating System | Debian or Ubuntu     |

AdGuard Home uses relatively few resources for a normal household network.

---

# 📍 Static IP Address

The AdGuard Home LXC should use a stable IP address.

Recommended options:

* Static address configured in Proxmox
* Static address configured inside the LXC
* DHCP reservation configured in the router

Example Proxmox LXC network configuration:

```ini
net0: name=eth0,bridge=vmbr0,ip=192.168.1.53/24,gw=192.168.1.1,type=veth
```

The exact address must match the home network.

Do not allow the AdGuard IP to change after clients start using it as their DNS server.

---

# 📦 Installation

Download and install AdGuard Home using the official installer.

Example:

```bash
curl -s -S -L \
  https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh \
  | sh -s -- -v
```

Check the service:

```bash
systemctl status AdGuardHome --no-pager
```

Start it:

```bash
systemctl start AdGuardHome
```

Enable it at boot:

```bash
systemctl enable AdGuardHome
```

---

# 🌐 Initial Configuration

The initial setup interface normally uses:

```text
http://ADGUARD_IP:3000
```

During setup, configure:

```text
Web interface:
0.0.0.0:3000

DNS server:
0.0.0.0:53
```

After setup, the administration interface normally uses port:

```text
3000
```

The exact port can be changed later.

---

# 🔍 Checking Listening Ports

Check whether AdGuard Home is listening:

```bash
ss -tulpn | grep AdGuardHome
```

Check DNS port 53:

```bash
ss -tulpn | grep ':53'
```

Expected listeners may include:

```text
0.0.0.0:53
[::]:53
```

Check the web interface:

```bash
ss -tulpn | grep ':3000'
```

---

# 🧭 DNS Configuration

DNS converts domain names into IP addresses.

Example:

```text
github.com → IP address
```

Clients send DNS requests to AdGuard Home.

AdGuard checks the requested domain against its rules.

If the domain is allowed, AdGuard forwards the query to an upstream resolver.

If the domain is blocked, AdGuard returns a blocked response.

---

# 🌍 Upstream DNS Servers

Example upstream resolvers:

```text
https://dns.cloudflare.com/dns-query
https://dns.quad9.net/dns-query
```

Standard DNS examples:

```text
1.1.1.1
1.0.0.1
9.9.9.9
149.112.112.112
```

Possible configuration:

```text
Primary:
https://dns.cloudflare.com/dns-query

Secondary:
https://dns.quad9.net/dns-query
```

Encrypted upstream DNS can improve privacy between AdGuard Home and the upstream provider.

---

# 🔁 Bootstrap DNS

Bootstrap DNS is used to resolve the hostname of encrypted upstream DNS servers.

Example:

```text
1.1.1.1
9.9.9.9
```

If using:

```text
https://dns.cloudflare.com/dns-query
```

AdGuard must first resolve:

```text
dns.cloudflare.com
```

Bootstrap DNS provides this initial resolution.

---

# ⚠️ Avoiding DNS Loops

A DNS loop can happen when:

```text
AdGuard Home
      │
      ▼
Home Router
      │
      ▼
AdGuard Home
```

This causes repeated or failed queries.

Avoid configuring the router as the AdGuard upstream when the router already sends DNS back to AdGuard.

Recommended flow:

```text
Clients
   │
AdGuard Home
   │
Public Upstream DNS
```

---

# 🏠 DHCP Configuration

The router should provide the AdGuard Home IP as the DNS server.

Example:

```text
Primary DNS:
ADGUARD_IP
```

Possible secondary DNS choices:

```text
None
```

or another internal AdGuard instance.

Using a public secondary resolver allows clients to bypass AdGuard Home.

For example:

```text
Primary DNS:
ADGUARD_IP

Secondary DNS:
1.1.1.1
```

This does not provide strict filtering because clients may choose the secondary server directly.

---

# 📡 TP-Link Deco Configuration

In the Deco application, DNS settings may be available under:

```text
More
→ Advanced
→ DHCP Server
```

Set the primary DNS to the AdGuard Home IP.

Example:

```text
Primary DNS:
192.168.1.53
```

The exact menus depend on the Deco model and firmware.

After changing DNS settings:

1. Save the configuration.
2. Disconnect and reconnect client devices.
3. Renew their DHCP leases.
4. Verify the DNS server received by the client.

---

# 🧪 Testing from Linux

Check the current DNS configuration:

```bash
cat /etc/resolv.conf
```

Test AdGuard directly:

```bash
nslookup example.com ADGUARD_IP
```

Using `dig`:

```bash
dig @ADGUARD_IP example.com
```

Test a blocked domain:

```bash
nslookup BLOCKED_DOMAIN ADGUARD_IP
```

The query should also appear in the AdGuard Home query log.

---

# 🧪 Testing from Windows

Open Command Prompt:

```cmd
ipconfig /all
```

Check the DNS server entry.

Test AdGuard:

```cmd
nslookup example.com ADGUARD_IP
```

Display the current DNS server:

```cmd
nslookup
```

Clear the Windows DNS cache:

```cmd
ipconfig /flushdns
```

Renew the DHCP lease:

```cmd
ipconfig /release
ipconfig /renew
```

---

# 📊 Query Log

The AdGuard Home query log shows:

* Client address
* Requested domain
* Response status
* Upstream resolver
* Processing time
* Filtering rule
* Blocked or allowed result

Use the query log to verify that clients are using AdGuard Home.

If no queries appear, the client is probably using another resolver.

---

# 🛡️ Filter Lists

AdGuard Home supports multiple blocklists.

Recommended approach:

* Start with the default AdGuard list
* Add lists gradually
* Avoid enabling too many overlapping lists
* Review false positives
* Use allowlists for required services

Too many lists can cause:

* False positives
* Slower updates
* Difficult troubleshooting
* Broken websites or applications

---

# ✅ Allowlist

Add a domain to the allowlist when a required service is blocked.

Example:

```text
@@||example.com^
```

Before allowing a domain:

1. Check the AdGuard query log.
2. Confirm which rule blocked it.
3. Allow only the required domain.
4. Retest the application.

Avoid disabling all filtering to fix one website.

---

# 🚫 Custom Blocking Rules

Example domain block:

```text
||tracking.example.com^
```

Example exact domain block:

```text
|https://ads.example.com|
```

Use custom rules carefully.

Document important custom rules so they can be recreated after recovery.

---

# 🏷️ Local DNS Records

AdGuard Home can create local DNS names for homelab services.

Examples:

```text
proxmox.home
vault.home
jellyfin.home
servarr.home
immich.home
vaultwarden.home
amp.home
```

Example mapping:

```text
proxmox.home → PROXMOX_IP
jellyfin.home → JELLYFIN_IP
```

This makes services easier to access on the local network.

---

# 📱 Tailscale and AdGuard Home

Tailscale can use AdGuard Home as a DNS resolver.

Possible flow:

```text
Remote Tailscale Device
          │
          ▼
Tailscale DNS Configuration
          │
          ▼
AdGuard Home
          │
          ▼
Upstream DNS
```

Before enabling this for the entire tailnet, verify that AdGuard Home remains reachable remotely.

If AdGuard is unavailable, Tailscale clients may lose DNS resolution.

---

# 🔐 Installing Tailscale

Install inside the AdGuard LXC:

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

The LXC requires access to `/dev/net/tun`.

---

# 🔌 LXC TUN Passthrough

On the Proxmox host:

```bash
ls -l /dev/net/tun
```

Stop the AdGuard LXC:

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

# ⚠️ Tailscale DNS Interaction

Tailscale can modify `/etc/resolv.conf`.

Example:

```text
nameserver 100.100.100.100
```

This is the Tailscale MagicDNS resolver.

If the AdGuard LXC accepts Tailscale DNS while also providing DNS services, the resolver design must be checked carefully.

Possible problems include:

* DNS loops
* External resolution failure
* AdGuard using itself as upstream
* Tailscale DNS depending on AdGuard
* AdGuard depending on Tailscale DNS

Use explicit upstream resolvers inside AdGuard Home.

---

# 🛠️ Disabling Tailscale DNS Locally

If the AdGuard LXC should not use Tailscale DNS:

```bash
tailscale set --accept-dns=false
```

Then verify:

```bash
cat /etc/resolv.conf
getent hosts example.com
```

A separate working resolver must be configured for the LXC itself.

---

# 📡 Deco Guest Network

The TP-Link Deco guest network isolates guest devices from the primary LAN.

This can prevent guest devices from reaching AdGuard Home.

Possible symptoms:

* Guest client receives the AdGuard IP
* DNS requests time out
* Guest client cannot ping AdGuard
* Main network clients work normally
* Proxmox services are inaccessible from the guest network

This is expected behavior when guest isolation is active.

---

# ⚠️ Guest Network DNS Limitations

The guest network may not support a custom DNS server located on the main LAN.

Possible options:

* Use public DNS for guest devices
* Use the main network for trusted household devices
* Use a router with VLAN support
* Create explicit firewall rules on VLAN-capable equipment
* Run a separate resolver reachable from the guest network

Port forwarding DNS port 53 is not a recommended solution.

Do not expose the internal DNS server directly to the internet.

---

# 🔥 Firewall

Check UFW:

```bash
ufw status
```

Allow DNS from the LAN:

```bash
ufw allow from LAN_SUBNET to any port 53 proto udp
ufw allow from LAN_SUBNET to any port 53 proto tcp
```

Allow the web interface from the LAN:

```bash
ufw allow from LAN_SUBNET to any port 3000 proto tcp
```

Allow DNS through Tailscale:

```bash
ufw allow in on tailscale0 to any port 53
```

If UFW is inactive, the rules are not enforced.

---

# 🔍 Connectivity Checks

Check the interface:

```bash
ip -br address
```

Check routing:

```bash
ip route
```

Test the router:

```bash
ping -c 3 ROUTER_IP
```

Test internet connectivity:

```bash
ping -c 3 1.1.1.1
```

Test DNS:

```bash
getent hosts example.com
```

Check neighbors:

```bash
ip neigh
```

---

# ⚠️ Destination Host Unreachable

Example:

```text
Destination Host Unreachable
```

Possible causes:

* Incorrect subnet
* Incorrect gateway
* Duplicate IP address
* Deco access-point or router mode issue
* LXC bridge problem
* Network isolation
* Physical network problem
* Incorrect VLAN

Check:

```bash
ip -br address
ip route
ping -c 3 ROUTER_IP
ip neigh
```

---

# 🔄 Service Management

Check AdGuard Home:

```bash
systemctl status AdGuardHome --no-pager
```

Restart:

```bash
systemctl restart AdGuardHome
```

Enable at boot:

```bash
systemctl enable AdGuardHome
```

View logs:

```bash
journalctl -u AdGuardHome -n 100 --no-pager
```

Follow logs:

```bash
journalctl -u AdGuardHome -f
```

---

# 📂 Configuration Files

The AdGuard Home installation directory may contain:

```text
AdGuardHome
AdGuardHome.yaml
data/
```

Find the service configuration:

```bash
systemctl cat AdGuardHome
```

Search for the configuration file:

```bash
find / -name "AdGuardHome.yaml" 2>/dev/null
```

The YAML file contains important DNS and authentication configuration.

Do not publish it without reviewing sensitive fields.

---

# 💾 Backup

Back up:

```text
AdGuardHome.yaml
Custom filtering rules
Allowlist
Blocklist configuration
Local DNS records
Client configuration
DHCP configuration if used
```

Example:

```bash
systemctl stop AdGuardHome
tar -czf /backup/adguard-home.tar.gz /opt/AdGuardHome
systemctl start AdGuardHome
```

The actual installation directory must be confirmed.

---

# 🔄 Restore

General restore process:

1. Install the same or compatible AdGuard Home version.
2. Stop the service.
3. Restore the configuration directory.
4. Correct ownership and permissions.
5. Start the service.
6. Test DNS queries.
7. Verify clients and filtering rules.

A DNS backup is important because losing the resolver configuration can affect the entire network.

---

# 📊 Monitoring

Monitor:

* DNS service availability
* Port 53
* Web administration interface
* Query response time
* Upstream resolver failures
* Number of blocked requests
* LXC resource usage
* Storage usage

Uptime Kuma can test:

```text
ADGUARD_IP:53
```

and:

```text
http://ADGUARD_IP:3000
```

---

# 🔍 Troubleshooting Checklist

Check the service:

```bash
systemctl status AdGuardHome --no-pager
```

Check DNS port:

```bash
ss -tulpn | grep ':53'
```

Test locally:

```bash
nslookup example.com 127.0.0.1
```

Test using the LXC address:

```bash
nslookup example.com ADGUARD_IP
```

Check routing:

```bash
ip route
```

Check DNS configuration:

```bash
cat /etc/resolv.conf
```

Check Tailscale:

```bash
tailscale status
```

Review logs:

```bash
journalctl -u AdGuardHome -n 100 --no-pager
```

---

# ⚠️ Common Problems

## Client does not appear in query log

Possible causes:

* Client is using another DNS server
* Browser is using secure DNS
* Android Private DNS is enabled
* Apple Private Relay is enabled
* VPN application provides its own DNS
* Router still provides public DNS
* DHCP lease has not renewed

Check the client's DNS configuration.

---

## AdGuard resolves locally but clients fail

Possible causes:

* Firewall blocking port 53
* Incorrect DHCP DNS setting
* Guest-network isolation
* AdGuard listening only on localhost
* Incorrect client subnet
* Duplicate IP address

Check:

```bash
ss -tulpn | grep ':53'
ufw status
```

---

## Internet works by IP but not by hostname

Example:

```bash
ping -c 3 1.1.1.1
```

works, but:

```bash
ping -c 3 example.com
```

fails.

This indicates a DNS problem.

Check:

```bash
cat /etc/resolv.conf
nslookup example.com ADGUARD_IP
```

---

## AdGuard Home is unavailable

Clients may appear to lose internet access because DNS requests fail.

Recovery:

1. Check the LXC.
2. Restart AdGuard Home.
3. Verify port 53.
4. Temporarily configure a public DNS server on the affected client.
5. Repair the AdGuard service.
6. Restore the normal DNS configuration.

---

## Some websites break

Review the query log.

Look for blocked domains associated with the website.

Add only the required domains to the allowlist.

Avoid disabling all filters.

---

# 🛠️ Useful Commands

```bash
# Check AdGuard Home
systemctl status AdGuardHome --no-pager

# Restart AdGuard Home
systemctl restart AdGuardHome

# View logs
journalctl -u AdGuardHome -n 100 --no-pager

# Check DNS port
ss -tulpn | grep ':53'

# Check web interface
ss -tulpn | grep ':3000'

# Test DNS locally
nslookup example.com 127.0.0.1

# Test DNS remotely
nslookup example.com ADGUARD_IP

# Check network
ip -br address
ip route

# Check resolver
cat /etc/resolv.conf

# Check Tailscale
tailscale status

# Show Tailscale IP
tailscale ip -4

# Check LXC configuration
pct config CTID
```

---

# ⚠️ Important Notes

* Use a static IP for AdGuard Home.
* Avoid DNS loops.
* Do not expose port 53 publicly.
* Do not use public DNS as a secondary if strict filtering is required.
* Review browser secure-DNS settings.
* Guest-network isolation may block AdGuard.
* Back up the configuration.
* Monitor service availability.
* Keep upstream DNS settings explicit.
* Test recovery before making AdGuard the only network resolver.

---

# ✅ Current Status

| Task                           | Status             |
| ------------------------------ | ------------------ |
| AdGuard Home LXC               | ✅ Complete         |
| AdGuard Home service           | ✅ Running          |
| Static IP                      | ✅ Configured       |
| Web interface                  | ✅ Working          |
| DNS filtering                  | ✅ Working          |
| Upstream DNS                   | ✅ Configured       |
| Local-network clients          | ✅ Working          |
| Tailscale installed            | ✅ Complete         |
| Remote DNS access              | 🟡 Testing         |
| Deco DHCP integration          | 🟡 In Progress     |
| Guest-network DNS              | 🟡 Troubleshooting |
| Local DNS records              | ⏳ Planned          |
| Automated configuration backup | ⏳ Planned          |
| Monitoring                     | ⏳ Planned          |

---

# 🚀 Future Improvements

* Complete Deco DHCP integration
* Add local DNS records
* Configure Tailscale DNS
* Add Uptime Kuma monitoring
* Add resolver-failure alerts
* Automate configuration backups
* Review blocklists
* Document client-specific DNS behavior
* Improve guest-network design
* Add VLAN-aware routing
