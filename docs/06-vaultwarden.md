# 🔐 Vaultwarden

> Self-hosted password management using Vaultwarden, a dedicated Proxmox LXC, and private HTTPS access through Tailscale Serve.

---

# 📖 Overview

Vaultwarden is a lightweight, self-hosted implementation of the Bitwarden server API.

It provides password synchronization for:

* Web browsers
* Desktop applications
* Mobile devices
* Bitwarden-compatible clients

Vaultwarden runs in a dedicated Proxmox LXC and is accessed privately through Tailscale.

The service is not exposed directly to the public internet.

---

# 🏗️ Architecture

```text
Bitwarden Client
       │
       │ HTTPS
       ▼
Tailscale Network
       │
       ▼
Tailscale Serve
       │
       │ Local HTTP
       ▼
Vaultwarden
127.0.0.1:8000
       │
       ▼
Vaultwarden Database
```

Tailscale handles encrypted remote access and HTTPS termination.

Vaultwarden only needs to provide HTTP internally.

---

# 📦 Deployment

Vaultwarden runs in a dedicated Linux container.

Example layout:

```text
Proxmox VE
   │
   └── Vaultwarden LXC
       ├── Vaultwarden binary
       ├── Web vault
       ├── Application data
       ├── systemd service
       └── Tailscale
```

The installation paths in this environment include:

```text
/opt/vaultwarden
/opt/vaultwarden/data
/opt/vaultwarden/.env
```

The exact paths may differ depending on the installation method.

---

# ⚙️ System Requirements

Vaultwarden requires relatively few resources.

Example LXC allocation:

| Resource         | Suggested Allocation |
| ---------------- | -------------------- |
| CPU              | 1 or 2 cores         |
| RAM              | 512 MB to 1 GB       |
| Storage          | 8 GB or more         |
| Network          | LAN and Tailscale    |
| Operating System | Debian or Ubuntu     |

Additional storage may be required when using many attachments.

---

# 🔍 Checking the Service

Check Vaultwarden:

```bash
systemctl status vaultwarden --no-pager
```

Restart it:

```bash
systemctl restart vaultwarden
```

Enable it at boot:

```bash
systemctl enable vaultwarden
```

View recent logs:

```bash
journalctl -u vaultwarden -n 100 --no-pager
```

Follow logs:

```bash
journalctl -u vaultwarden -f
```

---

# 🌐 Network Port

Vaultwarden listens on port:

```text
8000
```

Check the listening socket:

```bash
ss -tlnp | grep 8000
```

Expected example:

```text
LISTEN 0 4096 0.0.0.0:8000
```

Test locally:

```bash
curl http://127.0.0.1:8000
```

A working response should return HTML from the Vaultwarden web vault.

---

# ⚙️ Environment Configuration

The main configuration file is:

```text
/opt/vaultwarden/.env
```

Example configuration:

```env
ROCKET_ADDRESS=0.0.0.0
ROCKET_PORT=8000

DATA_FOLDER=/opt/vaultwarden/data
WEB_VAULT_FOLDER=/opt/vaultwarden/web-vault
WEB_VAULT_ENABLED=true

DOMAIN=https://vaultwarden.REPLACE_ME.ts.net

SIGNUPS_ALLOWED=false
INVITATIONS_ALLOWED=true

ADMIN_TOKEN=REPLACE_ME
```

Sensitive values must not be committed to GitHub.

---

# 🔐 Admin Token

The Vaultwarden admin page is normally available at:

```text
https://vaultwarden.REPLACE_ME.ts.net/admin
```

The `ADMIN_TOKEN` should be protected using an Argon2 PHC string instead of plain text.

Generate a secure value using Vaultwarden:

```bash
/opt/vaultwarden/bin/vaultwarden hash
```

Enter a strong password when requested.

The output will look similar to:

```text
$argon2id$v=19$m=65540,t=3,p=4$...
```

Add the complete value to:

```env
ADMIN_TOKEN='$argon2id$v=19$m=65540,t=3,p=4$REPLACE_ME'
```

Restart Vaultwarden:

