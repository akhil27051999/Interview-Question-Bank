# Step 1: Network Fundamentals

## Introduction

**Scenario:** You need to understand how networks work to investigate security incidents and analyze network traffic. Network models provide a framework for understanding how data moves through networks.

**What we're learning:**

* Network models (OSI & TCP/IP)
* Network types
* Core networking concepts required for security engineers

**Expected Outcomes:**

* ✅ Understand OSI model (7 layers)
* ✅ Understand TCP/IP model (4 layers)
* ✅ Know different network types (LAN, WAN, VPN)
* ✅ Understand basic networking terminology
* ✅ Learn how data moves through networks

---

## Instructions

> **Note:** All commands in this lab are run in the **Killercoda terminal**.

---

## Understanding Networks

### Check Network Connectivity

```bash
ping -c 4 8.8.8.8
```

* Tests basic network connectivity
* If this works, your system is connected to a network

### View Network Interfaces

```bash
ip addr show | head -20
# OR
ifconfig | head -15
```

* Displays network interfaces (physical or virtual)

### What Is a Network?

```text
NETWORK DEFINITION:
A network is two or more devices connected together to share information.

Key Components:
- Devices (computers, servers, routers, switches)
- Cables or wireless connections
- Protocols (rules for communication)
- IP addresses (unique identifiers)

Think of it like a postal system:
- Devices = houses
- Network = streets and roads
- IP addresses = street addresses
- Data packets = letters/packages
```

---

## OSI Model (7 Layers)

### OSI Layers Explained

```text
OSI MODEL (7 LAYERS):

Layer 7 - Application
- User interfaces (web browsers, email clients)

Layer 6 - Presentation
- Data encryption, compression, translation

Layer 5 - Session
- Establishes and manages connections

Layer 4 - Transport
- TCP / UDP (reliable vs unreliable delivery)

Layer 3 - Network
- IP addressing and routing

Layer 2 - Data Link
- MAC addresses and frame delivery

Layer 1 - Physical
- Cables, wireless signals, hardware

Memory Trick:
"Please Do Not Throw Sausage Pizza Away"
7 6 5 4 3 2 1
```

### Protocols by OSI Layer

```text
Layer 7 (Application): HTTP, HTTPS, DNS, SMTP, FTP, SSH
Layer 4 (Transport): TCP, UDP
Layer 3 (Network): IP, ICMP
Layer 2 (Data Link): Ethernet, MAC
Layer 1 (Physical): Cables, Wi-Fi signals
```

---

## TCP/IP Model (4 Layers)

### TCP/IP Layers

```text
TCP/IP MODEL (4 LAYERS):

Layer 4 - Application
- HTTP, DNS, SSH, FTP (OSI Layers 5–7)

Layer 3 - Transport
- TCP, UDP (same as OSI Layer 4)

Layer 2 - Internet
- IP, ICMP (same as OSI Layer 3)

Layer 1 - Network Access
- Ethernet, MAC (OSI Layers 1–2)

Note:
TCP/IP is simpler and used by the internet in real-world networking.
```

### OSI vs TCP/IP Comparison

```text
OSI Model (Theoretical):
- 7 layers
- More detailed
- Used for learning and troubleshooting

TCP/IP Model (Practical):
- 4 layers
- Used by the internet
- Easier to implement

Troubleshooting Guide:
- App issues → Layer 7 / TCP-IP Layer 4
- Network connectivity → Layer 3
- Physical problems → Layer 1
```

---

## Network Types

```text
LAN (Local Area Network):
- Small area (home, office)
- High speed

WAN (Wide Area Network):
- Large area (cities, countries)
- Example: Internet

VPN (Virtual Private Network):
- Secure, encrypted tunnel
- Used for remote access

MAN (Metropolitan Area Network):
- City-wide networks

PAN (Personal Area Network):
- Very small range
- Example: Bluetooth, USB
```

### View Routing Table

```bash
echo "Checking network configuration..."
ip route show | head -5
# OR
route -n | head -5
```

---

## Basic Networking Concepts

### Packets vs Frames

```text
Packets:
- Layer 3 (Network)
- Contains IP addresses
- Routed across networks

Frames:
- Layer 2 (Data Link)
- Contains MAC addresses
- Delivered within local networks

Encapsulation:
Application Data
→ TCP Segment
→ IP Packet
→ Ethernet Frame
```

### MAC Addresses

```text
MAC Address (Layer 2):
- 48-bit hardware identifier
- Format: XX:XX:XX:XX:XX:XX
- Example: 00:1B:44:11:3A:B7

Security Uses:
- Device identification
- Detect MAC spoofing
```

#### View MAC Address

```bash
ip link show | grep -i ether | head -3
# OR
ifconfig | grep -i ether | head -3
```

### IP Addresses

```text
IPv4:
- 32-bit address
- Example: 192.168.1.1

IPv6:
- 128-bit address
- Example: 2001:db8::1

Private IPv4 Ranges:
- 10.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16
```

#### View IP Address

```bash
hostname -I
# OR
ip addr show | grep "inet " | head -3
```

---

## Network Devices

```text
Router:
- Connects different networks
- Layer 3 (IP-based)

Switch:
- Connects devices in same network
- Layer 2 (MAC-based)

Hub:
- Broadcasts traffic
- Layer 1 (legacy)

Firewall:
- Filters traffic
- Protects networks
```

### Check Default Gateway

```bash
ip route | grep default
# OR
route -n | grep "^0.0.0.0"
```

---

## Network Topologies

```text
Star Topology:
- Central switch/hub
- Most common

Bus Topology:
- Single cable
- Legacy

Ring Topology:
- Circular data path

Mesh Topology:
- High redundancy
- Used in critical systems
```

---

## Key Learning Points

* Network = interconnected devices
* OSI = conceptual model
* TCP/IP = real-world model
* Packets (IP) vs Frames (MAC)
* Routers route, switches switch

---

## Security Relevance

* Identify attack layers
* Analyze network traffic
* Design secure architectures
* Track devices
* Detect protocol-based attacks

---

## Summary

You now understand:

* OSI & TCP/IP models
* Network types
* Core networking components
* Devices and topologies

➡️ These fundamentals are critical for networking, security, and cloud engineering.
