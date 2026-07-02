# 🏠 Homelab01-2026

> Self-hosted homelab built with Proxmox, TrueNAS, Docker, and AI services for learning infrastructure, virtualization, networking, Linux administration, and self-hosting.

---

## 📖 About

Hi! I'm a second-year Computer Science student building this homelab to gain hands-on experience with technologies used in real-world IT environments.

This repository documents my journey from planning and hardware selection to deployment, troubleshooting, and continuous improvements.

The goal of this project is to gain practical experience by designing, deploying, and maintaining a production-inspired homelab while documenting everything I learn.

---

## 🎯 Project Goals

This homelab is designed to serve two purposes:

- A learning platform for virtualization, storage, networking, Linux, Docker, and AI.
- A self-hosted server providing reliable services for multiple users, including media streaming, photo management, password management, game servers, and AI services.

The infrastructure is being designed with reliability, security, scalability, and maintainability in mind rather than as a single-user lab.

---

# ✨ Features

- 🖥️ Proxmox VE virtualization
- 💾 TrueNAS SCALE with ZFS RAIDZ2 storage
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
| Storage | 5 × 12TB SAS HDD *(Planned)* |
| HBA | LSI 9300-8i (IT Mode) |
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
   ZFS RAIDZ2 Pool       ├── Glance             ├── Minecraft
        │                ├── Dockge             ├── Palworld
        ├── Media         ├── Open WebUI        ├── Satisfactory
        ├── Photos        ├── Ollama            ├── Project Zomboid*
        ├── Documents     ├── Jellyfin          └── More...
        ├── Backups       ├── Immich
        ├── ISOs          ├── Vaultwarden
        └── Docker Data   ├── Pi-hole / AdGuard
                           ├── Grafana
                           ├── Prometheus
                           └── Uptime Kuma
```

---

# 📦 Planned Services

| Service | Purpose | Status |
|----------|---------|--------|
| Proxmox VE | Hypervisor | ⏳ Planned |
| TrueNAS SCALE | Storage Server | ⏳ Planned |
| Docker | Container Platform | ⏳ Planned |
| Dockge | Docker Compose Management | ⏳ Planned |
| Glance | Homelab Dashboard | ⏳ Planned |
| Immich | Photo Backup & Management | ⏳ Planned |
| Jellyfin | Media Streaming | ⏳ Planned |
| Vaultwarden | Password Manager | ⏳ Planned |
| Pi-hole / AdGuard Home | DNS Ad Blocking | ⏳ Planned |
| Grafana | Monitoring Dashboards | ⏳ Planned |
| Prometheus | Metrics Collection | ⏳ Planned |
| Uptime Kuma | Service Monitoring | ⏳ Planned |
| Ollama | Local AI Model Runtime | ⏳ Planned |
| Open WebUI | Multi-user AI Interface | ⏳ Planned |
| Tailscale | Secure Remote Access | ⏳ Planned |
| Game Servers | Multiplayer Hosting | ⏳ Planned |

---

# 🤖 AI Services

The homelab will host a private AI platform for household users.

### Planned Features

- Multi-user accounts
- Private chat history for each user
- Local LLM inference
- Multiple selectable AI models
- GPU acceleration
- Future Retrieval-Augmented Generation (RAG)

Planned software:

- Ollama
- Open WebUI

---

# 🎮 Planned Game Servers

Depending on available resources, I plan to host servers for games including:

- Minecraft
- Palworld
- Satisfactory
- Project Zomboid
- Valheim *(Future)*

---

# 📚 Learning Objectives

Throughout this project I hope to gain practical experience with:

- Linux Administration
- Virtualization
- Docker
- Storage Systems (ZFS)
- Networking
- Monitoring & Observability
- Self-hosting
- Local AI Deployment
- Infrastructure Design
- Documentation
- Automation
- System Administration

---

# 📂 Repository Structure

```text
.
├── docs/
│   ├── hardware.md
│   ├── storage.md
│   ├── networking.md
│   ├── docker.md
│   ├── ai.md
│   ├── monitoring.md
│   ├── backups.md
│   └── troubleshooting.md
│
├── docker/
│   ├── glance/
│   ├── dockge/
│   ├── immich/
│   ├── jellyfin/
│   ├── vaultwarden/
│   ├── open-webui/
│   ├── ollama/
│   ├── grafana/
│   ├── prometheus/
│   ├── uptime-kuma/
│   └── pihole/
│
├── diagrams/
├── images/
├── scripts/
└── README.md
```

---

# 📝 Deployment Roadmap

- [ ] Purchase remaining hardware
- [ ] Assemble server
- [ ] Update motherboard BIOS
- [ ] Install Proxmox VE
- [ ] Configure networking
- [ ] Deploy TrueNAS SCALE
- [ ] Configure ZFS RAIDZ2
- [ ] Create datasets and shares
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

**Status:** 🟡 Planning

| Task | Status |
|------|--------|
| Hardware Planning | ✅ Complete |
| Purchasing Components | 🟡 In Progress |
| Server Assembly | ⏳ Pending |
| Software Deployment | ⏳ Pending |

---

# 🏛️ Design Principles

- Reliability over complexity
- Secure remote access
- Privacy-focused self-hosting
- Infrastructure as documentation
- Centralized monitoring
- Automated backups
- Scalable architecture
- Learn by building

---

# 🚀 Future Roadmap

After completing the initial homelab, I plan to explore:

- Kubernetes
- Ansible
- Terraform
- Infrastructure as Code (IaC)
- CI/CD pipelines
- Home Assistant integration
- Advanced monitoring
- GPU virtualization
- Retrieval-Augmented Generation (RAG)
- AI Agents
- Multi-node Proxmox Cluster
- High Availability (HA)

---

# 📷 Screenshots

Coming soon...

- Proxmox Dashboard
- TrueNAS Dashboard
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
