# Step 2: IP Addressing and Subnetting Basics

## Introduction

**Scenario:** You need to understand IP addressing and subnetting to analyze network traffic, configure security zones, and understand how networks are divided. Subnetting is essential for network segmentation and security.

**What we're learning:**

* IPv4 addressing fundamentals
* Subnet masks and CIDR notation
* Simple subnetting calculations
* Network vs host portions of IP addresses
* Security relevance of subnetting

**Expected Outcomes:**

* ✅ Understand IPv4 addressing
* ✅ Read and interpret subnet masks
* ✅ Understand CIDR notation
* ✅ Perform simple subnetting calculations
* ✅ Identify network and host portions
* ✅ Calculate network & broadcast addresses

---

## Instructions

> **Note:** All commands in this lab are run in the **Killercoda terminal**.

---

## Understanding IP Addresses

### View Your IP Address

```bash
ip addr show | grep "inet " | head -3
```

* Displays IPv4 address and CIDR subnet mask

### IP Address Structure (IPv4)

```text
IP ADDRESS STRUCTURE:
Format: XXX.XXX.XXX.XXX (4 octets, each 0–255)
Example: 192.168.1.100

Each octet = 8 bits (binary):
192 = 11000000
168 = 10101000
1   = 00000001
100 = 01100100

Total: 32 bits (4 bytes)

Two parts:
1. Network portion → identifies the network
2. Host portion → identifies the device
```

### View Subnet Mask (CIDR)

```bash
ip addr show | grep "inet " | grep -o "/[0-9]*" | head -1
```

* Shows CIDR prefix (example: `/24`)

---

## Subnet Masks

### What Is a Subnet Mask?

```text
SUBNET MASK:
Purpose: Separates network bits from host bits
Format: Same as IP (XXX.XXX.XXX.XXX)

Common Masks:
- 255.255.255.0  = /24 (home networks)
- 255.255.0.0    = /16 (medium networks)
- 255.0.0.0      = /8  (large networks)
- 255.255.255.252 = /30 (point-to-point)

Binary Meaning:
- 255 = network bit (1)
- 0   = host bit (0)

Example:
IP:   192.168.1.100
Mask: 255.255.255.0
Network: 192.168.1.0
```

### Subnet Mask in Binary

```text
Subnet Mask: 255.255.255.0

255 = 11111111 (8 network bits)
255 = 11111111 (8 network bits)
255 = 11111111 (8 network bits)
0   = 00000000 (8 host bits)

Total:
- Network bits: 24
- Host bits: 8
- CIDR: /24

Addresses:
- Total: 256
- Usable: 254 (excluding network & broadcast)
```

---

## CIDR Notation

### Understanding CIDR

```text
CIDR (Classless Inter-Domain Routing):
Format: IP_ADDRESS/PREFIX_LENGTH

Example:
192.168.1.0/24

Prefix Length:
- Number of network bits

Common CIDR Blocks:
- /24 = 256 addresses (254 usable)
- /16 = 65,536 addresses
- /8  = 16,777,216 addresses
- /32 = Single host
- /0  = Default route (entire internet)

Benefits:
- Flexible subnet sizes
- Efficient IP allocation
- Replaces classful networking
```

### CIDR Examples

```text
192.168.1.0/24
- Network:   192.168.1.0
- Broadcast: 192.168.1.255
- Usable:    192.168.1.1 – 192.168.1.254

10.0.0.0/8
- Network:   10.0.0.0
- Broadcast: 10.255.255.255
- Usable:    10.0.0.1 – 10.255.255.254
```

---

## Simple Subnetting Examples

### Example 1: /24 Network

```text
Given: 192.168.1.0/24

Network:   192.168.1.0
Broadcast: 192.168.1.255
Usable:    192.168.1.1 – 192.168.1.254
Total usable hosts: 254

Reserved:
- .0   → Network address
- .255 → Broadcast address
```

### Example 2: Smaller Subnet (/28)

