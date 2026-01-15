# 1. Network Fundamentals

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

## Practical Scenario

```sh

# --------------- Check network connectivity ---------------
# ping tests network connectivity - if this works, your device is connected to a network.

ubuntu:~$ ping -c 2 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=111 time=0.921 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=111 time=0.882 ms

--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.882/0.901/0.921/0.019 ms

# --------------- View network interfaces ---------------
# Shows network interfaces - physical or virtual connections to networks.

ubuntu:~$ ip addr show | head -20 || ifconfig | head -15
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 56:61:ab:c6:10:29 brd ff:ff:ff:ff:ff:ff
    inet 172.30.1.2/24 brd 172.30.1.255 scope global dynamic noprefixroute enp1s0
       valid_lft 86313078sec preferred_lft 75523878sec
    inet6 fe80::70aa:8e2a:ea46:e544/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1454 qdisc noqueue state DOWN group default 
    link/ether e6:dd:d0:94:44:c6 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever

# --------------- Check your network type ---------------
# Shows routing table - helps understand network topology

ubuntu:~$ ip route show | head -5 || route -n | head -5
default via 172.30.1.1 dev enp1s0 proto dhcp src 172.30.1.2 metric 1002 mtu 1500 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
172.30.1.0/24 dev enp1s0 proto dhcp scope link src 172.30.1.2 metric 1002 mtu 1500 

# --------------- Understand MAC addresses ---------------
# Shows MAC address of network interfaces

ubuntu:~$ ip link show | grep -i "ether" | head -3 || ifconfig | grep -i "ether" | head -3
    link/ether 56:61:ab:c6:10:29 brd ff:ff:ff:ff:ff:ff
    link/ether e6:dd:d0:94:44:c6 brd ff:ff:ff:ff:ff:ff

# --------------- View your IP address ---------------
# Shows IP address(es) of your system

ubuntu:~$ hostname -I 2>/dev/null || ip addr show | grep "inet " | head -3
172.30.1.2 172.17.0.1

# --------------- Check default gateway (router) ---------------
# Shows default gateway - the router that connects you to other networks

ubuntu:~$ ip route | grep default || route -n | grep "^0.0.0.0"
default via 172.30.1.1 dev enp1s0 proto dhcp src 172.30.1.2 metric 1002 mtu 1500 
```
