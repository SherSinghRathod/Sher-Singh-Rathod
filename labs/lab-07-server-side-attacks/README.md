# 🧪 Lab 07: Server-Side Attacks — Gaining Access

## 📋 Lab Overview

| Detail | Value |
|---|---|
| **Objective** | Exploit vulnerable services on a target server to gain remote access |
| **Attacker** | Kali Linux |
| **Target** | Metasploitable 2 |
| **Network** | VMware NAT Network (same subnet) |
| **Key Technique** | Service enumeration → exploiting insecure remote login (rlogin) |
| **Date** | May 2026 |

---

## 🎯 What is Server-Side Attack?

In previous labs, all attacks required the victim to **do something** — visit a website (MITM), connect to WiFi (deauth). Server-side attacks are different:

> **Server-side attacks target the server directly — no user interaction required. If a service is running and vulnerable, the attacker can exploit it remotely.**

This is the core of **penetration testing** — scanning a target, finding open services, and exploiting them to gain access.

---

## 🖥️ Lab Setup

### Metasploitable 2

Metasploitable 2 is an **intentionally vulnerable** Linux virtual machine designed for practicing penetration testing. It runs dozens of misconfigured and outdated services.

**Setup steps:**
1. Downloaded Metasploitable 2 VM image
2. Imported into VMware Workstation
3. Connected to the same NAT Network as Kali
4. Verified connectivity between Kali and Metasploitable

---

## 📝 Attack Chain

### Step 1 — Reconnaissance with Nmap

Scanned the Metasploitable target to discover all open ports and running services:

```bash
nmap -sV [TARGET_IP]
```

`-sV` = Service Version detection — not just which ports are open, but **what software and version** is running on each port.

**Key findings:**

| Port | State | Service | Why It's Vulnerable |
|---|---|---|---|
| 21 | Open | FTP (vsftpd 2.3.4) | Known backdoor vulnerability |
| 22 | Open | SSH (OpenSSH 4.7p1) | Old version, potential exploits |
| 23 | Open | Telnet | Cleartext protocol — no encryption |
| 80 | Open | HTTP (Apache 2.2.8) | Old web server, multiple vulns |
| 139/445 | Open | Samba (3.x) | SMB file sharing — exploitable |
| 512 | Open | exec | Remote execution service |
| **513** | **Open** | **rlogin** | **No authentication — direct root access** |
| 514 | Open | shell | Remote shell service |
| 3306 | Open | MySQL (5.0.51a) | Database exposed to network |

> **Lesson:** A single Nmap scan revealed 9+ exploitable services. In a real pentest, each one is a potential entry point.

---

### Step 2 — Exploiting rlogin (Port 513)

**What is rlogin?**

rlogin (Remote Login) is an old Unix service from the 1980s that allows remote shell access. It was designed for trusted networks where security wasn't a concern.

**Why it's vulnerable:**
- **No encryption** — everything sent in cleartext (like Telnet)
- **Trust-based authentication** — if the server trusts your host, no password required
- Metasploitable trusts all connections by default

**The exploit:**

```bash
rlogin -l root [TARGET_IP]
```

- `-l root` = login as the root user
- No password was required
- **Instant root shell on the target server**

**Result:**
```
root@metasploitable:~#
```

✅ **Full root access gained** — complete control over the target system.

---

### Step 3 — Verifying Access

Once inside the target:

```bash
# Confirm identity
whoami
# Output: root

# Check system info
uname -a
# Output: Linux metasploitable 2.6.24-16-server

# List all users
cat /etc/passwd

# View sensitive files
cat /etc/shadow
```

Having root access means:
- Read/modify any file on the system
- Install backdoors for persistent access
- Pivot to other machines on the network
- Access databases, credentials, and sensitive data

---

## 🔑 Key Concepts Learned

### Server-Side vs Client-Side Attacks

| | Server-Side | Client-Side |
|---|---|---|
| **Target** | The server directly | The user (human) |
| **Requires user action?** | No | Yes (click link, visit page) |
| **Entry point** | Open ports & vulnerable services | Browser, email, social engineering |
| **Example** | Exploiting rlogin, FTP backdoor | MITM + DNS spoofing, phishing |
| **My labs** | Lab 07 (this one) | Labs 03, 04, 06 |

### Why Legacy Services Are Dangerous

| Service | Port | Problem |
|---|---|---|
| rlogin | 513 | Trust-based auth, no encryption |
| Telnet | 23 | No encryption — credentials sent in cleartext |
| FTP | 21 | No encryption — credentials visible in Wireshark |
| rsh (remote shell) | 514 | No authentication on trusted hosts |

> **Defence:** Disable all legacy services. Use SSH (encrypted, key-based auth) instead of Telnet/rlogin/rsh.

### MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| Network Service Scanning | T1046 | Nmap scan to discover open ports |
| Exploitation of Remote Services | T1210 | Exploiting rlogin to gain access |
| Valid Accounts | T1078 | Using root account via trusted host bypass |

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| `nmap -sV` | Port scanning + service version detection |
| `rlogin` | Remote login exploitation |
| Metasploitable 2 | Intentionally vulnerable target server |

---

## 📊 Lab Results

| Test | Technique | Result | Notes |
|---|---|---|---|
| Nmap Service Scan | `nmap -sV` against Metasploitable | ✅ | Discovered 9+ open vulnerable services |
| rlogin Exploitation | `rlogin -l root [TARGET]` | ✅ | Gained root shell — no password required |
| Access Verification | `whoami`, `cat /etc/shadow` | ✅ | Confirmed full root access |

---

## 🧠 Skills Demonstrated

| Skill | Evidence |
|---|---|
| Service Enumeration | Used Nmap to discover and fingerprint running services |
| Vulnerability Identification | Identified rlogin as exploitable due to lack of authentication |
| Remote Exploitation | Gained root access without any credentials |
| Understanding Legacy Protocols | Explained why rlogin, Telnet, and FTP are insecure |
| MITRE ATT&CK Mapping | Mapped techniques to T1046, T1210, T1078 |

---

[← Lab 06: Wireshark Traffic Analysis](../lab-06-wireshark-traffic-analysis/README.md) · [Lab 08: Coming Soon →]()
