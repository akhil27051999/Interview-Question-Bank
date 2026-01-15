# Step 3: Network Commands and Configuration

## Introduction

**Scenario:** You need to configure network settings, view routing tables, inspect ARP tables, and understand how DHCP and DNS work. These commands are essential for network troubleshooting and security investigations.

**What we're learning:**

* Network interface inspection and configuration
* Routing tables and default gateway
* ARP tables and their security impact
* DHCP and DNS basics
* Hosts file and local name resolution

**Expected Outcomes:**

* ✅ View and configure network interfaces
* ✅ Understand routing tables and default gateway
* ✅ View and understand ARP table
* ✅ Understand DHCP configuration
* ✅ Configure and test DNS
* ✅ Use hosts file for local name resolution

---

## Instructions

> **Note:** All commands in this lab are run in the **Killercoda terminal**.

---

## Environment Setup

### Create Practice Directory

```bash
RANDOM_3CHAR=$(openssl rand -hex 3 | cut -c1-3)
PRACTICE_DIR="$HOME/peachycloudsecurity-network-${RANDOM_3CHAR}"
mkdir -p "$PRACTICE_DIR"
cd "$PRACTICE_DIR"
echo "Practice directory: $PRACTICE_DIR"
```

---

## Network Interface Commands

### View Network Interfaces (Modern)

```bash
ip addr show
```

* Modern Linux networking command

### View Interfaces (Brief)

```bash
ip -br addr show
```

* Human‑readable summary

### View Interface Statistics

```bash
ip -s link show
```

* Shows packets, bytes, and errors

### View Interfaces (Legacy)

```bash
ifconfig 2>/dev/null | head -20 || echo "ifconfig not available (use ip)"
```

### View Default Route Interface

```bash
ip addr show $(ip route | grep default | awk '{print $5}' | head -1)
```

* Displays the interface used for outbound traffic

---

## Routing Commands

### View Routing Table

```bash
ip route show
```

### View Default Route

```bash
ip route | grep default || route -n | grep "^0.0.0.0"
```

### Routing Table Explanation

```text
Default Route (0.0.0.0/0):
- Gateway to all external networks

Local Routes:
- Directly connected subnets

Loopback Route:
- 127.0.0.0/8 (localhost only)
```

### View All Routing Tables

```bash
ip route show table all | head -10
```

### Test Routing Path

```bash
traceroute -m 5 8.8.8.8 2>/dev/null || tracepath 8.8.8.8 2>/dev/null | head -10
```

---

## ARP (Address Resolution Protocol)

### View ARP Table

```bash
ip neigh show || arp -a
```

### ARP Explained

```text
ARP maps IP addresses to MAC addresses (Layer 2).

Process:
1. Device asks: Who has IP X?
2. Owner responds with MAC
3. Mapping stored in ARP cache

Security Risk:
- ARP spoofing enables MITM attacks
```

### ARP Table States

```text
REACHABLE – Valid entry
STALE     – Might be outdated
DELAY     – Being verified
FAILED    – Invalid entry
```

### Flush ARP Cache (Reference)

```text
sudo ip neigh flush all
```

---

## DHCP (Dynamic Host Configuration Protocol)

### View DHCP Client Config

```bash
cat /etc/dhcp/dhclient.conf 2>/dev/null | head -20 || echo "DHCP config location varies"
```

### DHCP Explained

```text
DHCP assigns:
- IP address
- Subnet mask
- Gateway
- DNS servers

DORA Process:
1. Discover
2. Offer
3. Request
4. Acknowledge

Security Risk:
- Rogue DHCP servers
```

### View DHCP Lease

```bash
cat /var/lib/dhcp/dhclient.leases 2>/dev/null | head -20 || echo "Lease file varies"
```

### View Network Configuration

```bash
cat /etc/network/interfaces 2>/dev/null || \
cat /etc/netplan/*.yaml 2>/dev/null | head -20 || \
echo "Network config varies by distro"
```

---

## DNS (Domain Name System)

### Test DNS Resolution

```bash
nslookup google.com 2>/dev/null || host google.com
```

### View DNS Servers

```bash
cat /etc/resolv.conf
```

### DNS Explained

```text
DNS resolves names to IPs.

Resolution Order:
1. /etc/hosts
2. DNS server

Security Risk:
- DNS spoofing / poisoning
```

