# 🎬 Media Automation Stack

> Docker-based media management using Sonarr, Radarr, Prowlarr, qBittorrent, Gluetun, Seerr, and supporting services.

---

# 📖 Overview

The media platform is divided into two main parts:

* Jellyfin runs in a dedicated LXC with NVIDIA GPU passthrough.
* The Servarr applications run as Docker containers inside a separate LXC.

This separation keeps media playback independent from download automation and Docker management.

---

# 🏗️ Architecture

```text
                         Proxmox VE
                              │
             ┌────────────────┴────────────────┐
             │                                 │
       Jellyfin LXC                      Servarr LXC
             │                                 │
      NVIDIA RTX 3060                   Docker Engine
             │                                 │
      Media Playback                    ├── Sonarr
                                        ├── Radarr
                                        ├── Prowlarr
                                        ├── qBittorrent
                                        ├── Gluetun
                                        ├── FlareSolverr
                                        ├── Seerr
                                        ├── Profilarr
                                        ├── Portainer
                                        └── Dozzle
```

The Jellyfin and Servarr containers access the same Proxmox ZFS datasets through bind mounts.

---

# 📦 Installed Services

| Service      | Purpose                             |
| ------------ | ----------------------------------- |
| Jellyfin     | Media playback and streaming        |
| Sonarr       | TV and anime management             |
| Radarr       | Movie management                    |
| Prowlarr     | Indexer management                  |
| qBittorrent  | Download client                     |
| Gluetun      | VPN gateway and kill switch         |
| FlareSolverr | Browser challenge proxy             |
| Seerr        | Media request management            |
| Profilarr    | Quality profiles and custom formats |
| Portainer    | Docker container management         |
| Dozzle       | Docker log viewer                   |

---

# 📂 Storage Layout

All media-related applications use the same `/data` root.

```text
/data
├── downloads
│   └── qbittorrent
│       ├── completed
│       ├── incomplete
│       └── torrents
├── movies
├── shows
├── anime
├── music
└── photos
```

Using one shared root provides:

* Consistent paths
* Easier imports
* Hardlink support
* Faster moves
* Simpler permissions
* Easier troubleshooting

---

# 🔗 Proxmox Bind Mounts

The ZFS datasets are mounted into the Servarr LXC.

Example:

```bash
pct set CTID -mp0 /pool/media,mp=/data
```

Example LXC configuration:

```ini
mp0: /pool/media,mp=/data
```

Check the mount:

```bash
pct config CTID
```

Inside the LXC:

```bash
ls -la /data
df -h /data
```

The mount must be available before starting Docker.

If the ZFS dataset is unavailable, Docker applications may create local folders and fill the LXC system disk.

---

# 🐳 Docker Configuration Directory

Docker configurations are stored separately from media files.

Example:

```text
/docker
├── servarr
│   ├── compose.yaml
│   └── .env
├── sonarr
├── radarr
├── prowlarr
├── qbittorrent
├── gluetun
├── seerr
├── profilarr
├── portainer
└── dozzle
```

Create the directories:

```bash
mkdir -p /docker/servarr
mkdir -p /docker/{sonarr,radarr,prowlarr,qbittorrent,gluetun,seerr,profilarr,portainer,dozzle}
```

Set ownership:

```bash
chown -R 1000:1000 /docker
```

The correct UID and GID depend on the user running Docker.

---

# ⚙️ Environment File

Example `.env` file:

```env
TZ=Asia/Bangkok

PUID=1000
PGID=1000

CONFIG_PATH=/docker
MEDIA_PATH=/data

LAN_NETWORK=192.168.1.0/24

VPN_SERVICE_PROVIDER=REPLACE_ME
VPN_TYPE=wireguard

WIREGUARD_PRIVATE_KEY=REPLACE_ME
WIREGUARD_PUBLIC_KEY=REPLACE_ME
WIREGUARD_ADDRESSES=REPLACE_ME
```

The `.env` file must not be committed when it contains secrets.

Add it to `.gitignore`:

```text
.env
.env.*
```

---

# 🌐 Docker Network

A dedicated Docker network allows containers to communicate using service names.

Example:

```yaml
networks:
  servarr:
    name: servarr
```

Services attached to the same network can use:

```text
http://sonarr:8989
http://radarr:7878
http://prowlarr:9696
```

Do not use `localhost` to connect to another Docker container.

Inside a container, `localhost` refers to that same container.

---

# 🛡️ Gluetun VPN Gateway

Gluetun provides VPN routing and a kill switch for selected containers.

