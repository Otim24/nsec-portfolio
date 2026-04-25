#  Network Security Engineering Portfolio

![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![Focus](https://img.shields.io/badge/Focus-Network%20Security-blue)
![Path](https://img.shields.io/badge/Path-CCNP%20Security-orange)
![Tools](https://img.shields.io/badge/Tools-Cisco%20Packet%20Tracer%20%7C%20Wazuh%20%7C%20Suricata-lightgrey)
![License](https://img.shields.io/badge/License-MIT-yellow)

---

##  About Me

I'm **Timothy Otim Lutara**, a cybersecurity student and aspiring Network Security Engineer based in Kampala, Uganda. This repository documents my hands-on journey through enterprise networking, network security, and security engineering building toward **CCNP Security** and a career in network security.

I believe in learning by doing. Every lab here represents a real concept I've configured, broken, debugged, and understood from the ground up.

---

##  Goals

- Master enterprise networking fundamentals (VLANs, routing, switching)
- Build practical network security skills (firewalls, IDS/IPS, VPNs)
- Earn the **CCNP Security** certification
- Contribute to cybersecurity in the East African technology landscape

---

##  Repository Structure

```
nsec-portfolio/
│
├── 01-enterprise-switching/       # VLANs, Trunking, VTP, Inter-VLAN Routing
├── 02-network-security/           # Firewalls, ACLs, Security Policies
├── 03-ids-ips/                    # Suricata, Wazuh, pfSense
├── 04-vpn-and-encryption/         # Site-to-Site VPN, SSL VPN
├── 05-threat-intelligence/        # CTI frameworks, MITRE ATT&CK, Kill Chain
└── README.md
```

---

##  Labs & Projects

---

### 01 — Enterprise Switching
 [`01-enterprise-switching/`](./01-enterprise-switching/)

**Description:**
A full enterprise switching topology built in Cisco Packet Tracer featuring hierarchical network design with Core, Distribution, and Access layers. Includes three segmented VLANs, dynamic DHCP assignment, and inter-VLAN routing through a dedicated router.

**Topology Overview:**

>  **<img width="1557" height="592" alt="image" src="https://github.com/user-attachments/assets/4763ab9d-2722-4947-8710-b3a68c312a60" />
**

**VLAN Design:**

| VLAN | Name | Network | Purpose |
|------|------|---------|---------|
| 10 | IT | 192.168.10.0/24 | IT Department |
| 20 | SALES | 192.168.20.0/24 | Sales Department |
| 30 | HR | 192.168.30.0/24 | Human Resources |

**Server Network:**

| Device | IP | Role |
|--------|-----|------|
| Active Directory | 172.16.0.10 | Authentication Server |
| File Server | 172.16.0.20 | HR File Storage |
| Router Fa0/1 | 172.16.0.1 | Server Network Gateway |

**Key Concepts Covered:**
- IEEE 802.1Q Trunking across hierarchical switch layers
- VLAN Trunking Protocol (VTP)  Server/Client/Transparent modes
- Router on a Stick (inter-VLAN routing via sub-interfaces)
- DHCP pools per VLAN with ip helper-address forwarding
- Spanning Tree Protocol (STP) PortFast configuration
- Static IP assignment for server infrastructure

**Verification:**

> 📸 **[SCREENSHOT: show vlan brief output showing VLAN 10 IT, VLAN 20 SALES, VLAN 30 HR all active]**

> 📸 **[SCREENSHOT: Successful ping from PC0 (VLAN 10) to PC in VLAN 30 proving inter-VLAN routing]**

> 📸 **[SCREENSHOT: Successful ping from PC to AD Server at 172.16.0.10 proving cross-router connectivity]**

**Tools Used:** Cisco Packet Tracer, Cisco IOS CLI, Cisco 2960 Switches, Cisco 2811 Router

---

### 02 — Network Security *(Coming Soon)*
 `02-network-security/`

Topics planned: ACLs, Zone-Based Firewalls, Security Policies, Port Security

---

### 03 — IDS/IPS *(Coming Soon)*
 `03-ids-ips/`

Topics planned: Suricata on pfSense, Wazuh SIEM, custom rule writing, alert analysis

---

### 04 — VPN & Encryption *(Coming Soon)*
 `04-vpn-and-encryption/`

Topics planned: IPSec Site-to-Site VPN, GRE Tunnels, SSL VPN

---

### 05 — Threat Intelligence *(Coming Soon)*
 `05-threat-intelligence/`

Topics planned: CTI lifecycle, MITRE ATT&CK framework, Diamond Model, Kill Chain analysis

---

##  Tools & Technologies

![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?logo=cisco)
![Packet Tracer](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-blue)
![Suricata](https://img.shields.io/badge/IDS-Suricata-orange)
![pfSense](https://img.shields.io/badge/Firewall-pfSense-darkblue)
![Linux](https://img.shields.io/badge/OS-Linux-black?logo=linux)

---

##  Certifications & Studies

| Certification | Status |
|---------------|--------|
| ISC2 Certified in Cybersecurity (CC) | In Progress |
| Cisco CCNA | Studying |
| Cisco CCNP Security | Target |

---

## Connect With Me

[![GitHub](https://img.shields.io/badge/GitHub-Otim24-black?logo=github)](https://github.com/Otim24)

---

##  License

This project is licensed under the **MIT License** — see the [LICENSE](./LICENSE) file for details.

---

*Built with curiosity, configured with purpose.*
