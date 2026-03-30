# 🧪 Lab 01: Penetration Testing Environment Setup

<div align="center">

**Category:** Lab Infrastructure &nbsp;|&nbsp; **Difficulty:** Beginner &nbsp;|&nbsp; **Status:** ✅ Completed

</div>

---

## 📋 Lab Overview

| Field | Details |
|---|---|
| **Objective** | Build a fully isolated, repeatable penetration testing lab environment |
| **Attacker Machine** | Kali Linux (Virtual Machine) |
| **Target Machine** | Windows 10/11 (Virtual Machine) |
| **Hypervisor** | VirtualBox / VMware |
| **Network Mode** | NAT Network (inter-VM communication, isolated from host) |
| **Time Required** | ~2-3 hours |

### Why This Lab Matters
Every penetration test begins with a well-configured environment. A misconfigured lab can lead to:
- Attacks leaking onto production networks (legal risk)
- VMs unable to communicate (wasted troubleshooting time)
- Inconsistent results across tests

This lab establishes the **foundation** for all subsequent offensive security exercises.

---

## 🏗️ Architecture Diagram

```
┌─────────────────────────────────────────────────────┐
│                    HOST MACHINE                      │
│                  (Your real PC)                      │
│                                                     │
│  ┌─────────────────────────────────────────────┐    │
│  │           VirtualBox / VMware                │    │
│  │                                             │    │
│  │   ┌─────────────┐     ┌─────────────┐      │    │
│  │   │  KALI LINUX │     │  WINDOWS VM │      │    │
│  │   │  (Attacker) │     │  (Target)   │      │    │
│  │   │             │     │             │      │    │
│  │   │ 10.0.2.x    │     │ 10.0.2.x   │      │    │
│  │   └──────┬──────┘     └──────┬──────┘      │    │
│  │          │                   │              │    │
│  │   ┌──────┴───────────────────┴──────┐      │    │
│  │   │      NAT Network (10.0.2.0/24) │      │    │
│  │   │      Virtual Router: 10.0.2.1  │      │    │
│  │   └─────────────────────────────────┘      │    │
│  │          ❌ Isolated from Host              │    │
│  └─────────────────────────────────────────────┘    │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 📝 Step-by-Step Walkthrough

### Step 1: Install the Hypervisor

The hypervisor is the software that lets you run virtual machines on your real computer.

**Using VirtualBox (Free):**
1. Download VirtualBox from [virtualbox.org](https://www.virtualbox.org/wiki/Downloads)
2. Select the installer for your host OS (Windows/macOS/Linux)
3. Run the installer → Accept defaults → Complete installation
4. Launch VirtualBox Manager

**Verification:**
```
VirtualBox Manager should open with an empty VM list.
Version should be 7.x or later for best compatibility.
```

---

### Step 2: Download the Operating System Images

You need two ISOs:

| Machine | Download Source | File Type |
|---|---|---|
| **Kali Linux** | [kali.org/get-kali](https://www.kali.org/get-kali/#kali-virtual-machines) | Pre-built `.ova` (recommended) or `.iso` |
| **Windows 10/11** | [microsoft.com](https://www.microsoft.com/software-download/windows10ISO) | `.iso` file |

> 💡 **Pro Tip:** Download the Kali `.ova` file — it's a pre-configured VM image that saves 30+ minutes of manual installation.

---

### Step 3: Deploy Kali Linux (Attacker Machine)

**If using the `.ova` file (recommended):**
1. In VirtualBox → `File` → `Import Appliance`
2. Browse to the downloaded `.ova` file → Click `Next`
3. Review settings (RAM: minimum 2GB, recommended 4GB) → Click `Import`
4. Wait for import to complete (~5-10 minutes)

**If using the `.iso` file:**
1. Click `New` in VirtualBox
2. Name: `Kali-Attacker` | Type: `Linux` | Version: `Debian (64-bit)`
3. Memory: `4096 MB` (4GB)
4. Create virtual hard disk: `VDI`, `Dynamically allocated`, `80 GB`
5. Go to `Settings` → `Storage` → Click the empty CD icon → Choose your Kali `.iso`
6. Start the VM and follow the graphical installer

**Default Credentials (for .ova):**
```
Username: kali
Password: kali
```

**Post-Install Verification:**
```bash
# Check Kali version
cat /etc/os-release

# Verify essential tools are installed
which nmap msfconsole aircrack-ng wireshark

# Update package repositories
sudo apt update && sudo apt upgrade -y
```

---

### Step 4: Deploy Windows VM (Target Machine)

1. In VirtualBox → Click `New`
2. Configure the VM:
   - **Name:** `Windows-Target`
   - **Type:** `Microsoft Windows`
   - **Version:** `Windows 10 (64-bit)`
   - **Memory:** `4096 MB` (4GB)
   - **Hard Disk:** `Create a virtual hard disk` → `VDI` → `50 GB`
3. Go to `Settings` → `Storage` → Attach the Windows `.iso` to the optical drive
4. Start the VM → Follow the Windows installer
5. Select `I don't have a product key` (evaluation is sufficient for labs)
6. Choose `Windows 10 Pro` → Custom installation → Complete setup