Recommended use:

```text
Internet
   │
VPN Provider
   │
Gluetun
   │
qBittorrent
```

If the VPN tunnel fails, Gluetun blocks qBittorrent internet access.

---

# 🔌 TUN Device Requirement

Gluetun requires:

```text
/dev/net/tun
```

The Servarr LXC must receive the TUN device from Proxmox.

Add to the LXC configuration:

```ini
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

Verify inside the LXC:

```bash
ls -l /dev/net/tun
```

Expected result:

```text
crw-rw-rw- 1 root root 10, 200 /dev/net/tun
```

---

# 🧩 Example Docker Compose

This is a simplified example. Secrets must remain in the `.env` file.

```yaml
services:
  gluetun:
    image: qmcgaw/gluetun:latest
    container_name: gluetun
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    environment:
      - TZ=${TZ}
      - VPN_SERVICE_PROVIDER=${VPN_SERVICE_PROVIDER}
      - VPN_TYPE=${VPN_TYPE}
      - WIREGUARD_PRIVATE_KEY=${WIREGUARD_PRIVATE_KEY}
      - WIREGUARD_ADDRESSES=${WIREGUARD_ADDRESSES}
    ports:
      - "8080:8080"
      - "8191:8191"
    restart: unless-stopped

  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    network_mode: "service:gluetun"
    depends_on:
      gluetun:
        condition: service_healthy
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=${TZ}
      - WEBUI_PORT=8080
    volumes:
      - ${CONFIG_PATH}/qbittorrent:/config
      - ${MEDIA_PATH}:/data
    restart: unless-stopped

  flaresolverr:
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    network_mode: "service:gluetun"
    depends_on:
      gluetun:
        condition: service_healthy
    environment:
      - LOG_LEVEL=info
      - LOG_HTML=false
      - CAPTCHA_SOLVER=none
      - TZ=${TZ}
    restart: unless-stopped

  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    networks:
      - servarr
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=${TZ}
    volumes:
      - ${CONFIG_PATH}/sonarr:/config
      - ${MEDIA_PATH}:/data
    ports:
      - "8989:8989"
    restart: unless-stopped

  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    networks:
      - servarr
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=${TZ}
    volumes:
      - ${CONFIG_PATH}/radarr:/config
      - ${MEDIA_PATH}:/data
    ports:
      - "7878:7878"
    restart: unless-stopped

  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    networks:
      - servarr
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=${TZ}
    volumes:
      - ${CONFIG_PATH}/prowlarr:/config
    ports:
      - "9696:9696"
    restart: unless-stopped

  seerr:
    image: ghcr.io/seerr-team/seerr:latest
    container_name: seerr
    networks:
      - servarr
    environment:
      - TZ=${TZ}
      - PORT=5055
    volumes:
      - ${CONFIG_PATH}/seerr:/app/config
    ports:
      - "5055:5055"
    restart: unless-stopped

  profilarr:
    image: ghcr.io/dictionarry-hub/profilarr:latest
    container_name: profilarr
    networks:
      - servarr
    environment:
      - TZ=${TZ}
    volumes:
      - ${CONFIG_PATH}/profilarr:/config
    ports:
      - "6868:6868"
    restart: unless-stopped

  dozzle:
    image: amir20/dozzle:latest
    container_name: dozzle
    networks:
      - servarr
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    ports:
      - "8888:8080"
    restart: unless-stopped

networks:
  servarr:
    name: servarr
```

---

# ✅ Validating Docker Compose

Before starting the stack:

```bash
cd /docker/servarr
docker compose config
```

If the configuration is valid:

```bash
docker compose pull
docker compose up -d
```

Check status:

```bash
docker compose ps
```

Check all running containers:

```bash
docker ps
```

---

# 🔍 Checking Container Logs

View logs for one service:

```bash
docker logs sonarr
```

Follow logs:

```bash
docker logs -f sonarr
```

Show the last 100 lines:

```bash
docker logs --tail 100 sonarr
```

Using Docker Compose:

```bash
docker compose logs -f
```

Dozzle can also be used to view logs through a web interface.

---

# 🧪 Testing Gluetun

Test the public IP used by Gluetun:

```bash
docker run --rm \
  --network=container:gluetun \
  alpine:3.20 \
  sh -c "apk add --no-cache wget >/dev/null && wget -qO- https://ipinfo.io"