```bash
systemctl restart vaultwarden
```

Do not publish the token or its source password.

---

# ⚠️ Empty Admin Token

An empty value may disable the admin page or produce warnings.

Incorrect:

```env
ADMIN_TOKEN=
```

Recommended options:

Use a secure token:

```env
ADMIN_TOKEN='REPLACE_WITH_ARGON2_HASH'
```

Or disable the admin page if it is not required.

The exact option depends on the Vaultwarden version.

---

# 🔒 HTTPS Requirement

The Vaultwarden web vault uses browser cryptography.

The browser requires a secure context for the SubtleCrypto API.

Direct HTTP access may show an error similar to:

```text
You are not using a secure context which is required for the SubtleCrypto API to work
```

This access method is not suitable:

```text
http://TAILSCALE_IP:8000
```

Vaultwarden should be accessed through HTTPS:

```text
https://vaultwarden.REPLACE_ME.ts.net
```

---

# 📱 Tailscale Installation

Install Tailscale inside the Vaultwarden LXC:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Authenticate the device:

```bash
tailscale up
```

Check status:

```bash
tailscale status
```

Show the Tailscale address:

```bash
tailscale ip -4
```

The LXC requires access to `/dev/net/tun`.

---

# 🔌 LXC TUN Configuration

On the Proxmox host, check:

```bash
ls -l /dev/net/tun
```

Stop the Vaultwarden LXC:

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
```

Restart Tailscale:

```bash
systemctl restart tailscaled
tailscale up
```

---

# 🌐 Tailscale Serve

Tailscale Serve provides HTTPS access to the local Vaultwarden service.

Reset any previous configuration:

```bash
tailscale serve reset
```

Create the HTTPS proxy:

```bash
tailscale serve --https=443 http://127.0.0.1:8000
```

Check the configuration:

```bash
tailscale serve status
```

Expected structure:

```text
https://vaultwarden.REPLACE_ME.ts.net
    proxy http://127.0.0.1:8000
```

---

# 🔁 Request Flow

```text
Browser or Bitwarden Client
          │
          │ HTTPS
          ▼
Tailscale Serve
          │
          │ HTTP
          ▼
127.0.0.1:8000
          │
          ▼
Vaultwarden
```

The backend traffic remains local to the Vaultwarden LXC.

---

# ⚠️ ROCKET_TLS Configuration Error

Vaultwarden previously failed after the TLS setting was changed to:

```env
ROCKET_TLS=
```

The logs showed:

```text
invalid type: found string "", expected struct TlsConfig
```

An empty `ROCKET_TLS` variable is invalid.

Incorrect:

```env
ROCKET_TLS=
```

Correct solution:

Remove the entire `ROCKET_TLS` line.

Vaultwarden will then use HTTP internally, while Tailscale Serve provides HTTPS.

Restart the service:

```bash
systemctl restart vaultwarden
```

Check:

```bash
systemctl status vaultwarden --no-pager
```

---

# ⚠️ HTTP and HTTPS Mismatch

Vaultwarden was initially configured with a local TLS certificate.

The backend was therefore listening using HTTPS:

```text
https://0.0.0.0:8000
```

HTTP requests sent to that HTTPS port generated errors such as:

```text
TLS handshake failed
InvalidContentType
ERR_INVALID_HTTP_RESPONSE
```

The request protocols did not match.

Incorrect configuration:

```text
Tailscale Serve HTTP proxy
        │
        ▼
Vaultwarden HTTPS backend
```

Correct configuration:

```text
Tailscale Serve HTTPS
        │
        ▼
