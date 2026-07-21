# 🏠 Homelab01-2026

> Self-hosted homelab built with Proxmox VE, ZFS, Docker, Linux containers, Tailscale, and an NVIDIA GPU for learning infrastructure, virtualization, networking, Linux administration, and self-hosting.

---

## 📖 About

Hi! I'm a second-year Computer Science student building this homelab to gain hands-on experience with technologies used in real-world IT environments.

This repository documents my journey from hardware planning and server assembly to deployment, troubleshooting, and continuous improvement.

The goal is to design and maintain a production-inspired environment while learning practical infrastructure and system administration skills.

---

# 🛠️ Tech Stack

| Category            | Technology                       |
| ------------------- | -------------------------------- |
| Hypervisor          | Proxmox VE                       |
| Storage             | Proxmox ZFS                      |
| Containers          | Linux Containers and Docker      |
| Remote Access       | Tailscale                        |
| File Sharing        | Samba and Cockpit                |
| Media               | Jellyfin                         |
| Media Automation    | Sonarr, Radarr, Prowlarr, Seerr  |
| Photo Management    | Immich                           |
| Password Management | Vaultwarden                      |
| DNS Filtering       | AdGuard Home                     |
| VPN Routing         | Gluetun                          |
| Game Servers        | CubeCoders AMP                   |
| Monitoring          | Grafana, Prometheus, Uptime Kuma |
| AI                  | Ollama and Open WebUI            |

---

# ✨ Features

* 🖥️ Proxmox VE virtualization
* 💾 ZFS mirror storage managed directly by Proxmox
* 📂 Samba network file sharing
* 🐳 Docker-based application hosting
* 🎬 Jellyfin media streaming
* ⚡ NVIDIA hardware transcoding
* 📺 Automated media management
* 📸 Immich photo backup
* 🔐 Vaultwarden password management
* 🌐 AdGuard Home DNS filtering
* 📱 Secure remote access with Tailscale
* 🛡️ VPN routing through Gluetun
* 🎮 Game-server management with AMP
* 🤖 Planned private AI platform
* 📊 Planned centralized monitoring

---

# 🎯 Project Goals

This homelab serves two main purposes.

## 📚 Learning Platform

Gain practical experience with:

* Virtualization
* Linux administration
* Docker and containers
* ZFS storage
* Networking
* VPN routing
* Hardware passthrough
* Infrastructure monitoring
* Self-hosting
* System security
* Infrastructure documentation

## 🏠 Home Infrastructure

Provide reliable self-hosted services for household users, including:

* Media streaming
* Photo backup
* Password management
* Network-wide DNS filtering
* File sharing
* Game servers
* Secure remote access
* Future private AI services

The infrastructure is designed with reliability, security, maintainability, scalability, and documentation in mind.

---

# 🖥️ Hardware

| Component      | Model                                   |
| -------------- | --------------------------------------- |
| CPU            | AMD Ryzen 5 5600X                       |
| GPU            | NVIDIA RTX 3060 12 GB                   |
| RAM            | 64 GB DDR4                              |
| Motherboard    | ASUS TUF Gaming B550M-PLUS WiFi II      |
| Boot Drive     | 2 TB NVMe SSD                           |
| Storage        | 2 × 8 TB SATA HDD and 2 × 4 TB SATA HDD |
| Storage Layout | Two ZFS mirror pools                    |
| Case           | JONSBO N6                               |

---

# 🏗️ Current Infrastructure

```text
                           Internet
                               │
                       TP-Link Deco Network
                               │
                        Tailscale Remote Access
                               │
                          Proxmox VE Host
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
   Proxmox ZFS            Application LXCs       AMP Game Servers
        │                      │                      │
        ├── Media              ├── Jellyfin           ├── Minecraft
        ├── Photos             ├── Immich             └── Future Servers
        ├── Downloads          ├── Vaultwarden
        ├── Documents          ├── AdGuard Home
        └── Backups            ├── Vault
                               └── Servarr
                                   ├── Sonarr
                                   ├── Radarr
                                   ├── Prowlarr
                                   ├── qBittorrent
                                   ├── Gluetun
                                   ├── Seerr
                                   └── Portainer
```

---

# 🏗️ Infrastructure Decisions

The homelab uses Proxmox VE as both the virtualization and storage platform.