```

The returned address should be the VPN exit address.

It must not be the home public IP.

Check Gluetun logs:

```bash
docker logs gluetun
```

Look for:

```text
VPN connected
Public IP
Firewall enabled
```

---

# 🧪 Testing qBittorrent VPN Routing

Because qBittorrent shares the Gluetun network namespace, it should use the same public IP.

Check the network mode:

```bash
docker inspect qbittorrent --format '{{.HostConfig.NetworkMode}}'
```

Expected:

```text
container:gluetun
```

or a container ID associated with Gluetun.

If Gluetun stops, qBittorrent should lose internet access.

---

# 📥 qBittorrent Paths

Configure qBittorrent:

```text
Default Save Path:
/data/downloads/qbittorrent/completed

Keep incomplete torrents in:
/data/downloads/qbittorrent/incomplete

Copy torrent files to:
/data/downloads/qbittorrent/torrents
```

These paths must match the paths visible inside Sonarr and Radarr.

---

# 🔗 Sonarr Download Client

In Sonarr:

```text
Settings
→ Download Clients
→ Add qBittorrent
```

When qBittorrent uses Gluetun:

```text
Host: SERVARR_LXC_IP
Port: 8080
Username: REPLACE_ME
Password: REPLACE_ME
```

If Sonarr can reach the Gluetun service name through Docker networking:

```text
Host: gluetun
Port: 8080
```

Test the connection before saving.

---

# 🔗 Radarr Download Client

In Radarr:

```text
Settings
→ Download Clients
→ Add qBittorrent
```

Use the same qBittorrent settings as Sonarr.

Recommended categories:

```text
Sonarr category: tv
Radarr category: movies
```

Categories help separate downloads.

---

# 📺 Sonarr Root Folders

Recommended root folders:

```text
TV:
/data/shows

Anime:
/data/anime
```

Enable file renaming:

```text
Settings
→ Media Management
→ Rename Episodes
```

---

# 🎞️ Radarr Root Folder

Recommended root folder:

```text
/data/movies
```

Enable movie renaming:

```text
Settings
→ Media Management
→ Rename Movies
```

---

# 🔎 Prowlarr Integration

Prowlarr manages indexers and synchronizes them with Sonarr and Radarr.

Connect Sonarr:

```text
Settings
→ Apps
→ Add Sonarr
```

Example:

```text
Prowlarr Server:
http://prowlarr:9696

Sonarr Server:
http://sonarr:8989
```

Connect Radarr:

```text
Radarr Server:
http://radarr:7878
```

Use the API key from each application.

API keys must not be committed to GitHub.

---

# 🧩 FlareSolverr

FlareSolverr helps Prowlarr access supported sites using browser challenges.

Test locally through Gluetun:

```bash
docker exec gluetun wget -qO- http://localhost:8191
```

Expected response:

```json
{"msg":"FlareSolverr is ready!"}
```

Configure it in Prowlarr:

```text
Settings
→ Indexers
→ Indexer Proxies
→ Add FlareSolverr
```

URL when FlareSolverr shares Gluetun networking:

```text
http://SERVARR_LXC_IP:8191
```

or when reachable internally:

```text
http://gluetun:8191
```

---

# ⚠️ FlareSolverr Limitations

FlareSolverr may still fail when:

* The VPN address has a poor reputation
* A site changes its browser challenge
* A CAPTCHA is required
* Chromium times out
* Shared memory is insufficient
* The target site blocks automated browsers

A timeout does not automatically mean the Docker Compose configuration is incorrect.

---

# 📺 Seerr

Seerr allows household users to request movies and series.

Connect it to:

* Jellyfin
* Sonarr
* Radarr

Example service addresses:

```text
Sonarr:
http://sonarr:8989

Radarr:
http://radarr:7878

Jellyfin:
http://JELLYFIN_LXC_IP:8096
```

Configure:

* Default quality profiles
* Root folders
* Anime profiles
* Tags
* Request permissions

---

# 🏷️ Anime Configuration

Anime releases often use filenames such as:

```text
Series Title - 05.mkv
```

instead of:

```text
Series.Title.S03E05.mkv
```

Recommended Sonarr configuration:

```text
Series Type: Anime
Root Folder: /data/anime
Quality Profile: Anime 1080p
Tag: anime
```

If an episode cannot be matched:

1. Open Sonarr.
2. Go to Activity.
3. Select Manual Import.
4. Choose the correct file.
5. Assign the correct season and episode.
6. Import it.

---

# 🧾 Profilarr

Profilarr manages quality profiles and custom formats.

Connect it to Sonarr and Radarr using their API keys.

Example addresses:

```text
http://sonarr:8989
http://radarr:7878
```

Profilarr does not normally need to use Gluetun.

It should remain on the standard Docker network unless VPN routing is specifically required.

---

# 🖥️ Portainer

Portainer provides a web interface for Docker management.

Typical deployment:

```bash
docker volume create portainer_data
```

```bash
docker run -d \
  -p 8000:8000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:lts
