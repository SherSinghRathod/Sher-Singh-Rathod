# 🧪 Lab 06: Wireshark — Deep Packet Inspection & Network Traffic Analysis

<div align="center">

**Category:** Network Traffic Analysis & Forensics &nbsp;|&nbsp; **Difficulty:** Intermediate &nbsp;|&nbsp; **Status:** ✅ Completed

</div>

---

## 📋 Lab Overview

| Field | Details |
|---|---|
| **Objective** | Use Wireshark to capture, filter, and analyze network traffic — identify sensitive data, profile victim behaviour, and extract credentials from unencrypted protocols |
| **Attacker Machine** | Kali Linux VM (`10.0.2.x`) — in MITM position via ARP spoofing |
| **Target Machine** | Windows VM (`10.0.2.x`) |
| **Key Tools** | `wireshark`, `arpspoof`, `bettercap` |
| **MITRE ATT&CK** | T1040 (Network Sniffing), T1557 (Adversary-in-the-Middle), T1083 (File & Directory Discovery) |
| **Prerequisite** | Lab 03 completed (ARP Spoofing + MITM position established) |

### Why This Lab Matters
Wireshark is the industry-standard tool for network forensics and traffic analysis. Whether you're a **penetration tester** proving that unencrypted traffic is dangerous, or a **SOC analyst** investigating a suspicious host, Wireshark gives you full visibility into what's happening on the wire. This lab covers the core skills needed to rapidly extract intelligence from captured traffic.

---

## 🏗️ Environment Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    LAB NETWORK (NAT Network)                     │
│                                                                   │
│  ┌──────────────┐     ARP Spoof      ┌──────────────────────┐   │
│  │  KALI LINUX  │◄───────────────────►│   WINDOWS TARGET     │   │
│  │  (Attacker)  │                    │   (Victim)           │   │
│  │  10.0.2.x    │                    │   10.0.2.15          │   │
│  │              │    All traffic     │                      │   │
│  │  Wireshark   │◄───flows through───│  Browsing HTTP sites │   │
│  │  running on  │                    │  Submitting forms    │   │
│  │  eth0        │                    │  Making DNS queries  │   │
│  └──────┬───────┘                    └──────────────────────┘   │
│         │                                                         │
│         ▼                                                         │
│  ┌──────────────┐                                                 │
│  │   GATEWAY    │──────────────────────────────► Internet        │
│  │  10.0.2.1    │                                                 │
│  └──────────────┘                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📝 Step-by-Step Walkthrough

### Part A: Setup — Establishing MITM Position for Capture

Before Wireshark can see the victim's traffic, we need to be in the path.

#### Step 1: Enable IP Forwarding (so victim doesn't lose internet)

```bash
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
# Output: 1 ✅
```

#### Step 2: ARP Spoof the Victim (both directions)

```bash
# Terminal 1: Poison victim's ARP cache (pretend to be the gateway)
sudo arpspoof -i eth0 -t 10.0.2.15 10.0.2.1

# Terminal 2: Poison gateway's ARP cache (pretend to be the victim)
sudo arpspoof -i eth0 -t 10.0.2.1 10.0.2.15
```

> ✅ Both ARP spoof terminals running = all victim traffic now flows through Kali.

---

### Part B: Wireshark — Capture & Interface Setup

#### Step 3: Launch Wireshark

```bash
sudo wireshark &
```

**Inside Wireshark:**
1. Select interface: **`eth0`** (the interface connected to the lab network)
2. Click the **blue shark fin** (▶) to start capture
3. Traffic immediately starts flowing — victim's traffic is visible

---

### Part C: Mastering Display Filters

Wireshark captures **everything** — the skill is filtering to find what matters. These are the most important filters for a penetration tester.

#### 🔵 Protocol Filters (Broad Scope)

```
http          → Show all HTTP traffic (unencrypted web traffic)
dns           → Show all DNS queries (what sites victim is visiting)
tcp           → All TCP connections
arp           → ARP packets (verify your poisoning is working)
icmp          → Ping traffic
ftp           → FTP file transfers (often cleartext)
telnet        → Telnet sessions (completely cleartext)
```

#### 🔴 Credential Extraction Filters (High Value Targets)

```
http.request.method == "POST"                    → Form submissions (logins!)
http contains "password"                          → Packets with "password" string
http contains "pass"                              → Catch variants like "passwd"
http contains "login"                             → Login-related packets
http contains "user"                              → Username fields
http.request.method == "POST" && http contains "pass"   → POST with password field
```

#### 🟡 Session & Cookie Analysis

```
http.cookie                  → Packets containing session cookies
http.set-cookie              → Server setting a cookie (victim just logged in!)
http.request.uri contains "admin"    → Admin panel access attempts
http.response.code == 200            → Successful HTTP responses only
http.response.code == 302            → Redirects (login success redirects)
```

#### 🟣 DNS Profiling — What Sites Is the Victim Visiting?

```
dns                              → All DNS queries
dns.qry.name contains "google"  → Google queries
dns.qry.name contains "bank"    → Banking sites
dns.qry.name contains "facebook"→ Social media
dns.qry.type == 1                → A records (IPv4 hostname lookups)
dns.flags.response == 0          → DNS queries only (not responses)
```

#### ⚪ Source/Destination Filtering

```
ip.src == 10.0.2.15              → All traffic FROM the victim
ip.dst == 10.0.2.15              → All traffic TO the victim
ip.addr == 10.0.2.15             → All traffic involving the victim
tcp.port == 80                   → HTTP only
tcp.port == 443                  → HTTPS only (encrypted — gibberish without MITM)
tcp.port == 21                   → FTP
tcp.port == 23                   → Telnet
```

#### 🔍 String Search (Nuclear Option)