```text
Given: 192.168.1.0/28

Host bits: 4
Total addresses: 2^4 = 16
Usable hosts: 14

Network:   192.168.1.0
Broadcast: 192.168.1.15
Usable:    192.168.1.1 – 192.168.1.14

Subnet Mask: 255.255.255.240
```

---

## Calculating Network Address (AND Operation)

```text
IP:   192.168.1.100
Mask: 255.255.255.0

Binary AND:
IP:   11000000.10101000.00000001.01100100
Mask: 11111111.11111111.11111111.00000000
----------------------------------------
Result:11000000.10101000.00000001.00000000

Network Address: 192.168.1.0
```

### Practice Calculation

```text
IP:   10.20.30.45
Mask: 255.255.0.0 (/16)

Network:   10.20.0.0
Broadcast: 10.20.255.255
```

---

## Private vs Public IP Addresses

### Private IP Ranges (RFC 1918)

```text
10.0.0.0/8
- Large organizations

172.16.0.0/12
- Medium networks

192.168.0.0/16
- Home & small offices

Private IPs:
- NOT routable on internet
- Used internally

Public IPs:
- Assigned by ISPs
- Routable globally
```

### Check If Your IP Is Private

```bash
MY_IP=$(hostname -I | awk '{print $1}')
echo "Your IP: $MY_IP"

if [[ $MY_IP =~ ^10\. ]] || \
   [[ $MY_IP =~ ^172\.(1[6-9]|2[0-9]|3[0-1])\. ]] || \
   [[ $MY_IP =~ ^192\.168\. ]]; then
  echo "This is a PRIVATE IP address"
else
  echo "This is a PUBLIC IP address (or special range)"
fi
```

---

## Special IP Addresses

```text
127.0.0.1 (localhost):
- Your own system
- Used for testing

0.0.0.0:
- Any interface
- Default route

255.255.255.255:
- Broadcast address

169.254.x.x (APIPA):
- Assigned when DHCP fails
- Link-local
```

### Test Localhost

```bash
ping -c 2 127.0.0.1
```

---

## Subnetting for Security

```text
Why Subnetting Matters:

Network Segmentation:
- Split network into zones
- Isolate sensitive systems

Security Benefits:
- Reduce attack surface
- Limit lateral movement
- Apply per-subnet firewall rules

Example:
- Web Tier: 10.0.1.0/24
- App Tier: 10.0.2.0/24
- DB Tier:  10.0.3.0/24

DB subnet blocks internet access
```

---

## Subnetting Practice

### Practice Example 1

```text
Network: 192.168.1.0/24
Need: 4 subnets with ~50 hosts

Use: /26 (64 addresses, 62 hosts)

Subnets:
1. 192.168.1.0/26
2. 192.168.1.64/26
3. 192.168.1.128/26
4. 192.168.1.192/26
```

### Practice Example 2

```text
Given: 10.0.0.0/8
Need: /16 subnets

Subnet bits: 8
Total subnets: 256

Examples:
10.0.0.0/16
10.1.0.0/16
...
10.255.0.0/16
```

---

## VLSM (Variable Length Subnet Masking)

```text
VLSM:
- Different subnet sizes in same network
- Efficient IP usage

Example:
192.168.0.0/16

- 192.168.1.0/24   (Office)
- 192.168.2.0/26   (Servers)
- 192.168.2.64/30  (Router link)
- 192.168.3.0/24   (Warehouse)
```

---

## Key Learning Points

* IPv4 = 32-bit address
* Subnet mask separates network & host
* CIDR = IP/PREFIX
* Network address = all host bits 0
* Broadcast address = all host bits 1
* Subnetting enables segmentation
* VLSM improves efficiency

---

## Security Relevance

* Creates security zones
* Enables firewall rule design
* Limits attack spread
* Improves traffic analysis
* Essential for incident response

---

## Summary

You’ve learned:

* IP structure & subnet masks
* CIDR notation
* Subnetting calculations
* Private vs public IPs
* Security-focused subnet design

➡️ These skills are critical for networking, cloud, and security engineering.