```

Access through Tailscale:

```text
https://SERVARR_TAILSCALE_IP:9443
```

Do not expose Portainer directly to the public internet.

---

# 🔐 Permissions

LinuxServer containers use `PUID` and `PGID`.

Check the user:

```bash
id
```

Example:

```text
uid=1000
gid=1000
```

Set:

```env
PUID=1000
PGID=1000
```

Check ownership:

```bash
ls -ln /data
ls -ln /docker
```

Incorrect permissions can cause:

* Failed imports
* Unable to rename files
* Unable to create directories
* Download failures
* Configuration database errors

---

# 🔗 Hardlinks

Hardlinks allow Sonarr and Radarr to import files without creating a second full copy.

Hardlinks require:

* Source and destination on the same filesystem
* Consistent container paths
* Correct permissions

Example:

```text
/data/downloads/qbittorrent/completed
/data/movies
/data/shows
```

Avoid separate mounts such as:

```text
/downloads
/movies
/tv
```

when they refer to the same storage.

Using `/data` as one shared root is simpler.

---

# 🔍 Checking Hardlinks

Check inode numbers:

```bash
ls -li /data/downloads/qbittorrent/completed/FILE
ls -li /data/movies/MOVIE/FILE
```

If the inode number is the same, the files are hardlinked.

---

# 🔄 Updating Containers

Pull new images:

```bash
docker compose pull
```

Recreate containers:

```bash
docker compose up -d
```

Remove unused images:

```bash
docker image prune
```

Check release notes before major application upgrades.

Back up configurations before updating.

---

# 💾 Backup Requirements

Back up:

```text
/docker/sonarr
/docker/radarr
/docker/prowlarr
/docker/qbittorrent
/docker/seerr
/docker/profilarr
Portainer data volume
compose.yaml
.env
```

The `.env` backup must remain private.

Media files and application configurations should not be treated as the same backup set.

---

# 🛠️ Useful Commands

```bash
# Validate Compose
docker compose config

# Start stack
docker compose up -d

# Stop stack
docker compose down

# Show services
docker compose ps

# Pull images
docker compose pull

# Follow logs
docker compose logs -f

# Check one container
docker logs --tail 100 CONTAINER

# Inspect network mode
docker inspect CONTAINER --format '{{.HostConfig.NetworkMode}}'

# Show Docker networks
docker network ls

# Show disk usage
docker system df

# Show mounted data
df -h /data

# Check permissions
ls -ln /data
```

---

# ⚠️ Important Notes

* Keep Docker configuration files on fast local storage.
* Keep media and downloads on ZFS datasets.
* Use one shared `/data` root.
* Never commit VPN keys.
* Verify Gluetun before starting qBittorrent.
* Do not expose Portainer publicly.
* Back up application databases before upgrades.
* Confirm permissions before recursive ownership changes.
* Check that ZFS datasets are mounted before Docker starts.
* Test hardlinks after configuring Sonarr and Radarr.

---

# ✅ Current Status

| Task                            | Status         |
| ------------------------------- | -------------- |
| Servarr LXC                     | ✅ Complete     |
| Docker Engine                   | ✅ Complete     |
| ZFS bind mount                  | ✅ Complete     |
| Sonarr                          | ✅ Running      |
| Radarr                          | ✅ Running      |
| Prowlarr                        | ✅ Running      |
| qBittorrent                     | ✅ Running      |
| Gluetun                         | ✅ Running      |
| VPN kill switch                 | ✅ Working      |
| FlareSolverr                    | ✅ Running      |
| Seerr                           | ✅ Running      |
| Profilarr                       | ✅ Running      |
| Portainer                       | ✅ Running      |
| Dozzle                          | ✅ Running      |
| Anime profiles                  | ✅ Configured   |
| Automated configuration backups | 🟡 In Progress |
| Monitoring                      | ⏳ Planned      |

---

# 🚀 Future Improvements

* Add automated Docker configuration backups
* Add Uptime Kuma monitoring
* Add Prometheus container metrics
* Add Grafana dashboards
* Add Docker health checks
* Add notification services
* Add automatic mount verification
* Document all quality profiles
* Document indexer configuration
* Test complete recovery of the Servarr stack