* Proxmox manages the physical disks directly with ZFS.
* Applications run in separate Linux containers.
* Docker is used for multi-service application stacks.
* ZFS datasets are mounted directly into LXCs.
* The Vault LXC provides Samba file sharing.
* Jellyfin runs in a dedicated LXC with NVIDIA GPU passthrough.
* Download traffic is routed through Gluetun.
* Tailscale provides remote access without exposing administrative ports publicly.
* Game servers run through CubeCoders AMP.

---

# 📦 Services

| Service            | Purpose                         | Status         |
| ------------------ | ------------------------------- | -------------- |
| Proxmox VE         | Hypervisor and storage platform | ✅ Running      |
| ZFS Mirror Pools   | Redundant storage               | ✅ Running      |
| Vault              | File sharing                    | ✅ Running      |
| Samba              | Network shares                  | ✅ Running      |
| Cockpit            | Server management               | ✅ Running      |
| Docker             | Container platform              | ✅ Running      |
| Jellyfin           | Media streaming                 | ✅ Running      |
| NVIDIA Transcoding | Hardware acceleration           | ✅ Running      |
| Sonarr             | TV and anime management         | ✅ Running      |
| Radarr             | Movie management                | ✅ Running      |
| Prowlarr           | Indexer management              | ✅ Running      |
| qBittorrent        | Download client                 | ✅ Running      |
| Gluetun            | VPN gateway                     | ✅ Running      |
| Seerr              | Media requests                  | ✅ Running      |
| Portainer          | Docker management               | ✅ Running      |
| Vaultwarden        | Password manager                | ✅ Running      |
| Immich             | Photo management                | ✅ Running      |
| Tailscale          | Secure remote access            | ✅ Running      |
| AdGuard Home       | DNS filtering                   | 🟡 In Progress |
| AMP                | Game-server management          | ✅ Running      |
| Minecraft          | Game server                     | 🟡 In Progress |
| Grafana            | Monitoring dashboards           | ⏳ Planned      |
| Prometheus         | Metrics collection              | ⏳ Planned      |
| Uptime Kuma        | Service monitoring              | ⏳ Planned      |
| Ollama             | Local AI runtime                | ⏳ Planned      |
| Open WebUI         | AI interface                    | ⏳ Planned      |
| Glance             | Homelab dashboard               | ⏳ Planned      |

---

# 💾 Storage Layout

```text
2 TB NVMe SSD
├── Proxmox VE
├── LXC system disks
├── Application configurations
└── Databases

2 × 8 TB HDD
└── ZFS Mirror
    ├── Media
    ├── Photos
    └── Downloads

2 × 4 TB HDD
└── ZFS Mirror
    ├── Documents
    └── Backups
```

ZFS datasets are mounted directly into LXCs using Proxmox bind mounts.

This provides fast local access without requiring a separate storage virtual machine.

---

# 🎬 Media Platform

The media platform includes:

* Jellyfin
* Sonarr
* Radarr
* Prowlarr
* qBittorrent
* Gluetun
* FlareSolverr
* Seerr
* Profilarr
* Portainer
* Dozzle

Jellyfin uses the NVIDIA RTX 3060 for hardware decoding and encoding.

qBittorrent traffic is routed through Gluetun to prevent direct internet access outside the VPN tunnel.

---

# 🔐 Remote Access and Security

Tailscale provides encrypted remote access to internal services.

No public router port forwarding is required for administrative interfaces.

Security principles include:

* No public Proxmox access
* No public Samba access
* Separate LXCs for important services
* VPN isolation for download traffic
* Hardware and application backups
* No credentials or private keys stored in GitHub
* ZFS redundancy combined with separate backups

---

# 🎮 Game Servers

CubeCoders AMP is used to manage game servers.

Current and planned servers include:

* Minecraft
* Palworld
* Satisfactory
* Project Zomboid
* Valheim

Minecraft is currently being configured.

---

# 🤖 AI Services

The future private AI platform will include:

* Ollama
* Open WebUI
* Local LLM inference
* GPU acceleration
* Multiple user accounts
* Private chat history
* Retrieval-Augmented Generation
* AI agents

---

# 📂 Repository Structure