Vaultwarden HTTP backend
```

---

# 🌍 Domain Configuration

Set the public application address in `.env`:

```env
DOMAIN=https://vaultwarden.REPLACE_ME.ts.net
```

This allows Vaultwarden to generate correct links and notifications.

Restart after changing it:

```bash
systemctl restart vaultwarden
```

The hostname should match the URL shown by:

```bash
tailscale serve status
```

---

# 👤 User Registration

For a private household installation, public registration should normally be disabled.

Recommended:

```env
SIGNUPS_ALLOWED=false
```

Create the required accounts before disabling registration or use the admin page to invite users.

Optional:

```env
INVITATIONS_ALLOWED=true
```

Restart Vaultwarden after changing the configuration.

---

# 📧 Email Configuration

SMTP allows Vaultwarden to send:

* Invitations
* Email verification
* Emergency-access messages
* Password hints
* Security notifications

Example:

```env
SMTP_HOST=smtp.example.com
SMTP_FROM=vaultwarden@example.com
SMTP_FROM_NAME=Vaultwarden
SMTP_SECURITY=starttls
SMTP_PORT=587
SMTP_USERNAME=REPLACE_ME
SMTP_PASSWORD=REPLACE_ME
```

SMTP credentials must not be committed to GitHub.

Test email delivery before relying on it for account recovery.

---

# 💾 Vaultwarden Data

Important data is normally stored in:

```text
/opt/vaultwarden/data
```

Possible contents include:

```text
db.sqlite3
attachments/
sends/
icon_cache/
rsa_key.pem
rsa_key.pub.pem
```

This folder contains sensitive password-vault information.

It must not be uploaded to GitHub or shared publicly.

---

# 💽 Backup Requirements

A complete backup should include:

```text
Vaultwarden database
Attachments
Sends
RSA keys
Environment configuration
Service configuration
```

Example backup:

```bash
systemctl stop vaultwarden
tar -czf /backup/vaultwarden-data.tar.gz /opt/vaultwarden/data
systemctl start vaultwarden
```

The backup archive contains sensitive information and must be encrypted or stored securely.

---

# 🗃️ SQLite Backup

When Vaultwarden uses SQLite, the database file is normally:

```text
/opt/vaultwarden/data/db.sqlite3
```

A safer live backup can be created using SQLite:

```bash
sqlite3 /opt/vaultwarden/data/db.sqlite3 \
  ".backup '/backup/vaultwarden-db.sqlite3'"
```

This avoids copying the database while it is being modified.

Check database integrity:

```bash
sqlite3 /opt/vaultwarden/data/db.sqlite3 "PRAGMA integrity_check;"
```

Expected result:

```text
ok
```

---

# 🔄 Restore Procedure

Example restore steps:

1. Install the same or compatible Vaultwarden version.
2. Stop the service.
3. Restore the data directory.
4. Restore the `.env` configuration.
5. Correct ownership and permissions.
6. Start Vaultwarden.
7. Test web and client access.

Example:

```bash
systemctl stop vaultwarden
tar -xzf vaultwarden-data.tar.gz -C /
chown -R vaultwarden:vaultwarden /opt/vaultwarden/data
systemctl start vaultwarden
```

The actual service user must be verified before changing ownership.

A backup is not considered reliable until the restore process has been tested.

---

# 🔐 File Permissions

Check ownership:

```bash
ls -la /opt/vaultwarden
ls -la /opt/vaultwarden/data
```

Check the service account:

```bash
systemctl cat vaultwarden
```

The data directory should only be accessible to the service user and administrators.

Example:

```bash
chmod 700 /opt/vaultwarden/data
```

Do not change permissions without checking the installation requirements.

---

# 📱 Bitwarden Clients

Vaultwarden works with official Bitwarden clients.

Supported clients include:

* Browser extension
* Windows desktop application
* Linux desktop application
* Android application
* iOS application
* Web vault

Configure the self-hosted server URL:

```text
https://vaultwarden.REPLACE_ME.ts.net
```

The client device must be connected to the same Tailscale network.

---

# 🔄 Updating Vaultwarden

Before updating:

1. Back up the database.
2. Back up attachments.
3. Back up the `.env` file.
4. Record the installed version.
5. Review release notes.
6. Confirm a restore method.

Check the current version in the logs:

```bash
journalctl -u vaultwarden -n 50 --no-pager
```

Restart after updating:

```bash
systemctl restart vaultwarden
```

Verify:

```bash
systemctl status vaultwarden --no-pager
curl http://127.0.0.1:8000
tailscale serve status
```

---

# 🔍 Troubleshooting Checklist

Check the service:

```bash
systemctl status vaultwarden --no-pager
```

Check logs:

```bash
journalctl -u vaultwarden -n 100 --no-pager
```

Check the backend:

```bash
curl http://127.0.0.1:8000
```

Check the port:

```bash
ss -tlnp | grep 8000
```

Check Tailscale:

```bash
tailscale status
```

Check HTTPS proxy:

```bash
tailscale serve status
```

Check DNS:

```bash
getent hosts vaultwarden.REPLACE_ME.ts.net
```

---

# ⚠️ Common Problems

## Service fails immediately

Check:

```bash
journalctl -u vaultwarden -n 100 --no-pager
```

Possible causes:

* Invalid `.env` syntax
* Empty `ROCKET_TLS`
* Incorrect file permissions
* Database corruption
* Missing data directory
* Invalid admin token format

---

## Local HTTP works but HTTPS does not

Check:

```bash
tailscale serve status
tailscale status
```

Reset and recreate the proxy:

```bash
tailscale serve reset
tailscale serve --https=443 http://127.0.0.1:8000
```

---

## Browser reports insecure context

Cause:

Vaultwarden was opened using HTTP.

Use the HTTPS tailnet URL instead.

---

## Tailscale daemon is unavailable

Check:

```bash
systemctl status tailscaled --no-pager
ls -l /dev/net/tun
```

The LXC may be missing TUN passthrough.

---

## Bitwarden client cannot connect

Check:

* Client is connected to Tailscale
* Server hostname is correct
* HTTPS certificate is valid
* Tailscale Serve is active
* Vaultwarden service is running
* The configured server URL includes `https://`

