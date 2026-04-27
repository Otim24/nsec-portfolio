# SOC Home Lab — Attack Detection & Log Analysis

![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![Tools](https://img.shields.io/badge/Tools-Wazuh%20%7C%20Splunk%20%7C%20pfSense%20%7C%20Kali-blue)
![Platform](https://img.shields.io/badge/Platform-VMware%20Workstation-lightgrey)
![OS](https://img.shields.io/badge/OS-Ubuntu%20%7C%20Windows%20Server%202019%20%7C%20Windows%2010-orange)

---

## Overview

This lab simulates a real-world Security Operations Centre (SOC) environment built entirely on VMware Workstation. The goal is to practice the full SOC workflow: build the infrastructure, monitor it with industry-standard tools, attack it from a dedicated attacker machine, and detect those attacks through log analysis and SIEM correlation.

The lab covers:
- Network segmentation and firewall configuration with pfSense
- Endpoint monitoring with Wazuh SIEM
- Centralized log ingestion and analysis with Splunk Enterprise
- Offensive techniques using Kali Linux (Nmap, Hydra, netexec)
- Detection engineering and threat hunting with SPL (Splunk Processing Language)

---

## Lab Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        VMware Workstation                        │
│                                                                  │
│  ┌──────────────┐    ┌────────────────────────────────────────┐ │
│  │   pfSense    │    │           Network Segments              │ │
│  │  Firewall    │    │                                         │ │
│  │  10.0.1.1    │    │  LAN (10.0.1.0/24)    → Kali Linux     │ │
│  │  10.0.2.1    │    │  MONITORING (10.0.2.0/24) → Ubuntu     │ │
│  │  10.0.3.1    │    │  ACTIVEDIR (10.0.3.0/24) → Win VMs     │ │
│  │  10.0.4.1    │    │  VULNERABLE (10.0.4.0/24) → Future use │ │
│  └──────────────┘    └────────────────────────────────────────┘ │
│                                                                  │
│  ┌──────────────────┐   ┌──────────────────┐                    │
│  │  Ubuntu Server   │   │   Kali Linux     │                    │
│  │  10.0.2.10       │   │   10.0.1.10      │                    │
│  │  ├─ Splunk 9.4.1 │   │   (Attacker)     │                    │
│  │  └─ Wazuh 4.14.5 │   └──────────────────┘                    │
│  └──────────────────┘                                            │
│                                                                  │
│  ┌──────────────────┐   ┌──────────────────┐                    │
│  │  Windows Client  │   │  Windows Server  │                    │
│  │  10.0.3.101      │   │  10.0.3.2        │                    │
│  │  Windows 10 Ent  │   │  Server 2019     │                    │
│  │  + Splunk UF     │   │  + Active Dir    │                    │
│  │  + Wazuh Agent   │   │  + Splunk UF     │                    │
│  └──────────────────┘   │  + Wazuh Agent   │                    │
│                          └──────────────────┘                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## Environment Specifications

| Component | Details |
|-----------|---------|
| Hypervisor | VMware Workstation 17.5 |
| Host Machine | AMD Ryzen 7 5800H, 16GB RAM |
| pfSense | 2.8.1-RELEASE (amd64) |
| Ubuntu | 24.04 LTS (MONITORINGMACHINE) |
| Wazuh | v4.14.5 (Manager + Dashboard) |
| Splunk | Enterprise 9.4.1 |
| Windows Client | Windows 10 Enterprise (10.0.19045.3803) |
| Windows Server | Windows Server 2019 Standard Evaluation |
| Attacker | Kali Linux 2026.1 |

---

## Network Design

pfSense was configured as the central firewall and router connecting all lab segments. Each interface represents a different security zone.

| Interface | Name | Subnet | Purpose |
|-----------|------|--------|---------|
| em0 | WAN | DHCP (192.168.202.x) | Internet access |
| em1 | LAN | 10.0.1.0/24 | Analyst workstation (Kali) |
| em2 | MONITORINGMACHINE | 10.0.2.0/24 | SIEM & monitoring tools |
| em3 | ACTIVEDIRECTORY | 10.0.3.0/24 | Windows endpoints & AD |
| em4 | VULNERABLE | 10.0.4.0/24 | Reserved for future labs |

### Key Firewall Rules Configured

| Rule | Interface | Source | Destination | Port | Purpose |
|------|-----------|--------|-------------|------|---------|
| Allow Splunk Forwarding | ACTIVEDIRECTORY | 10.0.3.0/24 | 10.0.2.10 | 9997/TCP | Log forwarding from Windows to Splunk |
| Allow Kali to AD | LAN | 10.0.1.10 | 10.0.3.0/24 | Any | Attack simulation from Kali |
| Allow LAN out | LAN | 10.0.1.0/24 | Any | Any | Internet access for Kali |

---

## Tools & Technologies

### Wazuh (SIEM + EDR)
Wazuh was deployed as an all-in-one installation on Ubuntu, providing endpoint detection and response (EDR) capabilities across both Windows endpoints.

**Components installed:**
- Wazuh Manager — central management and correlation engine
- Wazuh Indexer — OpenSearch-based log storage and indexing
- Wazuh Dashboard — web UI for alerts, agents, and visualizations

**Agents deployed:**
| Agent ID | Name | IP | OS | Status |
|----------|------|----|----|--------|
| 001 | Client01 | 10.0.3.101 | Windows 10 Enterprise | Active |
| 002 | WInServer | 10.0.3.2 | Windows Server 2019 | Active |

### Splunk Enterprise (Log Analysis & Threat Hunting)
Splunk was installed on the same Ubuntu machine to serve as the primary log analysis platform. Windows Event Logs are forwarded from both endpoints using the Splunk Universal Forwarder.

**Log pipeline:**
```
Windows Endpoints (Security.evtx + System.evtx)
        ↓  [Splunk Universal Forwarder v10.2.2]
        ↓  TCP Port 9997
        ↓  [pfSense — ACTIVEDIRECTORY → MONITORING firewall rule]
        ↓
Ubuntu Splunk Server (10.0.2.10:9997)
        ↓
Splunk Indexes (main)
        ↓
Analyst searches on Kali (http://10.0.2.10:8000)
```

---

## Phase 1 — Infrastructure Setup

### 1.1 Wazuh Installation on Ubuntu

The official Wazuh installation assistant was used to deploy all components in a single-node configuration.

```bash
# Download the installer
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh

# Run all-in-one installation
sudo bash wazuh-install.sh -a
```

**Access:** `https://10.0.2.10` — credentials printed at end of installation.

### 1.2 Wazuh Agent Deployment on Windows

Agents were deployed on both Windows machines through the Wazuh Dashboard:

**Dashboard → Agents → Deploy new agent → Windows**

The installer was deployed via MSI and configured to point to the Wazuh Manager at `10.0.2.10`.

**Result:** Both Windows endpoints appeared as Active agents within minutes of installation.

### 1.3 Splunk Installation on Ubuntu

Splunk Enterprise was installed and configured to listen for incoming forwarder data:

```bash
# Enable Splunk to receive data on port 9997
sudo /opt/splunk/bin/splunk enable listen 9997 -auth admin:<password>

# Start Splunk on boot
sudo /opt/splunk/bin/splunk enable boot-start
```

### 1.4 Splunk Universal Forwarder on Windows

The Splunk Universal Forwarder (v10.2.2) was installed on both Windows machines via the MSI wizard with the following configuration:

- **Receiving Indexer:** `10.0.2.10:9997`
- **Deployment Server:** None (standalone configuration)

After installation, the `inputs.conf` file was configured manually to correctly parse Windows Event Logs:

**File location:** `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`

```ini
[WinEventLog://Security]
index = main
sourcetype = WinEventLog:Security
disabled = false

[WinEventLog://System]
index = main
sourcetype = WinEventLog:System
disabled = false
```

> **Why this matters:** Without this configuration, Splunk assigns the wrong sourcetype (`syslog`) to Windows Event Logs, which prevents field extraction. Using `WinEventLog://` tells Splunk to use the Windows Event Log API directly, enabling proper parsing of fields like `EventCode`, `Account_Name`, `IpAddress`, and `LogonType`.

**Restart the forwarder after any configuration change:**
```powershell
Restart-Service SplunkForwarder
```

### 1.5 Confirming Log Ingestion

After configuration, log ingestion was verified in Splunk using:

```
| tstats count by index
```

**Result:** 17,702 initial events confirmed in the main index.

Sourcetype verification:
```
index=main | stats count by sourcetype
```

**Result:**
| Sourcetype | Count |
|------------|-------|
| WinEventLog:Security | 15,861 |
| WinEventLog:System | 4,084 |
| syslog (legacy) | 17,702 |

---

## Phase 2 — Reconnaissance (Attack Simulation)

### 2.1 Nmap Port Scan — Windows Client

**Attacker:** Kali Linux (10.0.1.10)  
**Target:** Windows Client (10.0.3.101)

```bash
nmap -sS -A 10.0.3.101
```

**Scan Results:**
```
PORT     STATE  SERVICE        VERSION
135/tcp  open   msrpc          Microsoft Windows RPC
139/tcp  open   netbios-ssn    Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds?
OS: Microsoft Windows 10 (1903 - 21H1)
SMB signing: enabled but not required
```

### 2.2 Nmap Port Scan — Windows Server

**Target:** Windows Server (10.0.3.2)

```bash
nmap -sS -A 10.0.3.2
```

**Scan Results:**
```
PORT     STATE  SERVICE         VERSION
53/tcp   open   domain          Simple DNS Plus
88/tcp   open   kerberos-sec    Microsoft Windows Kerberos
135/tcp  open   msrpc           Microsoft Windows RPC
139/tcp  open   netbios-ssn     Microsoft Windows netbios-ssn
389/tcp  open   ldap            Microsoft AD LDAP (Domain: soc.local)
445/tcp  open   microsoft-ds?
3268/tcp open   ldap            Microsoft AD LDAP
5357/tcp open   http            Microsoft HTTPAPI 2.0
OS: Windows Server 2019
Domain: soc.local
```

> **SOC Note:** The presence of ports 88 (Kerberos), 389 (LDAP), and 3268 (Global Catalog) immediately identifies this machine as an Active Directory Domain Controller — a high-value target for attackers.

### 2.3 SMB Brute Force — Windows Server

**Tool:** netexec (formerly crackmapexec)  
**Target:** Windows Server Administrator account

```bash
netexec smb 10.0.3.2 -u Administrator -p /usr/share/wordlists/rockyou.txt --ignore-pw-decoding
```

netexec successfully connected to the target over SMB port 445 and began password spraying the Administrator account with entries from the rockyou wordlist.

---

## Phase 3 — Detection & Threat Hunting in Splunk

### Key Windows Event IDs Reference

Understanding these Event IDs is fundamental to SOC work. They form the basis of all detection queries in this lab.

| Event ID | Category | Description | SOC Relevance |
|----------|----------|-------------|---------------|
| 4624 | Authentication | Successful logon | Baseline — monitor for unusual times/sources |
| 4625 | Authentication | Failed logon | Brute force indicator |
| 4634 | Authentication | Account logoff | Session tracking |
| 4648 | Authentication | Logon with explicit credentials | Pass-the-Hash indicator |
| 4672 | Privilege | Special privileges assigned at logon | Admin access tracking |
| 4720 | Account Mgmt | User account created | Persistence indicator |
| 4724 | Account Mgmt | Password reset attempt | Account takeover indicator |
| 4728 | Account Mgmt | User added to privileged group | Privilege escalation |
| 4732 | Account Mgmt | User added to local group | Lateral movement |
| 4771 | Kerberos | Kerberos pre-auth failed | AS-REP Roasting / password spray |
| 4776 | Authentication | Credential validation attempt | NTLM brute force |
| 7045 | System | New service installed | Malware persistence |
| 4698 | Scheduled Tasks | Scheduled task created | Persistence mechanism |

### 3.1 Verify Log Ingestion

```splunk
index=main | stats count by sourcetype
```

Confirms that Windows Security and System logs are being received and correctly parsed.

### 3.2 Failed Logon Detection (EventCode 4625)

```splunk
index=main sourcetype=WinEventLog:Security EventCode=4625
```

**Finding:** 286 failed logon events detected across the environment.

### 3.3 Identify Most Targeted Accounts

```splunk
index=main sourcetype=WinEventLog:Security EventCode=4625 
| stats count by Account_Name 
| sort -count
```

**Results:**

| Account Name | Failed Attempts | Analysis |
|-------------|----------------|----------|
| Administrator | 279 | 🚨 High — brute force target |
| - (blank) | 269 | Automated/service auth failures |
| WINSERVER$ | 17 | Machine account — normal AD traffic |
| WIN-1QNER218B3J | 4 | Machine account authentication |
| SOC | 3 | Local account — likely manual errors |

> **SOC Analysis:** The Administrator account with 279 failed login attempts is a significant red flag. In a real environment, this would trigger an immediate alert and investigation. The source IP and timing would be examined to determine if this is an automated brute force attack or a misconfigured service.

### 3.4 Investigate Failed Login Details

Expanding a 4625 event revealed the following structure:

```
EventCode=4625
ComputerName=WinServer.soc.local
Account Name: Administrator
Account Domain: SOC
Failure Reason: Unknown user name or bad password
Status: 0xC000006D
Sub Status: 0xC000006A
Logon Type: 7
Source Network Address: 127.0.0.1
Caller Process Name: C:\Windows\System32\svchost.exe
```

**Analysis of this specific event:**
- **Logon Type 7** = Screen unlock attempt (not a remote network attack)
- **Source: 127.0.0.1** = Local machine origin (not an external attacker)
- **svchost.exe** = Windows service process triggered it

This demonstrates an important SOC skill: **distinguishing noise from real attacks**. These 279 failures appear to be from local service authentication issues or screen unlock attempts, not the Kali brute force — showing that not every alert is malicious.

### 3.5 Detect Successful Logons

```splunk
index=main sourcetype=WinEventLog:Security EventCode=4624 
| stats count by Account_Name, ComputerName 
| sort -count
```

Tracks all successful authentications for baseline comparison.

### 3.6 Privilege Use Detection

```splunk
index=main sourcetype=WinEventLog:Security EventCode=4672 
| stats count by Account_Name 
| sort -count
```

Special privileges (admin-level) assigned at logon. Elevated counts on unexpected accounts indicate privilege escalation.

---

## Phase 4 — Cross-Platform Detection with Wazuh

Wazuh provides real-time endpoint detection independent of Splunk. With both agents active, the following is observable in the Wazuh Dashboard:

- **Agents view:** Both Client01 (10.0.3.101) and WInServer (10.0.3.2) show as Active
- **Alerts:** Wazuh generates automated alerts for suspicious activity including failed logins, new services, and file integrity violations
- **MITRE ATT&CK mapping:** Wazuh maps alerts to the MITRE ATT&CK framework automatically

---

## SPL Query Reference

A collection of useful Splunk queries used throughout this lab:

```splunk
-- Check all sourcetypes in the environment
index=main | stats count by sourcetype

-- All failed logins
index=main sourcetype=WinEventLog:Security EventCode=4625

-- Failed logins by account (ranked)
index=main sourcetype=WinEventLog:Security EventCode=4625 
| stats count by Account_Name | sort -count

-- Successful logins
index=main sourcetype=WinEventLog:Security EventCode=4624

-- Admin privilege use
index=main sourcetype=WinEventLog:Security EventCode=4672 
| stats count by Account_Name | sort -count

-- New user accounts created
index=main sourcetype=WinEventLog:Security EventCode=4720

-- New services installed (malware persistence)
index=main sourcetype=WinEventLog:System EventCode=7045

-- All events from a specific host
index=main host=WinServer

-- Events in a time window
index=main sourcetype=WinEventLog:Security earliest=-1h latest=now

-- Count events by EventCode (top event types)
index=main sourcetype=WinEventLog:Security 
| stats count by EventCode | sort -count
```

---

## Lessons Learned

### Technical Challenges

**1. Wazuh Indexer Download Timeout**  
The Wazuh Indexer package is 874MB. On slow connections this times out. Resolution: run `sudo bash wazuh-install.sh -a -o` to retry with overwrite flag, which resumes from cached packages.

**2. Splunk Sourcetype Misconfiguration**  
Using `add monitor` with `.evtx` file paths assigned the wrong sourcetype (`syslog`) which prevented field extraction. The correct approach is using `WinEventLog://` in `inputs.conf` to use the Windows Event Log API.

**3. Cross-Subnet Routing**  
pfSense blocked traffic between the ACTIVEDIRECTORY (`10.0.3.0/24`) and MONITORINGMACHINE (`10.0.2.0/24`) subnets by default. Explicit firewall rules were required to allow Splunk forwarder traffic on port 9997.

**4. SMB Brute Force Tooling**  
Hydra's SMB module does not handle SMBv2/v3 (required by Windows Server 2019). netexec (the successor to crackmapexec) handles modern SMB correctly.

### Key SOC Concepts Demonstrated

- **Alert triage:** Not all 4625 events are brute force attacks — logon type and source address determine legitimacy
- **Log pipeline:** Data must be properly parsed at ingestion; wrong sourcetype = no field extraction = no detection
- **Network segmentation:** Firewall rules between subnets are critical and must be explicitly configured
- **Cross-tool correlation:** Wazuh and Splunk provide complementary visibility — Wazuh for real-time EDR alerts, Splunk for deep log analysis and historical hunting

---

## Next Steps

- [ ] Complete SMB brute force simulation and capture 4625 events from Kali's IP
- [ ] Mimikatz credential dumping and detection via EventCode 4624/4672
- [ ] Pass-the-Hash attack simulation (EventCode 4648)
- [ ] Create Splunk alerts for brute force threshold (>10 failures in 1 minute)
- [ ] Build Splunk dashboard for SOC overview
- [ ] Integrate pfSense logs into Splunk for network-layer visibility
- [ ] AS-REP Roasting against Active Directory (EventCode 4771)
- [ ] Deploy Suricata IDS on pfSense for network-based detection

---

## Tools Used

| Tool | Version | Purpose |
|------|---------|---------|
| VMware Workstation | 17.5 | Hypervisor |
| pfSense | 2.8.1 | Firewall & network segmentation |
| Wazuh | 4.14.5 | SIEM + EDR |
| Splunk Enterprise | 9.4.1 | Log analysis & threat hunting |
| Splunk Universal Forwarder | 10.2.2 | Windows log forwarding |
| Kali Linux | 2026.1 | Attack simulation |
| Nmap | 7.98 | Port scanning & OS fingerprinting |
| netexec | Latest | SMB brute force |
| Hydra | 9.6 | Password spraying |

---

## References

- [Wazuh Documentation](https://documentation.wazuh.com)
- [Splunk Documentation](https://docs.splunk.com)
- [MITRE ATT&CK Framework](https://attack.mitre.org)
- [Windows Security Event IDs](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/)
- [pfSense Documentation](https://docs.netgate.com/pfsense/en/latest/)

---

*Part of the [nsec-portfolio](https://github.com/Otim24/nsec-portfolio) — documenting my path to CCNP Security and SOC Engineering.*
