# 🛡️ Sher Singh's Cybersecurity & Ethical Hacking Portfolio

<div align="center">

**Offensive Security** · **Penetration Testing** · **Network Exploitation** · **Web Application Hacking**

[![Labs](https://img.shields.io/badge/Labs_Completed-4-brightgreen)]()
[![TryHackMe](https://img.shields.io/badge/TryHackMe-Systems_as_Attack_Vectors_%E2%9C%85-green?logo=tryhackme)](https://tryhackme.com/room/systemsattackvectors)
[![Status](https://img.shields.io/badge/Status-Active-blue)]()
[![Focus](https://img.shields.io/badge/Focus-Offensive_Security-red)]()

</div>

---

## 👤 About Me

Aspiring Cybersecurity Professional with hands-on experience in **offensive penetration testing** and **network security**. Currently building practical skills through structured lab exercises covering wireless attacks, network exploitation, man-in-the-middle techniques, and web application hacking.

**Career Objectives:**
- Secure a role as a **Penetration Tester / SOC Analyst / Cybersecurity Professional**
- Complete extensive hands-on lab scenarios with documented proof of work
- Attain industry-standard certifications: **eJPT → CEH → OSCP → CySA+**
- Build custom security tools that demonstrate real coding ability

---

## 🔥 Featured Labs

### [🧪 Lab 03: ARP Spoofing & Post-Connection Attacks](labs/lab-03-network-post-connection/README.md)
Executed full bidirectional ARP cache poisoning against a Windows target on a NAT Network, intercepted HTTP traffic in real-time, captured cleartext credentials via Wireshark, and performed DNS spoofing to redirect victim traffic. Documented 2 real technical issues encountered (IP forwarding failure, NAT vs NAT Network misconfiguration) and their resolutions.

### [🧪 Lab 04: Man-in-the-Middle Deep Dive](labs/lab-04-mitm-deep-dive/README.md)
Advanced MITM techniques including deep Wireshark traffic analysis with credential extraction filters, HTTPS downgrade testing with SSLstrip, real-time JavaScript injection into HTTP pages, and complete Bettercap MITM workflow. Mapped all techniques to MITRE ATT&CK framework (T1557, T1040, T1185, T1573).

### [🧪 Lab 02: Wireless Pre-Connection Attacks](labs/lab-02-network-pre-connection/README.md)
Audited wireless networks using monitor mode, performed packet sniffing with `airodump-ng`, executed deauthentication attacks to force client reconnections, captured WPA2 4-way handshakes, and cracked network passwords using offline dictionary attacks with `aircrack-ng`.

### [🧪 Lab 01: Penetration Testing Environment Setup](labs/lab-01-environment-setup/README.md)
Built a production-grade isolated penetration testing lab with Kali Linux (attacker) and Windows (target) VMs on a custom NAT Network. Verified inter-VM connectivity and documented the complete architecture for repeatable testing.

---

## 🎯 Skills Demonstrated

| Skill Area | Techniques | Tools |
|---|---|---|
| **Wireless Security** | Monitor mode, packet sniffing, WPA/WPA2 cracking, deauth attacks | `airmon-ng`, `airodump-ng`, `aireplay-ng`, `aircrack-ng` |
| **Network Exploitation** | ARP spoofing/poisoning, network discovery, traffic interception | `arpspoof`, `arp-scan`, `netdiscover`, `bettercap` |
| **MITM Attacks** | ARP cache poisoning, DNS spoofing, HTTPS downgrade, JS injection | `bettercap`, `ettercap`, `sslstrip` |
| **Traffic Analysis** | Deep packet inspection, credential extraction, session analysis | `wireshark`, `urlsnarf`, `driftnet` |
| **Vulnerability Exploitation** | Service scanning, exploit execution, payload delivery | `nmap`, `msfconsole`, `msfvenom` |
| **Post-Exploitation** | Privilege escalation, persistence, hash dumping, keylogging | `meterpreter`, Metasploit Framework |
| **Web App Hacking** | SQLi, XSS, LFI, brute forcing | `sqlmap`, `hydra`, `burpsuite` |

---

## 📈 Learning Progress Tracker

| # | Module | Status | Lab Link |
|:---:|:---|:---:|:---|
| 1 | Setup & Fundamentals | ✅ Completed | [Lab 01](labs/lab-01-environment-setup/README.md) |
| 2 | Network Hacking (Pre-Connection) | ✅ Completed | [Lab 02](labs/lab-02-network-pre-connection/README.md) |
| 3 | Network Hacking (Post-Connection) | ✅ Completed | [Lab 03](labs/lab-03-network-post-connection/README.md) |
| 4 | Man-in-the-Middle Attacks | ✅ Completed | [Lab 04](labs/lab-04-mitm-deep-dive/README.md) |
| 5 | Gaining Access (System Hacking) | 🔒 Coming Soon | — |
| 6 | Post Exploitation | 🔒 Coming Soon | — |
| 7 | Web Application Hacking | 🔒 Coming Soon | — |

> **Legend:** ✅ Completed &nbsp;|&nbsp; 🔒 Coming Soon

---

## 🧰 The Penetration Tester's Toolkit

| Tool | Primary Purpose | Category |
|:---|:---|:---|
| **`nmap`** | Port/service scanning & reconnaissance | Reconnaissance |
| **`msfconsole`** | Enterprise exploitation framework | Exploitation |
| **`msfvenom`** | Custom payload generation | Exploitation |
| **`airmon-ng` / `airodump-ng` / `aircrack-ng`** | Wireless capture & cracking | Wireless |
| **`arpspoof`** | Layer 2 ARP cache poisoning | Network |
| **`bettercap`** | All-in-one MITM framework | Network |
| **`wireshark`** | Deep packet inspection & forensics | Analysis |
| **`ettercap`** | MITM with plugin support (DNS spoof) | Network |
| **`sqlmap`** | Automated SQL injection | Web |
| **`hydra`** | Protocol-agnostic brute forcing | Web |

---

## ⚠️ Issues Encountered & Resolutions

Real-world hacking involves as much debugging as executing attacks. These are genuine problems encountered during labs:

| # | Issue | Root Cause | Resolution |
|:---:|:---|:---|:---|
| 1 | ARP Spoofing caused victim to lose internet entirely | IP Forwarding disabled by default in Linux kernel | `echo 1 > /proc/sys/net/ipv4/ip_forward` before starting spoof |
| 2 | ARP spoof packets not reaching Windows target | VMs on regular "NAT" mode (isolated) instead of "NAT Network" | Reconfigured both VMs to same custom NAT Network |

---

## 📚 Certifications & Learning Path

| Certification | Status |
|:---|:---:|
| **eJPT** (eLearnSecurity Junior Penetration Tester) | 🎯 Target |
| **CEH** (Certified Ethical Hacker) | 🎯 Target |
| **OSCP** (Offensive Security Certified Professional) | 🎯 Target |
| **CySA+** (CompTIA Cybersecurity Analyst) | 🎯 Target |

---

## 📚 Resources & Training

- 🎓 **Course:** [Learn Ethical Hacking from Scratch — Zaid Sabih (Udemy)](https://www.udemy.com/course/learn-ethical-hacking-from-scratch/)
- 🏋️ **Practice:** [TryHackMe — Jr Penetration Tester Path](https://tryhackme.com/path/outline/jrpenetrationtester)
- 🏋️ **Advanced:** [HackTheBox](https://www.hackthebox.com/)
- 📖 **Methodology:** [HackTricks](https://book.hacktricks.xyz/)

### 🏆 TryHackMe Rooms Completed

| Room | Topic | Date | Certificate |
|:---|:---|:---:|:---:|
| [Systems as Attack Vectors](https://tryhackme.com/room/systemsattackvectors) | Exploiting vulnerable & misconfigured systems | Apr 2026 | ✅ [View](https://tryhackme.com/room/systemsattackvectors/congratulations?step=records) |

---

## 📂 Repository Structure

```
Ethical-Hacking-Portfolio/
├── README.md                                    ← You are here
├── labs/
│   ├── lab-01-environment-setup/README.md       ← Lab environment setup
│   ├── lab-02-network-pre-connection/README.md  ← Wireless pre-connection attacks
│   ├── lab-03-network-post-connection/README.md ← ARP spoofing & MITM
│   └── lab-04-mitm-deep-dive/README.md          ← Advanced MITM techniques
└── (more labs coming as sections are completed)
```

---

<div align="center">
  <sub>🛡️ All exercises performed in isolated lab environments. Always act with permission. Hack legally. Hack ethically.</sub>
</div>