### Multiple DNS Tests

```bash
ubuntu:~$ vi script.sh
ubuntu:~$ cat script.sh 
#!/bin/bash

echo "Testing DNS:"
for domain in google.com github.com; do
  host "$domain" 2>/dev/null | head -1 || nslookup "$domain" | head -1
done

ubuntu:~$ chmod +x script.sh 
ubuntu:~$ ./script.sh 
Testing DNS:
google.com has address 74.125.200.139
github.com has address 20.205.243.166
```

### Query Specific DNS Server

```bash
dig @8.8.8.8 google.com +short 2>/dev/null || host google.com 8.8.8.8
```

---

## Hosts File

### View Hosts File

```bash
cat /etc/hosts
```

### Hosts File Explained

```text
- Checked before DNS
- Used for overrides and blocking

Example:
127.0.0.1 malicious-site.com
```

### Backup & Modify Hosts File

```bash
sudo cp /etc/hosts /etc/hosts.backup.$(date +%s) 2>/dev/null || echo "Backup requires sudo"

echo "To add entry:"
echo "127.0.0.1 test.local" | sudo tee -a /etc/hosts
```

---

## Network Statistics

### View Statistics

```bash
ss -s 2>/dev/null || netstat -s 2>/dev/null | head -20
```

### View Active Connections

```bash
ss -tunp 2>/dev/null | head -10 || netstat -tunp | head -10
```

### Connection States Count

```bash
ss -ant | awk '{print $1}' | sort | uniq -c
```

---

## Practical Scripts

### Network Information Script

```bash
cat > network-info.sh << 'EOF'
#!/bin/bash
echo "=== Network Information ==="
echo "1. IP Addresses:"; ip -br addr show
echo "2. Default Route:"; ip route | grep default
echo "3. DNS Servers:"; grep nameserver /etc/resolv.conf
echo "4. ARP Table:"; ip neigh show | head -5
EOF
chmod +x network-info.sh
./network-info.sh
```

### Connectivity Test Script

```bash
cat > test-connectivity.sh << 'EOF'
#!/bin/bash
echo "=== Connectivity Test ==="
ping -c 1 127.0.0.1 && echo "Localhost OK"
GW=$(ip route | grep default | awk '{print $3}' | head -1)
[ -n "$GW" ] && ping -c 1 "$GW" && echo "Gateway OK"
ping -c 1 8.8.8.8 && echo "Internet OK"
host google.com && echo "DNS OK"
EOF
chmod +x test-connectivity.sh
./test-connectivity.sh
```

---

## Key Learning Points

* `ip` replaces `ifconfig`
* Routing tables control traffic flow
* ARP maps IP → MAC
* DHCP automates network config
* DNS resolves names
* Hosts file overrides DNS
* `ss`/`netstat` show connections

---

## Security Relevance

* Detect ARP spoofing
* Identify rogue DHCP
* Catch DNS poisoning
* Block domains via hosts file
* Spot suspicious connections

---

## Summary

You now know how to:

* Inspect interfaces & routes
* Analyze ARP, DHCP, and DNS
* Use hosts file safely
* Monitor connections

➡️ These commands are core skills for networking, cloud, and security engineers.


# Scripts

## 1. Network Information Script
```sh
ubuntu:~$ vi network-info.sh 
ubuntu:~$ cat network-info.sh 
#!/bin/bash
echo "=== Network Information ==="
echo ""
echo "1. IP Addresses:"
ip -br addr show
echo ""
echo "2. Default Route:"
ip route | grep default
echo ""
echo "3. DNS Servers:"
cat /etc/resolv.conf | grep nameserver
echo ""
echo "4. ARP Table (first 5 entries):"
ip neigh show | head -5

ubuntu:~$ chmod +x network-info.sh
ubuntu:~$ ./network-info.sh
=== Network Information ===

1. IP Addresses:
lo               UNKNOWN        127.0.0.1/8 ::1/128 
enp1s0           UP             172.30.1.2/24 fe80::70aa:8e2a:ea46:e544/64 
docker0          DOWN           172.17.0.1/16 

2. Default Route:
default via 172.30.1.1 dev enp1s0 proto dhcp src 172.30.1.2 metric 1002 mtu 1500 

3. DNS Servers:
nameserver 8.8.8.8
nameserver 1.1.1.1

4. ARP Table (first 5 entries):
172.30.1.1 dev enp1s0 lladdr 02:00:00:00:00:00 REACHABLE
```

