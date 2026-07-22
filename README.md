# Project Warden — Enterprise-Style Cybersecurity Lab

> A full **attack → detect → respond** pipeline built from scratch on a hardened, self-hosted infrastructure. Wazuh SIEM catches real attacks mapped to MITRE ATT&CK, with automated firewall response — running on a 4-node Proxmox cluster behind an OPNsense firewall.

![License](https://img.shields.io/badge/license-MIT-blue)
![Status](https://img.shields.io/badge/status-active-brightgreen)
![Platform](https://img.shields.io/badge/platform-Proxmox%20VE%209-orange)

---

## Overview

**Project Warden** is a self-directed home lab built to enterprise patterns, designed to demonstrate hands-on security detection-and-response capability. It is not a collection of tutorials followed once — it's a running environment where real attacks are launched against isolated targets, detected by a live SIEM, correlated, and responded to automatically.

Every phase is documented as a standalone, recruiter-readable case study explaining **what** was built, **how**, and **why** — including the trade-offs and the failures that shaped the final design.

**Design principles the lab is built around:**

- **Single hardened entry point** — the only way in is through a locked-down jump box (SSH keys only, no password login, no network root login, firewalled).
- **Strict network isolation** — attack traffic lives on a sealed network that can reach the internet but never the home LAN. This isolation is a legal and safety requirement, not a nice-to-have.
- **Prove it, don't assume it** — a backup that hasn't been restored isn't a backup; a detection that hasn't fired on a real attack isn't a detection.
- **Snapshot before, snapshot after** — every verified working state is captured before anything risky.

---

## Architecture

```
                        Internet
                           │
                  ┌────────┴────────┐
                  │  Cloudflare     │  (Tunnel + Zero Trust — no open router ports)
                  │  Zero Trust     │
                  └────────┬────────┘
                           │
                     ┌─────┴─────┐
                     │ OPNsense  │  Firewall / gateway / segmentation
                     └─────┬─────┘
                           │  isolated lab network
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────┴────┐       ┌─────┴─────┐     ┌──────┴──────┐
   │ Jump box│       │  Wazuh    │     │  Attacker / │
   │ (SSH    │       │  SIEM     │     │  Target VMs │
   │ gateway)│       │           │     │ (Kali, etc.)│
   └─────────┘       └───────────┘     └─────────────┘

   Proxmox VE cluster: pve1 · pve2 · pve3 · pve4
   Quorum tie-breaker: Raspberry Pi QDevice (split-brain protection)
   Remote access: Tailscale (WireGuard) via jump box subnet router
```

- **Virtualization:** 4-node **Proxmox VE 9** cluster with a **Raspberry Pi QDevice** providing quorum tie-breaking, so the even-node cluster can always reach a majority vote and avoid split-brain.
- **Firewall / routing:** **OPNsense** handling gateway duties, DHCP, and network segmentation between the isolated lab range and everything else.
- **Access model:** All remote administration flows over **Tailscale** (WireGuard mesh VPN) into a single **jump box**, which acts as a subnet router for the lab. No machine is reachable directly.

---

## Key Capabilities

### 🎯 Detection & Response (the centerpiece)

A live **Wazuh** SIEM ingests logs from lab hosts and surfaces suspicious activity in real time. The core exercise runs a full incident lifecycle:

1. **Attack** — SSH brute-force with **Hydra**, plus known exploits (vsftpd `CVE-2011-2523`, Samba `CVE-2007-2447`) launched from **Kali Linux** with **Metasploit**.
2. **Detect** — Wazuh correlates the resulting events into alerts.
3. **Map** — each detection is aligned to the **MITRE ATT&CK** framework, the industry-standard language for describing adversary behavior.
4. **Respond** — Wazuh active-response triggers an automated firewall-drop against the attacking host.
5. **Report** — findings written up as a professional incident summary.

### 🛡️ Secure Infrastructure

- Hardened jump box: SSH keys, no password auth, no network root login, **fail2ban**, **UFW**.
- Full network segmentation isolating all attack traffic from production/home devices.
- Public services exposed safely through **Cloudflare Tunnel** and **Cloudflare Zero Trust (mTLS)** — zero ports opened on the home router.

### 📊 Monitoring & Operations

- Infrastructure metrics via **Prometheus** and **Grafana**.
- Containerized services managed with **Docker** / **Docker Compose** behind **Nginx Proxy Manager**.
- Backups to a **Synology NAS** (SHR + Btrfs) using Proxmox **vzdump**, with off-site replication via **GoodSync** following the **3-2-1** rule.

---

## Technology Stack

| Category | Tools |
|---|---|
| **Virtualization** | Proxmox VE 9, Raspberry Pi QDevice, cloud-init templates |
| **OS** | Debian, Ubuntu Server, Kali Linux |
| **Network & Firewall** | OPNsense, VLAN segmentation, UFW, fail2ban |
| **Remote Access** | Tailscale (WireGuard), Termius, SSH keys / jump box |
| **SIEM & Detection** | Wazuh, MITRE ATT&CK |
| **Offensive Tooling** | Metasploit, Hydra, Nmap, Wireshark |
| **Incident Response** | TheHive, Wazuh active-response |
| **Containers & Web** | Docker, Docker Compose, Nginx Proxy Manager |
| **Public Exposure** | Cloudflare Tunnel, Cloudflare Zero Trust (mTLS), Let's Encrypt |
| **Monitoring** | Prometheus, Grafana |
| **Backup & Storage** | Synology NAS (SHR/Btrfs), Proxmox vzdump, GoodSync |
| **Docs & Version Control** | Git, GitHub, Markdown |

---

## Project Roadmap

Built in structured phases, each with a "Done when" gate that proves it's finished. Status reflects the current state of the lab.

| Phase | Focus | Status |
|---|---|---|
| **0** | Hypervisor setup — Proxmox cluster + jump box | ✅ Done |
| **1** | Secure the front door — hardened remote access | ✅ Done |
| **2** | Isolated lab network — OPNsense segmentation | ✅ Done |
| **3** | Vulnerability assessment — scan, identify, report | ✅ Done |
| **4** | Core VMs — templates, snapshot/clone workflow | ✅ Done |
| **5** | Apps & web — Docker, reverse proxy, safe public exposure | 🚧 In progress (SSL + mTLS hardening) |
| **6** | Monitoring — Prometheus / Grafana stack | 🚧 In progress |
| **7** | Attack → detect → respond with Wazuh SIEM | 🚧 In progress (SIEM live; running full simulation) |
| **8** | Backups & encryption — tested NAS restore, off-site replication | ✅ Done |

**Legend:** ✅ complete · 🚧 in progress · 📋 planned

---

## Highlighted Work

- **Wazuh detection lab** — a working SIEM catching real, launched attacks rather than replayed sample logs.
- **MITRE ATT&CK mapping exercise** — translating raw detections into the framework a SOC team actually communicates in.
- **Zero-trust public exposure** — hosting a publicly reachable service with valid HTTPS and mTLS device enforcement, without opening a single inbound port on the home router.
- **Resilient clustering** — solving the even-node quorum problem with a low-cost Raspberry Pi QDevice, demonstrating an understanding of *why* the infrastructure is built the way it is.

---

## Documentation

Each phase is published as its own self-contained case study — no forward references — so a reader can pick up any single deliverable and understand the full context. Documentation follows a consistent three-layer format:

1. **Plain English** — what was done and why it matters.
2. **Jargon defined** — the technical terms, explained.
3. **On-the-job framing** — how this maps to real security work.

---

## A Note on Scope

This is a **home lab built to enterprise patterns** — a learning and portfolio environment, not a production system serving real users. All offensive tooling is confined to an isolated network with no path to the internet or any home device. Deliberately vulnerable targets never leave that isolation.

---

## Repository Layout

Each phase lives in its own numbered folder and is fully self-contained:

```
warden-project/
├── README.md          ← this file (master overview)
├── LICENSE
└── phase-08/
    ├── README.md      ← the phase write-up
    ├── data/          ← screenshots, configs, assets
    └── reports/       ← detailed findings
```

New phases are added as new folders (`phase-07`, `phase-06`, …) without touching
existing work. Every phase README follows the same six-part format:
**What Went Well · Challenges I Faced · Key Learnings · Improvements I Made ·
Technical Skills Demonstrated · Terminologies**.

---

## License

Released under the **MIT License** — see [`LICENSE`](LICENSE) for details. You're welcome to learn from, adapt, or build on this work.

---

*Built independently, two focused hour at a time.*
