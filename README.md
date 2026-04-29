# Project Cerberus

**Enterprise-Style Homelab · Infrastructure, Networking, Virtualisation & Cybersecurity**

---

[![Live Infrastructure Page](https://img.shields.io/badge/Live%20Page-Project%20Cerberus-00e5ff?style=flat-square)](https://jburke-labs.github.io/project-cerberus)
[![Status](https://img.shields.io/badge/Status-Active%20Build-00ff88?style=flat-square)](#roadmap)
[![Docs](https://img.shields.io/badge/Full%20Docs-Notion-ffffff?style=flat-square)](https://www.notion.so/a83fe4fa7fd3833dab2a015171d94ea5)

---

## What This Is

Project Cerberus is an enterprise-style homelab built on physical enterprise hardware, designed to replicate the kind of environment I would expect to support or design in a real business.

The lab is not a collection of boxes. It is a deliberate learning environment where infrastructure is planned, segmented, documented and secured properly — and where every project is written up with the full context of what was built, what broke, how it was isolated, and what was left working at the end.

The name fits the purpose. Cerberus guards the gate. The lab is built around the same principle — controlled access, deliberate segmentation, and nothing getting through that has not been explicitly permitted.

---

## Live Infrastructure Page

The visual hardware and infrastructure registry for the lab is available as a live page:

🌐 **[https://jburke-labs.github.io/project-cerberus](https://jburke-labs.github.io/project-cerberus)**

It covers the full hardware inventory, status of each device, planned infrastructure phases, core skills developed, and the implementation roadmap.

---

## Hardware

The lab is built on real enterprise hardware rather than consumer equipment. This gives direct exposure to the same platforms, interfaces and operational considerations found in production environments.

| Device | Codename | Role | Status |
|---|---|---|---|
| HPE ProLiant DL380 Gen9 | Hyrule | Main virtualisation host / future FC storage platform | ✅ Active |
| Dell PowerEdge R620 | Deku | Virtualisation host / lab compute node | ✅ Active |
| FortiGate 300D | — | Firewall / perimeter security / inter-VLAN routing | ✅ Active |
| Cisco Catalyst 2960X 48-Port | — | Layer 2 access switching / VLAN trunking | ✅ Active |
| 12U Open Air Rack | — | Physical infrastructure housing | ✅ Active |
| HPE MSA 2040 | — | Shared storage via Fibre Channel to DL380 | 🔵 Planned |
| Cisco Catalyst 3650 48P PoE+ | — | Future Layer 3 switching / inter-VLAN routing | 🔵 Planned |
| MikroTik hEX S | — | Previous edge router — retired when FortiGate deployed | 🟡 Offline |

---

## Network Design

The lab is built around a VLAN segmentation strategy designed to mirror how business networks separate traffic by function and trust level. The goal is a default-deny posture between segments — nothing talks to anything unless it has been explicitly permitted.

| VLAN | Purpose | Subnet |
|---|---|---|
| VLAN 10 | Management | 10.0.10.0/24 |
| VLAN 20 | Servers | 10.0.20.0/24 |
| VLAN 30 | Storage | 10.0.30.0/24 |
| VLAN 40 | Clients / Workstation Testing | 10.0.40.0/24 |
| VLAN 50 | Attack / Purple Team | 10.0.50.0/24 |
| VLAN 60 | Defence / Monitoring | 10.0.60.0/24 |
| VLAN 70 | IoT / Untrusted | 10.0.70.0/24 |
| VLAN 80 | Guest | 10.0.80.0/24 |

The FortiGate handles inter-VLAN routing and policy enforcement. The Cisco 2960X carries VLANs via trunk to both Proxmox hosts and the firewall. The planned Cisco 3650 will take over Layer 3 switching and reduce reliance on the firewall for internal routing decisions.

---

## Virtualisation

Both servers run Proxmox VE as the hypervisor, with VLAN-aware bridges allowing VMs and LXC containers to be placed directly into the correct network segment based on their function.

| Host | Role | Workloads |
|---|---|---|
| Hyrule — DL380 Gen9 | Primary Proxmox node | Infrastructure services, SIEM, security tooling |
| Deku — R620 | Secondary Proxmox node | Additional VMs, WireGuard VPN, DNS, reverse proxy |

---

## Completed Projects

Each project below has a dedicated repository with a full write-up covering the design decisions, troubleshooting process, and outcome.

---

### 🔐 [NGINX Proxy Manager + AdGuard + Wildcard Certificate](https://github.com/jburke-labs/nginx-proxy-wildcard-cert)

Migrated from a CLI-based NGINX reverse proxy to NGINX Proxy Manager. Deployed internal DNS rewrites through AdGuard Home and implemented a mkcert wildcard certificate for `*.cerberus.home.arpa`. Services are now reachable by FQDN over trusted internal HTTPS rather than IP and port.

Resolved a Wazuh login loop caused by mixed HTTP/HTTPS frontend handling, and a browser security warning caused by stale cached trust state — diagnosed using DevTools Security tab. Created a repeatable onboarding pattern for future internal services.

**Tech:** NGINX Proxy Manager · AdGuard Home · mkcert · internal PKI · Proxmox LXC

---

### 🔒 [WireGuard VPN — Double NAT Deployment and Troubleshooting](https://github.com/jburke-labs/wireguard-vpn-deployment)

Deployed WireGuard in a Proxmox LXC behind a FortiGate operating in a double-NAT arrangement with a Virgin Media upstream router. Validated the full connection path end to end using FortiGate packet captures, tcpdump on the LXC, and structured layer-by-layer fault isolation.

Six distinct issues resolved in sequence — IP forwarding disabled, missing NAT masquerade rules, malformed config file, interface not instantiated, double-NAT not fully accounted for, and finally a client endpoint set to an internal IP. Tunnel established and internal lab access confirmed.

**Tech:** WireGuard · Proxmox LXC · FortiGate VIP · iptables · tcpdump · WGDashboard

---

### 🖥️ [Snipe-IT Replatform — Proxmox LXC to Dedicated Docker Host](https://github.com/jburke-labs/snipe-it-replatform)

Migrated a production Snipe-IT deployment from a Proxmox community-script LXC to a dedicated Ubuntu 24.04 host running Docker Compose with a Caddy reverse proxy. Full data migration covering 82 users and 148 assets, resolving a container symlink upload path mismatch and an APP_KEY alignment issue that would have caused silent decryption failures.

Corporate deployment design included FortiGate ACL allowlisting, default-deny network posture, SSH hardening, and ISO 27001-aligned evidence planning.

**Tech:** Docker Compose · Ubuntu 24.04 · Caddy · MariaDB · Snipe-IT · ISO 27001

---

### 🛡️ [Wazuh SOC Automation Pipeline](https://github.com/jburke-labs/wazuh-soc-automation) *(Active Build)*

Wazuh is deployed and receiving telemetry from multiple endpoints including Windows, Linux and FortiGate syslog. The pipeline architecture is defined and being built out in stages — Wazuh for detection, promotion logic to filter noise, Shuffle for orchestration, and DFIR-IRIS for incident case management.

The first automated use case is SSH brute-force detection with a safe, time-bound IP containment response via Wazuh Active Response.

**Tech:** Wazuh · OpenSearch · Shuffle · DFIR-IRIS · FortiGate syslog · detection engineering

---

## Roadmap

The lab is being built out in phases. Completed phases are marked, in-progress phases are noted, and planned phases are listed in order of priority.

| Phase | Focus | Status |
|---|---|---|
| 1 | Network and infrastructure foundations — VLANs, FortiGate, Cisco, Proxmox, storage | ✅ Complete |
| 2 | Internal DNS, reverse proxy, wildcard certificate | ✅ Complete |
| 3 | WireGuard VPN — remote access through double NAT | ✅ Complete |
| 4 | Snipe-IT replatform — Docker host migration | ✅ Complete |
| 5 | Wazuh SIEM — deployment and endpoint telemetry | ✅ Complete |
| 6 | Wazuh SOC pipeline — promotion logic, Shuffle, DFIR-IRIS | 🔄 In Progress |
| 7 | Windows enterprise lab — AD, DNS, GPO, PKI, file services | ⏭️ Planned |
| 8 | DNS filtering expansion — AdGuard, Unbound, conditional forwarding | ⏭️ Planned |
| 9 | Purple team range — isolated attack and defence VLANs | ⏭️ Planned |
| 10 | Docker and containerisation — Portainer, Gitea, monitoring stack | ⏭️ Planned |
| 11 | Kubernetes / k3s — lightweight cluster for cloud-native learning | ⏭️ Planned |
| 12 | Azure, Entra ID, Intune and hybrid cloud integration | ⏭️ Planned |

---

## Documentation

Every project in this lab has a matching write-up in Notion covering the full context — what was being built, why decisions were made that way, what went wrong, how it was isolated, and what was left working at the end.

The GitHub repositories give you the summary. The Notion workspace gives you the depth.

📓 **[Project Cerberus — Notion Workspace](https://dent-trampoline-c53.notion.site/Project-Cerberus-Public-Portfolio-351fe4fa7fd3816384c5d06cabbc1a9d?source=copy_link)**

Notion documentation includes:

- Hardware inventory and asset register
- Interface maps and VLAN design
- Full project runbooks and implementation notes
- Troubleshooting logs in STAR format
- Configuration references
- Phased roadmap and task tracking

---

## Skills Developed

```
Infrastructure            Networking                Security
──────────────────────    ──────────────────────    ──────────────────────
Proxmox VE                FortiGate 300D            Wazuh SIEM
VMware ESXi               Cisco Catalyst            AdGuard DNS filtering
Ubuntu / Debian           VLAN segmentation         Firewall policy design
Windows Server            Inter-VLAN routing        Internal PKI / mkcert
Docker Compose            WireGuard VPN             Reverse proxy / TLS
Linux administration      DNS architecture          ISO 27001 alignment
Snipe-IT                  NAT / double NAT          Packet capture / tcpdump
Proxmox Backup            Trunk / access ports      Structured troubleshooting
```

---

## Connect

- 💼 [LinkedIn](https://www.linkedin.com/in/jason-burke-822000142)
- 🌐 [Live Infrastructure Page](https://jburke-labs.github.io/project-cerberus)
- 📓 [Notion Documentation](https://dent-trampoline-c53.notion.site/Project-Cerberus-Public-Portfolio-351fe4fa7fd3816384c5d06cabbc1a9d)

---

*Built, broken, fixed and documented in a homelab in Northern Ireland.*
