# 🏠 Homelab01-2026

> Self-hosted homelab built with Proxmox, TrueNAS SCALE, Docker, and AI services for learning infrastructure, virtualization, networking, Linux administration, and self-hosting.

---

## 📖 About

Hi! I'm a second-year Computer Science student building this homelab to gain hands-on experience with technologies used in real-world IT environments.

This repository documents my journey from planning and hardware selection to deployment, troubleshooting, and continuous improvements.

The goal of this project is to gain practical experience by designing, deploying, and maintaining a production-inspired homelab while documenting everything I learn.

---

# 🛠️ Tech Stack

| Category | Technology |
|----------|------------|
| Hypervisor | Proxmox VE |
| Storage | TrueNAS SCALE + ZFS |
| Containers | Docker + Dockge |
| Dashboard | Glance |
| Monitoring | Grafana + Prometheus + Uptime Kuma |
| Remote Access | Tailscale |
| Password Management | Vaultwarden |
| Media | Jellyfin |
| Photo Management | Immich |
| AI | Ollama + Open WebUI |
| Game Servers | Dedicated VM |

---

# ✨ Features

- 🖥️ Proxmox VE virtualization
- 💾 TrueNAS SCALE with ZFS Mirror storage
- 🐳 Docker-based application hosting
- 📸 Immich photo management
- 🎬 Jellyfin media streaming
- 🤖 Private multi-user AI (Open WebUI + Ollama)
- 🔐 Vaultwarden password management
- 🌐 Pi-hole / AdGuard Home DNS filtering
- 📊 Grafana & Prometheus monitoring
- ❤️ Uptime Kuma service monitoring
- 🏠 Glance dashboard
- ⚙️ Dockge Docker Compose management
- 📱 Secure remote access with Tailscale
- 🎮 Dedicated game server hosting

---

# 🎯 Project Goals

This homelab serves two primary purposes.

## 📚 Learning Platform

Gain practical experience with:

- Virtualization
- Linux administration
- Docker & containerization
- Storage systems (ZFS)
- Networking
- Infrastructure monitoring
- Self-hosting
- AI deployment
- Infrastructure documentation
- System administration

## 🏠 Home Infrastructure

Provide reliable self-hosted services for multiple household users including:

- Media streaming
- Photo backup
- Password management
- Network-wide ad blocking
- Private AI assistants
- Game servers
- Remote access

The infrastructure is designed with **reliability, security, scalability, maintainability, and documentation** in mind.

---

# 🖥️ Hardware

| Component | Model |
|-----------|-------|
| CPU | AMD Ryzen 5 5600X |
| GPU | NVIDIA RTX 3060 12GB |
| RAM | 64GB DDR4 |
| Motherboard | ASUS TUF Gaming B550M-PLUS WiFi II |
| Boot Drive | 2TB NVMe SSD |
| Storage | 2 × 8TB SATA HDD + 2 × 4TB SATA HDD |
| Storage Layout | Two ZFS Mirror Pools *(Planned)* |
| Case | JONSBO N6 |

---

# 🏗️ Planned Infrastructure

```text
                           Internet
                               │
                           Home Router
                               │
                        Tailscale VPN
                               │
                          Proxmox VE Host
                               │
        ┌──────────────────────┼────────────────────────┐
        │                      │                        │
        │                      │                        │
   TrueNAS SCALE          Docker VM              Game Server VM
        │                      │                        │
 ZFS Mirror Pools        ├── Glance             ├── Minecraft
        │                ├── Dockge             ├── Palworld
        ├── Media        ├── Open WebUI         ├── Satisfactory
        ├── Photos       ├── Ollama             ├── Project Zomboid
        ├── Documents    ├── Jellyfin           └── More...
        ├── Backups      ├── Immich
        └── ISOs         ├── Vaultwarden
                          ├── Pi-hole / AdGuard
                          ├── Grafana
                          ├── Prometheus
                          └── Uptime Kuma
```

---

# 🏗️ Infrastructure Decisions

The homelab separates storage and compute into dedicated virtual machines.

- Proxmox VE manages virtualization.
- TrueNAS SCALE provides centralized ZFS storage.
- Docker hosts all self-contained applications.
- Game servers run in their own VM.
- Docker applications and databases run on the NVMe SSD.
- Media, photos, documents, and backups are stored on mirrored ZFS pools for redundancy and easier management.

---

# 📦 Planned Services