```text
.
├── docs/
│   ├── 01-proxmox-installation.md
│   ├── 02-proxmox-storage.md
│   ├── 03-networking-and-tailscale.md
│   ├── 04-media-stack.md
│   ├── 05-jellyfin-gpu.md
│   ├── 06-vaultwarden.md
│   ├── 07-immich.md
│   ├── 08-adguard-home.md
│   ├── 09-amp-game-servers.md
│   └── troubleshooting.md
│
├── docker/
├── diagrams/
├── images/
├── scripts/
├── SECURITY.md
├── CHANGELOG.md
└── README.md
```

---

# 📖 Documentation

Each deployment document may include:

* Installation steps
* Configuration
* Network design
* Storage layout
* Issues encountered
* Solutions
* Lessons learned
* Future improvements

---

# 📝 Deployment Roadmap

* [x] Purchase hardware
* [x] Assemble server
* [x] Update motherboard BIOS
* [x] Install Proxmox VE
* [x] Configure networking
* [x] Configure ZFS storage
* [x] Install Tailscale
* [x] Configure Vault and Samba
* [x] Install Docker
* [x] Deploy Jellyfin
* [x] Configure NVIDIA hardware transcoding
* [x] Deploy Servarr stack
* [x] Deploy Vaultwarden
* [x] Deploy Immich
* [x] Install AdGuard Home
* [x] Install AMP
* [ ] Complete AdGuard Home integration
* [ ] Complete Minecraft configuration
* [ ] Deploy Uptime Kuma
* [ ] Deploy Prometheus
* [ ] Deploy Grafana
* [ ] Deploy Glance
* [ ] Deploy Ollama
* [ ] Deploy Open WebUI
* [ ] Configure automated backups
* [ ] Test backup restoration
* [ ] Complete documentation

---

# 🚧 Current Progress

Status: 🟡 Core services deployed, monitoring and automation in progress

| Task                      | Status         |
| ------------------------- | -------------- |
| Hardware Setup            | ✅ Complete     |
| Proxmox VE                | ✅ Complete     |
| ZFS Storage               | ✅ Complete     |
| Tailscale                 | ✅ Complete     |
| Vault File Server         | ✅ Complete     |
| Docker Platform           | ✅ Complete     |
| Media Platform            | ✅ Complete     |
| Jellyfin GPU Acceleration | ✅ Complete     |
| Vaultwarden               | ✅ Complete     |
| Immich                    | ✅ Complete     |
| AMP                       | ✅ Complete     |
| AdGuard Home Integration  | 🟡 In Progress |
| Minecraft Server          | 🟡 In Progress |
| Monitoring Platform       | ⏳ Pending      |
| Private AI Platform       | ⏳ Pending      |
| Automated Backups         | ⏳ Pending      |

---

# 📅 Project Timeline

| Date      | Milestone                       |
| --------- | ------------------------------- |
| July 2026 | Hardware planned and purchased  |
| July 2026 | Server assembled                |
| July 2026 | Proxmox VE installed            |
| July 2026 | ZFS storage configured          |
| July 2026 | Tailscale deployed              |
| July 2026 | Media platform deployed         |
| July 2026 | NVIDIA transcoding configured   |
| July 2026 | Vaultwarden and Immich deployed |
| July 2026 | AMP installed                   |
| Next      | Complete DNS integration        |
| Next      | Deploy monitoring services      |
| Next      | Deploy local AI services        |

---

# 🏛️ Design Principles

* Reliability over complexity
* Direct ZFS storage management
* Separate services by responsibility
* Secure remote access
* Privacy-focused self-hosting
* Centralized monitoring
* Automated backups
* Scalable architecture
* Infrastructure as documentation
* Learn by building

---

# 🚀 Future Roadmap

* Kubernetes
* Ansible
* Terraform
* Infrastructure as Code
* CI/CD pipelines
* Home Assistant
* Advanced monitoring
* GPU virtualization
* Retrieval-Augmented Generation
* AI agents
* Multi-node Proxmox cluster
* High availability

---

# 📷 Screenshots

Coming soon:

* Proxmox dashboard
* ZFS storage
* Jellyfin
* Immich
* Vaultwarden
* Portainer
* AMP
* Grafana dashboards
* Open WebUI

---

# 📌 Notes

This repository is a living project and will continue evolving as I learn new technologies and improve the infrastructure.

Sensitive information, including passwords, credentials, API keys, SSH keys, certificates, VPN keys, environment variables, and licence keys, is excluded from this repository.

---

# 📜 License

This project is licensed under the MIT License.