```
frame contains "admin"           → Find "admin" in ANY packet
frame contains "password"        → Find "password" in ANY packet
tcp contains "secret"            → Find "secret" in any TCP stream
```

---

### Part D: Extracting Credentials from a POST Request

This is the core skill — finding login credentials in captured traffic.

#### Step 4: Apply the POST filter

```
http.request.method == "POST"
```

#### Step 5: Find a Login Packet

1. Look in the **Info** column for `POST /login.php` or similar
2. Click on that packet

#### Step 6: Expand the HTML Form Data

In the **Packet Details** pane (middle panel):
```
▼ Hypertext Transfer Protocol
  ▼ HTML Form URL Encoded: application/x-www-form-urlencoded
      Form item: "username" = "victim_user"     ← 🎯 USERNAME
      Form item: "password" = "secretpass123"   ← 🎯 PASSWORD
```

**Alternative — Follow the Full HTTP Stream:**
```
1. Right-click the POST packet
2. Follow → HTTP Stream
3. In the stream window, you'll see the raw conversation:

   POST /login.php HTTP/1.1
   Host: testphp.vulnweb.com
   Content-Type: application/x-www-form-urlencoded

   username=victim_user&password=secretpass123   ← Credentials in plaintext!
```

---

### Part E: DNS Profiling — Building a Victim's Web History

Even without reading HTTP content, DNS queries reveal everything.

#### Step 7: Apply DNS filter and analyze

```
dns && dns.flags.response == 0
```

What you can determine:
- **`dns.qry.name == "mail.google.com"`** → Victim uses Gmail
- **`dns.qry.name == "api.bank.com"`** → Victim banking online
- **`dns.qry.name == "netflix.com"`** → Entertainment habits
- **`dns.qry.name == "vpn.company.com"`** → Work VPN usage patterns

> 💡 This DNS traffic profiling is extremely stealthy — the victim has no indication they're being observed.

---

### Part F: Analysing FTP / Telnet (Fully Cleartext Protocols)

These legacy protocols send **everything** in plaintext — including credentials.

#### FTP Analysis
```
# Wireshark filter:
ftp

# In packet details, you'll see:
Request: USER john_doe        ← FTP username
Request: PASS hunter2         ← FTP password IN PLAINTEXT
```

#### Telnet Analysis
```
# Wireshark filter:
telnet

# Follow TCP Stream → entire terminal session visible
# Including commands run, files accessed, passwords typed
```

---

## 📊 Lab Results

| Exercise | Filter / Technique | Result |
|---|---|---|
| HTTP POST Credential Capture | `http.request.method == "POST"` | ✅ Username & password extracted from form submission |
| DNS Site Profiling | `dns.qry.name` analysis | ✅ Built a list of domains visited by victim |
| Follow HTTP Stream | Right-click → Follow → HTTP Stream | ✅ Read complete login request including credentials |
| Cookie Capture | `http.set-cookie` filter | ✅ Session cookie captured (could be used for session hijacking) |
| ARP Verification | `arp` filter | ✅ Confirmed bidirectional ARP spoofing is active |
| String Search | `frame contains "password"` | ✅ Located credential packets without knowing exact URL |

---

## ⚠️ Note on Bettercap GUI

> The Bettercap GUI section of this module was skipped due to known performance and stability issues with the web UI in newer versions. Bettercap CLI (covered in Lab 04) provides full equivalent functionality and is the **industry-standard** approach used by professional penetration testers. All MITM capabilities were exercised via CLI throughout this lab.

---

## 🛡️ Defensive Recommendations

| What Was Captured | Why It's Possible | How to Prevent It |
|---|---|---|
| **HTTP form credentials** | No encryption on HTTP | Use HTTPS with HSTS headers on all login pages |
| **DNS queries** | DNS is unencrypted by default | Use DNS-over-HTTPS (DoH) or DNS-over-TLS (DoT) |
| **Session cookies** | Sent in HTTP headers | `Secure` + `HttpOnly` cookie flags; HTTPS only |
| **FTP/Telnet credentials** | Legacy protocols = no encryption | Replace with SFTP/SSH; disable Telnet/FTP |
| **Browsing history via DNS** | DNS unencrypted | VPN + DoH to encrypt all DNS queries |

---

## 🧠 Key Takeaways

1. **Wireshark filters are essential** — Without filters, you drown in noise. Knowing `POST`, `contains`, `dns.qry.name` lets you find intelligence in seconds
2. **HTTP is completely transparent** — Any network-path attacker (or ISP) can read all HTTP traffic, including credentials and cookies
3. **DNS leaks your history even over HTTPS** — Even when content is encrypted, DNS queries reveal every site visited
4. **Legacy protocols (FTP, Telnet) are catastrophic** — Never use them on any network you don't fully control
5. **Bettercap CLI > GUI** — The CLI is faster, more stable, scriptable, and the standard approach in professional engagements

---

## 📊 Skills Demonstrated

| Skill | MITRE ATT&CK ID | Real-World Application |
|---|---|---|
| Network packet capture | T1040 | Capturing traffic on a compromised segment during a pen test |
| Credential extraction from HTTP | T1557 | Demonstrating HTTP risk to clients |
| DNS traffic analysis | T1040 | SOC analyst investigating suspicious DNS queries |
| Session cookie capture | T1185 | Web application penetration testing |
| Protocol analysis (FTP/Telnet) | T1040 | Legacy system security auditing |

---

## 🔗 Navigation
⬅️ [Lab 05: SIEM Home Lab — Splunk](../lab-05-siem-splunk-home-lab/README.md) &nbsp;|&nbsp; ➡️ Lab 07: Gaining Access — System Hacking *(Coming Soon)*