---

# 🛠️ Useful Commands

```bash
# Check Vaultwarden
systemctl status vaultwarden --no-pager

# Restart Vaultwarden
systemctl restart vaultwarden

# Follow logs
journalctl -u vaultwarden -f

# Test backend
curl http://127.0.0.1:8000

# Check port
ss -tlnp | grep 8000

# Check environment
cat /opt/vaultwarden/.env

# Check Tailscale
tailscale status

# Show Tailscale address
tailscale ip -4

# Check Tailscale Serve
tailscale serve status

# Reset Tailscale Serve
tailscale serve reset

# Configure HTTPS proxy
tailscale serve --https=443 http://127.0.0.1:8000

# Check database integrity
sqlite3 /opt/vaultwarden/data/db.sqlite3 "PRAGMA integrity_check;"
```

Do not display the environment file in screenshots or public logs when it contains secrets.

---

# ⚠️ Important Notes

* Vaultwarden stores highly sensitive information.
* Do not expose the backend port publicly.
* Use HTTPS for all client access.
* Do not commit the Vaultwarden data directory.
* Do not commit the `.env` file.
* Use a strong admin token.
* Disable public registrations.
* Back up the database and attachments.
* Encrypt backup archives.
* Test restoration regularly.
* Keep Tailscale accounts protected with multi-factor authentication.

---

# ✅ Current Status

| Task                             | Status             |
| -------------------------------- | ------------------ |
| Vaultwarden LXC                  | ✅ Complete         |
| Vaultwarden service              | ✅ Running          |
| Local HTTP backend               | ✅ Working          |
| Tailscale installed              | ✅ Complete         |
| LXC TUN passthrough              | ✅ Complete         |
| Tailscale Serve                  | ✅ Configured       |
| Private HTTPS access             | ✅ Working          |
| Bitwarden client synchronization | ✅ Working          |
| Public registration              | ✅ Disabled         |
| Secure admin token               | 🟡 Review Required |
| Automated database backup        | 🟡 In Progress     |
| Restore testing                  | ⏳ Planned          |
| SMTP notifications               | ⏳ Optional         |

---

# 🚀 Future Improvements

* Automate encrypted database backups
* Test a full restore
* Configure SMTP notifications
* Add Vaultwarden health monitoring
* Add Uptime Kuma checks
* Add backup-failure alerts
* Configure Tailscale ACL restrictions
* Add documented account-recovery procedures
* Review admin-page security
* Document update and rollback procedures
