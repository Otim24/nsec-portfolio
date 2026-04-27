# 🔐 SOC Home Lab — Detection Engineering & Threat Analysis

![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![Focus](https://img.shields.io/badge/Focus-Detection%20Engineering-blue)
![Tools](https://img.shields.io/badge/Stack-Wazuh%20%7C%20Splunk%20%7C%20pfSense%20%7C%20Kali-black)
![Platform](https://img.shields.io/badge/Platform-VMware%20Workstation-lightgrey)

---

## 🧭 Executive Summary

This project simulates a real-world **Security Operations Center (SOC)** focused on detection engineering, log analysis, and threat investigation.

The lab integrates **Splunk (SIEM)** and **Wazuh (EDR)** to detect and analyze adversary activity generated through controlled attack simulations.

### Key Outcomes

- Built a segmented SOC lab with Active Directory
- Engineered a working log pipeline with proper parsing
- Simulated attacker activity (Nmap + SMB brute force)
- Developed detection logic using Splunk SPL
- Performed alert triage and false-positive analysis
- Correlated logs across SIEM and EDR platforms

---

## 🏗️ Architecture Overview

```
Kali (Attacker)
      |
      v
[ pfSense Firewall ]
      |
-------------------------
|                       |
v                       v
Windows Client     Windows Server (AD)
      |                       |
      ---------> Splunk <-----
                 |
                 v
               Wazuh
```

---

## 📸 Screenshots (Lab Evidence)

### 🔍 Splunk Log Analysis
![Splunk Dashboard](screenshots/splunk-dashboard.png)

### 🧠 Splunk Threat Hunting (4625 Failed Logins)
![Splunk Search](screenshots/splunk-search.png)

### 🛡️ Wazuh Agent Status
![Wazuh Agents](screenshots/wazuh-agents.png)

### 🚨 Wazuh Alerts & Detection
![Wazuh Alerts](screenshots/wazuh-alerts.png)

### 🌐 Network Architecture (Optional)
![Architecture](screenshots/network-architecture.png)

---

## 🌐 Network Design

| Zone | Subnet | Purpose |
|------|--------|--------|
| LAN | 10.0.1.0/24 | Attacker (Kali) |
| MONITORING | 10.0.2.0/24 | Splunk + Wazuh |
| ACTIVEDIRECTORY | 10.0.3.0/24 | Windows systems |
| VULNERABLE | 10.0.4.0/24 | Future use |

---

## ⚙️ Phase 1 — Infrastructure & Log Pipeline

### Wazuh Installation

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

- Deployed Wazuh Manager, Indexer, and Dashboard
- Connected Windows endpoints as active agents

---

### Splunk Configuration

```bash
sudo /opt/splunk/bin/splunk enable listen 9997 -auth admin:<password>
sudo /opt/splunk/bin/splunk enable boot-start
```

---

### Windows Log Forwarding (Critical Config)

```ini
[WinEventLog://Security]
index = main
sourcetype = WinEventLog:Security

[WinEventLog://System]
index = main
sourcetype = WinEventLog:System
```

**Why this matters:**
Ensures proper field extraction for detection (EventCode, Account_Name, LogonType)

---

## ⚔️ Phase 2 — Attack Simulation

### Nmap Scan

```bash
nmap -sS -A 10.0.3.2
```

Identified Domain Controller via:
- Kerberos (88)
- LDAP (389)
- Global Catalog (3268)

---

### SMB Brute Force

```bash
netexec smb 10.0.3.2 -u Administrator -p rockyou.txt
```

Simulated password spraying attack.

---

## 🧠 Phase 3 — Detection Engineering

### Failed Logon Detection

```splunk
index=main sourcetype=WinEventLog:Security EventCode=4625
```

---

### Targeted Account Analysis

```splunk
index=main sourcetype=WinEventLog:Security EventCode=4625
| stats count by Account_Name
| sort -count
```

**Finding:**
- Administrator account heavily targeted

---

### Deep Analysis (False Positive Triage)

- Logon Type: 7 → Local unlock
- Source: 127.0.0.1 → Local system
- Process: svchost.exe → System-generated

### Conclusion

Not a brute force attack — system-generated noise.

### SOC Skill Demonstrated

Distinguishing real threats from false positives using context.

---

### Privilege Monitoring

```splunk
index=main EventCode=4672
| stats count by Account_Name
```

---

## 🛡️ Phase 4 — Wazuh Detection

- Real-time alerting
- File integrity monitoring
- MITRE ATT&CK mapping

### SIEM vs EDR

| Tool | Role |
|------|-----|
| Splunk | Deep analysis |
| Wazuh | Real-time detection |

---

## 🧪 Detection Logic Example

```splunk
index=main sourcetype=WinEventLog:Security EventCode=4625
| bucket _time span=1m
| stats count by _time, Account_Name
| where count > 10
```

Detects brute force behavior.

---

## 🧠 Skills Demonstrated

- SIEM Engineering (Splunk)
- EDR Deployment (Wazuh)
- Detection Engineering (SPL)
- Windows Log Analysis
- Network Segmentation (pfSense)
- Threat Simulation (Kali Linux)
- SOC Alert Triage

---

## 🚀 Next Steps

- Build Splunk dashboards
- Simulate Pass-the-Hash attacks
- Integrate pfSense logs
- Deploy Suricata IDS
- Simulate credential dumping

---

## 🧰 Tools Used

| Tool | Purpose |
|------|--------|
| VMware | Virtualization |
| pfSense | Firewall |
| Wazuh | EDR |
| Splunk | SIEM |
| Kali Linux | Attacker |
| Nmap | Recon |
| netexec | SMB attacks |

---

## 📌 Portfolio Context

Part of a cybersecurity portfolio focused on:

- SOC Engineering  
- Detection Engineering  
- Blue Team Operations  

---

**Author:** Timothy Otim  
**Track:** CCNP Security → SOC Engineer