| Service | Purpose | Status |
|----------|---------|--------|
| Proxmox VE | Hypervisor | ✅ Installed |
| TrueNAS SCALE | Storage Server | ⏳ Planned |
| Docker | Container Platform | ⏳ Planned |
| Dockge | Docker Compose Management | ⏳ Planned |
| Glance | Dashboard | ⏳ Planned |
| Immich | Photo Management | ⏳ Planned |
| Jellyfin | Media Streaming | ⏳ Planned |
| Vaultwarden | Password Manager | ⏳ Planned |
| Pi-hole / AdGuard Home | DNS Filtering | ⏳ Planned |
| Grafana | Monitoring | ⏳ Planned |
| Prometheus | Metrics | ⏳ Planned |
| Uptime Kuma | Service Monitoring | ⏳ Planned |
| Ollama | Local LLM Runtime | ⏳ Planned |
| Open WebUI | AI Interface | ⏳ Planned |
| Tailscale | Remote Access | ⏳ Planned |
| Game Servers | Multiplayer Hosting | ⏳ Planned |

---

# 🤖 AI Services

The homelab will host a private AI platform for household users.

Planned features include:

- Multi-user accounts
- Private chat history
- Local LLM inference
- Multiple selectable models
- GPU acceleration
- Future Retrieval-Augmented Generation (RAG)
- AI Agents

Software:

- Ollama
- Open WebUI

---

# 🎮 Planned Game Servers

- Minecraft
- Palworld
- Satisfactory
- Project Zomboid
- Valheim *(Future)*

---

# 📂 Repository Structure

```text
.
├── docs/
│   ├── 01-proxmox-installation.md
│   ├── 02-truenas.md
│   ├── 03-docker.md
│   ├── 04-networking.md
│   ├── 05-storage.md
│   ├── 06-monitoring.md
│   ├── 07-ai.md
│   ├── 08-game-servers.md
│   └── troubleshooting.md
│
├── docker/
├── diagrams/
├── images/
├── scripts/
└── README.md
```

---

# 📖 Documentation

Each deployment includes:

- Installation steps
- Configuration
- Network design
- Storage layout
- Issues encountered
- Solutions
- Lessons learned
- Future improvements

---

# 📝 Deployment Roadmap

- [x] Purchase hardware
- [x] Assemble server
- [x] Update motherboard BIOS
- [x] Install Proxmox VE
- [x] Configure networking
- [x] Configure no-subscription repository
- [ ] Deploy TrueNAS SCALE
- [ ] Configure ZFS Mirror Pools
- [ ] Create datasets
- [ ] Deploy Docker VM
- [ ] Install Docker Engine
- [ ] Install Dockge
- [ ] Deploy Glance
- [ ] Deploy Jellyfin
- [ ] Deploy Immich
- [ ] Deploy Vaultwarden
- [ ] Deploy Pi-hole / AdGuard Home
- [ ] Deploy Prometheus
- [ ] Deploy Grafana
- [ ] Deploy Uptime Kuma
- [ ] Deploy Ollama
- [ ] Deploy Open WebUI
- [ ] Configure Tailscale
- [ ] Deploy Game Servers
- [ ] Configure automated backups
- [ ] Complete documentation

---

# 🚧 Current Progress

**Status:** 🟡 Building

| Task | Status |
|------|--------|
| Hardware Planning | ✅ Complete |
| Hardware Purchased | ✅ Complete |
| Server Assembly | ✅ Complete |
| BIOS Updated | ✅ Complete |
| Proxmox Installed | ✅ Complete |
| Networking Configured | ✅ Complete |
| TrueNAS Deployment | ⏳ Pending |
| Docker Deployment | ⏳ Pending |

---

# 📅 Project Timeline

| Date | Milestone |
|------|-----------|
| July 2026 | Hardware planned |
| July 2026 | Hardware purchased |
| July 2026 | Server assembled |
| July 2026 | BIOS updated |
| July 2026 | Proxmox VE installed |
| Next | Deploy TrueNAS SCALE |
| Next | Deploy Docker VM |

---

# 🏛️ Design Principles

- Reliability over complexity
- Separate compute and storage
- Secure remote access
- Privacy-focused self-hosting
- Infrastructure as documentation
- Centralized monitoring
- Automated backups
- Scalable architecture
- Learn by building

---

# 🚀 Future Roadmap

- Kubernetes
- Ansible
- Terraform
- Infrastructure as Code (IaC)
- CI/CD Pipelines
- Home Assistant
- Advanced Monitoring
- GPU Virtualization
- Retrieval-Augmented Generation (RAG)
- AI Agents
- Multi-node Proxmox Cluster
- High Availability (HA)

---

# 📷 Screenshots

Coming soon...

- Proxmox Dashboard
- TrueNAS Dashboard
- Docker Dashboard
- Glance Homepage
- Grafana Dashboards
- Open WebUI
- Jellyfin
- Immich

---

# 📌 Notes

This repository is a living project and will continue evolving as I learn new technologies and improve the infrastructure.

Sensitive information—including credentials, API keys, SSH keys, certificates, secrets, and environment variables—is excluded from this repository.

---

# 📜 License

This project is licensed under the MIT License.