## 2. Test Connectivity Script

```sh
ubuntu:~$ vi test-connectivity.sh 
ubuntu:~$ cat test-connectivity.sh 
#!/bin/bash
echo "=== Network Connectivity Test ==="
echo ""
echo "1. Localhost:"
ping -c 1 127.0.0.1 > /dev/null 2>&1 && echo "✓ Localhost reachable" || echo "✗ Localhost failed"
echo ""
echo "2. Default Gateway:"
GATEWAY=$(ip route | grep default | awk '{print $3}' | head -1)
if [ -n "$GATEWAY" ]; then
    ping -c 1 $GATEWAY > /dev/null 2>&1 && echo "✓ Gateway $GATEWAY reachable" || echo "✗ Gateway unreachable"
else
    echo "No default gateway found"
fi
echo ""
echo "3. Internet (8.8.8.8):"
ping -c 1 8.8.8.8 > /dev/null 2>&1 && echo "✓ Internet reachable" || echo "✗ Internet unreachable"
echo ""
echo "4. DNS Resolution:"
host google.com > /dev/null 2>&1 && echo "✓ DNS working" || echo "✗ DNS failed"

ubuntu:~$ chmod +x test-connectivity.sh
ubuntu:~$ ./test-connectivity.sh
=== Network Connectivity Test ===

1. Localhost:
✓ Localhost reachable

2. Default Gateway:
✓ Gateway 172.30.1.1 reachable

3. Internet (8.8.8.8):
✓ Internet reachable

4. DNS Resolution:
✓ DNS working
```

## 3. Network Audit Script

```sh
ubuntu:~$ vi network-audit.sh 
ubuntu:~$ cat network-audit.sh 
#!/bin/bash
echo "=== Network Security Audit ==="
echo ""
echo "1. Network Interfaces:"
ip -br addr show
echo ""
echo "2. Active Connections:"
ss -tunp 2>/dev/null | head -5 || echo "ss not available"
echo ""
echo "3. ARP Table:"
ip neigh show | head -5
echo ""
echo "4. DNS Configuration:"
cat /etc/resolv.conf
echo ""
echo "5. Routing Table:"
ip route show | head -5
ubuntu:~$ 


ubuntu:~$ chmod +x network-audit.sh
ubuntu:~$ ./network-audit.sh

=== Network Security Audit ===

1. Network Interfaces:
lo               UNKNOWN        127.0.0.1/8 ::1/128 
enp1s0           UP             172.30.1.2/24 fe80::70aa:8e2a:ea46:e544/64 
docker0          DOWN           172.17.0.1/16 

2. Active Connections:
Netid State Recv-Q Send-Q Local Address:Port  Peer Address:Port Process                         
tcp   ESTAB 0      0         172.30.1.2:40205 10.244.4.181:42778 users:(("node",pid=1232,fd=19))
tcp   ESTAB 0      0         172.30.1.2:40200 10.244.3.183:39122 users:(("kc-terminal",pid=1281,fd=13))

3. ARP Table:
172.30.1.1 dev enp1s0 lladdr 02:00:00:00:00:00 REACHABLE 

4. DNS Configuration:
# This is /run/systemd/resolve/resolv.conf managed by man:systemd-resolved(8).
# Do not edit.
#
# This file might be symlinked as /etc/resolv.conf. If you're looking at
# /etc/resolv.conf and seeing this text, you have followed the symlink.
#
# This is a dynamic resolv.conf file for connecting local clients directly to
# all known uplink DNS servers. This file lists all configured search domains.
#
# Third party programs should typically not access this file directly, but only
# through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
# different way, replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 8.8.8.8
nameserver 1.1.1.1
search .

5. Routing Table:
default via 172.30.1.1 dev enp1s0 proto dhcp src 172.30.1.2 metric 1002 mtu 1500 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
172.30.1.0/24 dev enp1s0 proto dhcp scope link src 172.30.1.2 metric 1002 mtu 1500 
ubuntu:~$
```