**Post-Install — Disable Security Features (for lab only!):**

> ⚠️ **WARNING:** Only disable security on isolated lab VMs. NEVER do this on a real machine.

```
1. Windows Defender → Virus & threat protection → Manage settings → Turn OFF Real-time protection
2. Windows Firewall → Turn OFF for Private and Public networks
3. (Optional) Disable Windows Update to prevent patches from fixing vulnerabilities
```

---

### Step 5: Configure NAT Network (Critical Step)

This is where most beginners fail. You need **NAT Network**, NOT regular **NAT**.

| Mode | VMs See Each Other? | Internet Access? | Isolated from Host? |
|---|---|---|---|
| **NAT** (default) | ❌ No | ✅ Yes | ✅ Yes |
| **NAT Network** ✅ | ✅ Yes | ✅ Yes | ✅ Yes |
| **Bridged** | ✅ Yes | ✅ Yes | ❌ No (dangerous!) |

**Creating the NAT Network:**

1. In VirtualBox → `File` → `Preferences` → `Network`
   - (VirtualBox 7.x: `File` → `Tools` → `Network Manager` → `NAT Networks`)
2. Click the **+** icon to add a new NAT Network
3. Configure:
   - **Name:** `PenTestLab`
   - **Network CIDR:** `10.0.2.0/24`
   - **Enable DHCP:** ✅ Checked
4. Click `OK`

**Assigning VMs to the NAT Network:**

For **each VM** (Kali + Windows):
1. Right-click VM → `Settings` → `Network`
2. Adapter 1:
   - **Attached to:** `NAT Network`
   - **Name:** `PenTestLab`
3. Click `OK`

---

### Step 6: Verify Inter-VM Connectivity

Start both VMs and verify they can communicate.

**On Kali Linux — Find your IP:**
```bash
# Check Kali's IP address
ip addr show
# OR
ifconfig

# Expected output: 10.0.2.x (e.g., 10.0.2.5)
```

**On Windows — Find your IP:**
```cmd
ipconfig

REM Expected output: 10.0.2.x (e.g., 10.0.2.15)
```

**Ping Test — From Kali to Windows:**
```bash
ping -c 4 <WINDOWS-IP>

# Expected: 4 packets transmitted, 4 received, 0% packet loss
```

**Ping Test — From Windows to Kali:**
```cmd
ping <KALI-IP>

REM Expected: Reply from 10.0.2.x: bytes=32 time<1ms TTL=64
```

> ⚠️ If ping fails from Windows → Kali, Windows Firewall may be blocking ICMP. Disable it (Step 4).

---

### Step 7: Understand IP vs MAC Addressing

Before proceeding to attacks, solidify this foundational knowledge:

| Concept | IP Address | MAC Address |
|---|---|---|
| **Layer** | Layer 3 (Network) | Layer 2 (Data Link) |
| **Format** | `192.168.1.10` | `AA:BB:CC:DD:EE:FF` |
| **Assigned by** | DHCP / Manual | Hardware manufacturer |
| **Changes?** | Yes (can be reassigned) | Permanent (but can be spoofed!) |
| **Scope** | Can cross networks (routable) | Local network only |
| **Used for** | Routing between networks | Identifying devices on same network |

**Lab Verification — View both addresses:**
```bash
# On Kali: View IP and MAC
ip addr show eth0

# Look for:
#   inet 10.0.2.5/24       ← This is your IP
#   link/ether 08:00:27:... ← This is your MAC

# On Kali: View the ARP table (IP ↔ MAC mappings)
arp -a
```

---

## 📊 Lab Results

| Field | Value |
|---|---|
| **Kali IP** | `10.0.2.x` |
| **Windows VM IP** | `10.0.2.x` |
| **Virtual Router IP** | `10.0.2.1` |
| **Connectivity Verified** | ✅ |
| **Network Mode** | NAT Network |

---

## 🧠 Key Takeaways

1. **NAT ≠ NAT Network** — Regular NAT isolates each VM; NAT Network lets them talk to each other
2. **Always disable Windows Defender & Firewall** on target VMs (lab only!)
3. **Kali `.ova` import** saves significant setup time vs manual ISO install
4. **Verify connectivity before attacking** — `ping` is your first diagnostic tool
5. **Document your IPs** — they may change after VM restarts (DHCP)

---

## 🔗 Next Lab
➡️ [Lab 02: Network Hacking — Pre-Connection Attacks](../lab-02-network-pre-connection/README.md)
