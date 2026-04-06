# 🌐 CCNA 200-301 — Networking Labs (Cisco Packet Tracer)

<div align="center">

**Network Fundamentals** · **IP Addressing** · **Routing** · **Switching** · **Security**

[![CCNA](https://img.shields.io/badge/CCNA-200--301-blue?logo=cisco)](https://www.cisco.com/c/en/us/training-events/training-certifications/certifications/associate/ccna.html)
[![Packet Tracer](https://img.shields.io/badge/Tool-Cisco_Packet_Tracer-1ba0d7?logo=cisco)]()
[![Course](https://img.shields.io/badge/Course-Pradeep_Kumar_(YouTube)-red?logo=youtube)](https://www.youtube.com/watch?v=EPeFhfENuXA&list=PL2l13o6BVteMQGXFw7WEovEWoROLGdgwE)
[![Status](https://img.shields.io/badge/Status-Completed-brightgreen)]()

</div>

---

## 🎯 Overview

This section documents all practical networking labs built in **Cisco Packet Tracer** while completing the **CCNA 200-301 Full Course** by Pradeep Kumar (YouTube, Hindi). Each lab demonstrates hands-on implementation of core networking concepts aligned with the official Cisco CCNA 200-301 exam objectives.

**Course:** [CCNA 200-301 Full Course — Pradeep Kumar (YouTube)](https://www.youtube.com/watch?v=EPeFhfENuXA&list=PL2l13o6BVteMQGXFw7WEovEWoROLGdgwE)
**Tool Used:** Cisco Packet Tracer (Cisco's official network simulation software)

---

## 🗺️ Lab Index

| # | Lab | Topic | Key Skill |
|:---:|:---|:---|:---|
| 01 | [OSI & TCP/IP Model](labs/lab-01-osi-tcpip-model.md) | Network Fundamentals | Protocol stack & data encapsulation |
| 02 | [IP Addressing & Subnetting](labs/lab-02-ip-subnetting.md) | IP Addressing | CIDR, subnet masks, host calculation |
| 03 | [VLAN Configuration](labs/lab-03-vlans.md) | Switching | Segmenting networks with VLANs |
| 04 | [Inter-VLAN Routing](labs/lab-04-inter-vlan-routing.md) | Routing + Switching | Router-on-a-stick & L3 switching |
| 05 | [Static Routing](labs/lab-05-static-routing.md) | Routing | Manual route configuration |
| 06 | [Dynamic Routing — OSPF](labs/lab-06-ospf.md) | Routing Protocols | OSPF single-area configuration |
| 07 | [DHCP Configuration](labs/lab-07-dhcp.md) | Network Services | Auto IP assignment for clients |
| 08 | [NAT & PAT](labs/lab-08-nat-pat.md) | Address Translation | Private → Public IP mapping |
| 09 | [Access Control Lists (ACLs)](labs/lab-09-acls.md) | Network Security | Traffic filtering rules |
| 10 | [Spanning Tree Protocol (STP)](labs/lab-10-stp.md) | Switching | Loop prevention in switched networks |

---

## 🏗️ Network Topologies Built

### 1. Basic LAN Topology
```
PC1 ──┐
PC2 ──┼── Switch (S1) ── Router (R1) ── Internet
PC3 ──┘
```

### 2. Multi-VLAN Office Network
```
                    ┌── VLAN 10 (Sales) ── PC1, PC2
Router ── Switch ───┤
                    └── VLAN 20 (IT)    ── PC3, PC4
```

### 3. Multi-Router OSPF Topology
```
R1 (10.0.0.0/8) ── R2 (172.16.0.0/16) ── R3 (192.168.1.0/24)
     |_________________OSPF Area 0___________________|
```

### 4. NAT / PAT Topology
```
Private Network          ISP Router         Internet
192.168.1.0/24 ── Router(NAT) ── 203.0.113.1 ── 8.8.8.8
```

---

## 🧠 Core Concepts Mastered

### Network Fundamentals
| Concept | What Was Practiced |
|:---|:---|
| OSI 7-Layer Model | Mapped real protocols to each layer |
| TCP/IP Model | Compared with OSI, traced packets |
| Ethernet | Frame structure, MAC addressing |
| Cabling | Straight-through vs crossover, fiber vs copper |

### IP Addressing & Subnetting
| Concept | What Was Practiced |
|:---|:---|
| IPv4 Classes | Class A/B/C ranges and default masks |
| CIDR Notation | /24, /26, /30 subnetting |
| Subnetting | Dividing networks into smaller subnets |
| VLSM | Variable Length Subnet Masking |
| IPv6 Basics | EUI-64 addressing, link-local addresses |

### Switching
| Concept | What Was Practiced |
|:---|:---|
| VLANs | Created, named, and assigned ports to VLANs |
| Trunk Ports | Configured 802.1Q trunking between switches |
| STP | Identified root bridge, blocked redundant paths |
| EtherChannel | Bundled multiple links with LACP |

### Routing
| Concept | What Was Practiced |
|:---|:---|
| Static Routes | Configured manual routes between networks |
| Default Routes | Setup 0.0.0.0/0 gateway of last resort |
| RIP v2 | Configured distance-vector routing |
| OSPF | Single-area OSPF with DR/BDR election |
| EIGRP | Cisco's advanced distance-vector protocol |

### Network Services
| Concept | What Was Practiced |
|:---|:---|
| DHCP | Configured router as DHCP server |
| DNS | Configured name resolution |
| NAT Static | One-to-one IP mapping |
| PAT (Overload) | Many-to-one (port-based) NAT |

### Security
| Concept | What Was Practiced |
|:---|:---|
| Standard ACLs | Filter by source IP |
| Extended ACLs | Filter by source, destination, port |
| Port Security | Limit MAC addresses on switch ports |
| SSH | Replaced Telnet with encrypted SSH on routers |
| Password Encryption | Used `service password-encryption` |

---

## 🔗 Relevance to Cybersecurity / Ethical Hacking

Understanding networking at this depth directly supports penetration testing:

| CCNA Topic | Hacking Application |
|:---|:---|
| **Subnetting** | Understanding target network ranges during recon |
| **VLANs** | VLAN hopping attack technique |
| **ARP** | ARP spoofing / poisoning (Lab 03 in this portfolio) |
| **NAT** | Understanding why IPs change between internal/external |
| **ACLs** | Bypassing firewall rules during exploitation |
| **OSPF/Routing** | Route injection attacks on network infrastructure |
| **Spanning Tree** | STP manipulation attacks in switched environments |
| **Port Security** | Understanding what defenders set up vs. how to bypass |

---

## 📚 Reference Materials

- 📹 **Course:** [CCNA 200-301 Full Course — Pradeep Kumar (YouTube)](https://www.youtube.com/watch?v=EPeFhfENuXA&list=PL2l13o6BVteMQGXFw7WEovEWoROLGdgwE)
- 📖 **Official:** [Cisco CCNA 200-301 Exam Topics](https://learningnetwork.cisco.com/s/ccna-exam-topics)
- 🛠️ **Tool:** [Cisco Packet Tracer — Download](https://www.netacad.com/courses/packet-tracer)
- 📗 **Book:** Wendell Odom — *CCNA 200-301 Official Cert Guide*

---

<div align="center">
  <sub>🌐 Labs built in Cisco Packet Tracer for educational purposes. Network configurations follow real-world enterprise best practices.</sub>
</div>
