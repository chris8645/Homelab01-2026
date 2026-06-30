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

## 🎯 Objectives

- Learn Proxmox virtualization
- Learn TrueNAS and ZFS storage
- Learn Docker and containerization
- Learn Linux system administration
- Learn networking fundamentals
- Learn monitoring and observability
- Learn self-hosting
- Learn AI deployment using local LLMs
- Learn infrastructure design and documentation

---

## 🖥️ Hardware

| Component | Model |
|-----------|-------|
| CPU | AMD Ryzen 5 5600X |
| GPU | NVIDIA RTX 3060 |
| RAM | 64GB DDR4 |
| Motherboard | ASUS TUF Gaming B550M-PLUS WiFi II |
| Boot Drive | 2TB NVMe SSD |
| Storage | 5 × 12TB SAS HDD *(Planned)* |
| HBA | LSI 9300-8i (IT Mode) |
| Case | JONSBO N6 |

---

## 🏗️ Planned Architecture

```text
                           Internet
                               │
                            Router
                               │
                          Tailscale VPN
                               │
                           Proxmox VE
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
    TrueNAS              Docker VM              Game Server VM
        │                      │                      │
        │                 ├── Immich          ├── Minecraft
        │                 ├── Jellyfin        ├── Palworld
        │                 ├── Vaultwarden     └── More... 
        │                 ├── Pi-hole         
        │                 ├── Grafana
        │                 ├── Prometheus
        │                 └── Ollama
        │
     ZFS RAIDZ2
```

---

## 📦 Planned Services

The homelab will host several self-hosted services for learning and everyday use.

| Service | Purpose | Status |
|----------|---------|--------|
| Proxmox VE | Hypervisor | ⏳ Planned |
| TrueNAS SCALE | Storage Server | ⏳ Planned |
| Immich | Photo Backup & Management | ⏳ Planned |
| Jellyfin | Media Streaming | ⏳ Planned |
| Vaultwarden | Password Manager | ⏳ Planned |
| Pi-hole / AdGuard Home | DNS Ad Blocking | ⏳ Planned |
| Grafana | Monitoring Dashboards | ⏳ Planned |
| Prometheus | Metrics Collection | ⏳ Planned |
| Ollama | Local AI Models | ⏳ Planned |
| Tailscale | Secure Remote Access | ⏳ Planned |
| Game Servers | Multiplayer Game Hosting | ⏳ Planned |

---

## 🎮 Planned Game Servers

Depending on hardware resources and community interest, I plan to host servers for games such as:

- Minecraft
- Palworld
- Satisfactory
- Subnautica

---

## 📚 Learning Goals

Throughout this project I hope to gain practical experience with:

- Virtualization
- Storage systems (ZFS)
- Linux administration
- Docker & Containers
- Networking
- Monitoring & Observability
- Self-hosting
- AI & Local LLMs
- Infrastructure Documentation
- System Administration

---

## 📂 Repository Structure

```text
docs/
├── hardware.md
├── storage.md
├── networking.md
├── monitoring.md
└── troubleshooting.md

diagrams/
docker/
scripts/
images/
```

---

## 📝 Project Log

- [ ] Purchase hardware
- [ ] Assemble server
- [ ] Install Proxmox VE
- [ ] Configure ZFS RAIDZ2
- [ ] Deploy TrueNAS SCALE
- [ ] Configure Docker
- [ ] Deploy Immich
- [ ] Deploy Jellyfin
- [ ] Deploy Vaultwarden
- [ ] Configure Pi-hole / AdGuard Home
- [ ] Configure Grafana & Prometheus
- [ ] Deploy Ollama
- [ ] Deploy game servers
- [ ] Configure automated backups
- [ ] Document the complete infrastructure

---

## 🚧 Current Progress

**Status:** 🟡 Planning

- ✅ Hardware selected
- 🟡 Purchasing components
- ⏳ Building server
- ⏳ Installing Proxmox VE

---

## 🏛️ Design Principles

- Reliability for multiple users
- Data redundancy using ZFS
- Secure remote access with Tailscale
- Privacy-focused self-hosted services
- Centralized monitoring
- Automated backups
- Scalable infrastructure
- Learn by building and documenting

---

## 🚀 Future Plans

After completing the initial homelab, I plan to explore:

- Kubernetes
- Infrastructure automation with Ansible
- Local AI development
- Advanced monitoring
- High Availability (HA)
- Home automation integration
- Game server automation
- CI/CD pipelines
- Infrastructure as Code (IaC)

---

## 📌 Notes

This repository is a living project and will continue to evolve as I learn new technologies and improve the infrastructure.

All credentials, API keys, private keys, certificates, and sensitive configuration files are excluded from this repository.

---

## 📜 License

This project is licensed under the MIT License.
