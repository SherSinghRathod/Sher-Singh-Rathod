# 🔍 Lab 05: SIEM Home Lab — Splunk Enterprise

<div align="center">

**Blue Team** · **SOC Analysis** · **SIEM Architecture** · **Log Management**

[![Splunk](https://img.shields.io/badge/Splunk-Enterprise_9.3.2-orange?logo=splunk)](https://splunk.com)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04.4_LTS_Server-purple?logo=ubuntu)](https://ubuntu.com)
[![Status](https://img.shields.io/badge/Status-Completed-brightgreen)]()
[![Type](https://img.shields.io/badge/Type-Blue_Team_/_Defensive-blue)]()

</div>

---

## 🎯 Objective

Design, build, and configure a fully functional **SIEM (Security Information and Event Management)** home lab from scratch using **Splunk Enterprise**. The goal is to create a real-world defensive security monitoring environment that can ingest logs from target machines, correlate security events, and detect attacks launched from the Kali Linux attacker VM.

This lab bridges the gap between **offensive skills** (previous labs) and **defensive/SOC analysis skills** — demonstrating a complete attacker + defender perspective.

---

## 🏗️ Lab Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    VMware Workstation (Host)                        │
│                    Network: NAT — 192.168.181.x/24                 │
│                                                                     │
│   ┌───────────────────┐          ┌──────────────────────────────┐  │
│   │   Kali Linux      │          │   Windows 10 (Target)        │  │
│   │   (Attacker)      │─ Attack ▶│   IP: 192.168.181.133        │  │
│   │   192.168.x.x     │          │   + Splunk Universal Fwd     │  │
│   └───────────────────┘          └──────────────┬───────────────┘  │
│                                                  │                  │
│                                    Forwards logs │ Port 9997        │
│                                                  ▼                  │
│                                  ┌───────────────────────────────┐  │
│                                  │  Ubuntu Server 24.04.4 LTS    │  │
│                                  │  Splunk Enterprise 9.3.2      │  │
│                                  │  IP: 192.168.181.134          │  │
│                                  │  Web UI: Port 8000            │  │
│                                  │  Indexer: Port 9997           │  │
│                                  └───────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🖥️ Environment Specifications

| Component | Details |
|:---|:---|
| **Hypervisor** | VMware Workstation |
| **SIEM Server OS** | Ubuntu Server 24.04.4 LTS |
| **SIEM Software** | Splunk Enterprise 9.3.2 |
| **SIEM Server IP** | `192.168.181.134` |
| **SIEM Web UI** | `http://192.168.181.134:8000` |
| **Target Machine** | Windows 10 — `192.168.181.133` |
| **Attacker Machine** | Kali Linux |
| **Network Type** | VMware NAT (`192.168.181.x/24`) |
| **Splunk Indexer Port** | `9997` (receives logs from forwarders) |
| **Splunk Web Port** | `8000` (analyst dashboard) |

---

## 📋 Phase 1: Ubuntu Server Installation

### Steps Performed
1. Downloaded **Ubuntu Server 24.04.4 LTS** ISO
2. Created a new VM in VMware Workstation:
   - RAM: 4 GB
   - CPU: 2 cores
   - Disk: 40 GB
   - Network: NAT (same as other lab VMs)
3. Completed Ubuntu Server installation with profile:
   - Username: `splunk`
   - Server name: `splunkserver`
4. Verified network connectivity with `ip a`

### Key Configuration
```bash
# Confirmed Ubuntu VM IP
ip a
# Result: inet 192.168.181.134/24 on ens33
```

---

## 📋 Phase 2: Splunk Enterprise Installation

### Installation Commands
```bash
# Step 1: Update system packages
sudo apt update && sudo apt upgrade -y

# Step 2: Download Splunk Enterprise 9.3.2
wget -O splunk.deb "https://download.splunk.com/products/splunk/releases/9.3.2/linux/splunk-9.3.2-d8bb32809498-linux-2.6-amd64.deb"

# Step 3: Install Splunk
sudo dpkg -i splunk.deb

# Step 4: First start — accepts license and sets admin credentials
sudo /opt/splunk/bin/splunk start --accept-license

# Step 5: Enable Splunk as a boot service
sudo /opt/splunk/bin/splunk enable boot-start

# Step 6: Disable UFW firewall (lab environment)
sudo systemctl disable ufw
sudo systemctl stop ufw
```

### Verification
```bash
# Check Splunk is running
sudo /opt/splunk/bin/splunk status
# Output: splunkd is running

# Confirm port 8000 listening on all interfaces
sudo ss -tlnp | grep 8000
# Output: LISTEN 0 128 0.0.0.0:8000 0.0.0.0:*
```

---

## ✅ Lab Status: FULLY COMPLETE

| Phase | Task | Status |
|:---|:---|:---:|
| 1 | Ubuntu Server 24.04.4 LTS installed in VMware | ✅ |
| 2 | Splunk Enterprise 9.3.2 installed & running | ✅ |
| 3 | Splunk Web UI accessible at `192.168.181.134:8000` | ✅ |
| 4 | Splunk Universal Forwarder installed on Windows 10 VM | ✅ |
| 5 | Windows Event Logs flowing from Windows → Splunk | ✅ |
| 6 | Nmap scan from Kali detected in Splunk (`EventCode=4625`) | ✅ |

---

## 🌐 Phase 3: Splunk Web UI Access

Accessed the Splunk dashboard from **Windows 10 Target VM** browser:
```
http://192.168.181.134:8000
```

Successfully logged in with admin credentials and confirmed the **Search & Reporting** app was available.

---

## 🔍 Phase 4: SPL Searches — Detecting Attacks

With forwarder configured, the following SPL (Search Processing Language) queries can detect attacks launched from Kali:

### Failed Login Attempts (Brute Force Detection)
```spl
index=* EventCode=4625
| stats count by src_ip, user
| where count > 5
| sort -count
```

### New Process Creation (Payload Execution)
```spl
index=* EventCode=4688
| table _time, host, user, New_Process_Name, Creator_Process_Name
```

### PowerShell Script Execution (Post-Exploitation)
```spl
index=* EventCode=4104
| table _time, host, ScriptBlockText
```

### Network Connections Established
```spl
index=* EventCode=3
| table _time, host, DestinationIp, DestinationPort, Image
```

---

## ⚠️ Issues Encountered & Resolutions

| # | Issue | Root Cause | Resolution |
|:---:|:---|:---|:---|
| 1 | `ERR_CONNECTION_REFUSED` on port 8000 | UFW firewall blocking port 8000 | `sudo ufw disable` / `sudo systemctl stop ufw` |
| 2 | `ERR_CONNECTION_TIMED_OUT` even after UFW disabled | Ubuntu and Windows VMs on different VMware subnets (`.101.x` vs `.181.x`) | Changed Ubuntu network adapter to NAT — DHCP assigned same subnet IP |
| 3 | `apt update` Permission Denied | Running without `sudo` | Used `sudo apt update` |
| 4 | "Characters not permitted" in profile setup | Hyphens/apostrophes not allowed in "Your name" field | Used `splunk` (letters only) |

---

## 🎯 Skills Demonstrated

| Skill | Description |
|:---|:---|
| **SIEM Architecture** | Understood and built Splunk's 3-tier model (Forwarder → Indexer → Search Head) |
| **Linux Server Administration** | Installed and configured Ubuntu Server from scratch |
| **Network Troubleshooting** | Diagnosed VMware subnet mismatch causing connection failure |
| **Log Analysis** | Wrote SPL queries to detect security events |
| **Firewall Management** | Configured UFW rules for lab environment |
| **SOC Tooling** | Hands-on with industry-standard SIEM used in real enterprise SOCs |

---

## 🔗 MITRE ATT&CK Mapping

| Technique | ID | Detection via Splunk |
|:---|:---|:---|
| Brute Force | T1110 | EventCode 4625 (Failed Logons) |
| Create or Modify System Process | T1543 | EventCode 4697 |
| Command & Scripting Interpreter (PowerShell) | T1059.001 | EventCode 4104 |
| Valid Accounts | T1078 | EventCode 4624 (Successful Logon after failures) |
| Process Injection | T1055 | EventCode 4688 (Suspicious parent-child processes) |

---

## 📚 References

- [Splunk Enterprise Documentation](https://docs.splunk.com/Documentation/Splunk)
- [Splunk SPL Reference](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference)
- [TryHackMe — SOC Level 1 Path](https://tryhackme.com/path/outline/soclevel1)
- [Windows Security Event IDs Cheat Sheet](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)

---

<div align="center">
  <sub>🛡️ All exercises performed in isolated VMware lab environments. SIEM configured for defensive monitoring and educational purposes only.</sub>
</div>
