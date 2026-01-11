# Linux Networking Troubleshooting Lab & Mastery Project

This document contains a layered Linux networking troubleshooting lab sheet and a detailed hands-on mastery project. It preserves the original tasks, commands, and explanations while formatting them for readability. Each task includes what it does, use cases, the OSI layer(s) involved, and how layers interact.

---

## Table of Contents

- [Layer 1 & 2: Physical & Data Link Layer](#layer-1--2-physical--data-link-layer)  
  - [Task 1: `ip link show` / `ifconfig -a`](#task-1-ip-link-show--ifconfig-a)  
  - [Task 2: `ethtool eth0`](#task-2-ethtool-eth0)  
  - [Task 3: `arp -n` / `ip neigh show`](#task-3-arp--n--ip-neigh-show)

- [Layer 3: Network Layer](#layer-3-network-layer)  
  - [Task 4: `ip addr show`](#task-4-ip-addr-show)  
  - [Task 5: `ip route show` / `route -n`](#task-5-ip-route-show--route-n)  
  - [Task 6: `ping -c 4 <host>`](#task-6-ping--c-4-host)  
  - [Task 7: `traceroute <host>` / `tracepath <host>`](#task-7-traceroute-host--tracepath-host)  
  - [Task 8: `ping -M do -s <size>`](#task-8-ping--m-do--s-size)  
  - [Task 9: `nslookup` / `dig`](#task-9-nslookup--dig)

- [Layer 4: Transport Layer](#layer-4-transport-layer)  
  - [Task 10: `ss -tulnp` / `netstat -tulnp`](#task-10-ss--tulnp--netstat--tulnp)  
  - [Task 11: `nc -zv <host> <port>`](#task-11-nc--zv-host--port)  
  - [Task 12: `ss -s` / `ss -antp`](#task-12-ss--s--ss--antp)

- [Advanced / Troubleshooting Tasks (13–20)](#advanced--troubleshooting-tasks-13--20)  
  - [Task 13: `iptables -L -n -v` / `nft list ruleset`](#task-13-iptables--l--n--v--nft-list-ruleset)  
  - [Task 14: `tcpdump -i eth0 -c 10`](#task-14-tcpdump--i-eth0--c-10)  
  - [Task 15: `ip -6 neigh show`](#task-15-ip--6-neigh-show)  
  - [Task 16: Multicast / Broadcast Testing (`ping -b` / `ss -u -a`)](#task-16-multicast--broadcast-testing-ping--b--ss--u--a)  
  - [Task 17: Subnet Verification (`ip addr show`)](#task-17-subnet-verification-ip-addr-show)  
  - [Task 18: Test Routing with Specific Interface (`ping -I eth0`)](#task-18-test-routing-with-specific-interface-ping--i-eth0)  
  - [Task 19: Verify External Connectivity (`curl -I http://example.com`)](#task-19-verify-external-connectivity-curl--i-httpexamplecom)  
  - [Task 20: Network Statistics (`netstat -i` / `ifconfig`)](#task-20-network-statistics-netstat--i--ifconfig)

- [Layer → Tasks Mapping](#layer--tasks-mapping)

- [Linux Networking Troubleshooting Lab Sheet (table)](#linux-networking-troubleshooting-lab-sheet-table)

- [Linux Networking Mastery Lab Project](#linux-networking-mastery-lab-project)  
  - [Phase 0: Lab Setup & Preparation](#phase-0-lab-setup--preparation)  
  - [Phase 1: Basic Interface and IP Configuration](#phase-1-basic-interface-and-ip-configuration)  
  - [Phase 2: LAN Troubleshooting](#phase-2-lan-troubleshooting)  
  - [Phase 3: Routing & Inter-network Troubleshooting](#phase-3-routing--inter-network-troubleshooting)  
  - [Phase 4: DNS Troubleshooting](#phase-4-dns-troubleshooting)  
  - [Phase 5: Transport Layer Troubleshooting](#phase-5-transport-layer-troubleshooting)  
  - [Phase 6: Firewall & Security](#phase-6-firewall--security)  
  - [Phase 7: Packet Capture & Analysis](#phase-7-packet-capture--analysis)  
  - [Phase 8: IPv6 Troubleshooting](#phase-8-ipv6-troubleshooting)  
  - [Phase 9: End-to-End Service Testing](#phase-9-end-to-end-service-testing)  
  - [Phase 10: Advanced Troubleshooting Scenarios](#phase-10-advanced-troubleshooting-scenarios)

- [Project Outcome](#project-outcome)

---

## Layer 1 & 2: Physical & Data Link Layer

### Task 1: ip link show / ifconfig -a
- What it does: Shows all network interfaces on the system, including their state (UP/DOWN), MTU, and MAC addresses.  
- Use case:
  - Troubleshooting whether interfaces are active
  - Detecting hardware/network card issues
  - Checking MAC address for ARP or LAN connectivity issues
- Layer:
  - Physical Layer (Layer 1): Status of the network interface (cable connected, link detected)
  - Data Link Layer (Layer 2): MAC addresses, frame-level info
- Layer interaction:
  - Physical layer provides the actual signal; Data Link uses MAC to identify devices and transmit frames.

Example commands:
```bash
ip link show
ifconfig -a
```

---

### Task 2: ethtool eth0
- What it does: Shows link speed, duplex, auto-negotiation, and driver info for the NIC.  
- Use case:
  - Detecting speed mismatches (100 Mbps vs 1 Gbps)
  - Duplex errors causing collisions or packet loss
  - Troubleshooting poor performance on LAN
- Layer:
  - Physical Layer (Layer 1) for speed/duplex
  - Data Link Layer (Layer 2) for MAC/driver info
- Layer interaction:
  - NIC must operate at correct physical layer speed to deliver frames without collisions.

Example command:
```bash
ethtool eth0
```

---

### Task 3: arp -n / ip neigh show
- What it does: Shows the mapping of IP addresses to MAC addresses on your LAN.  
- Use case:
  - Troubleshooting local network communication issues
  - Detecting duplicate IPs or ARP poisoning
- Layer:
  - Data Link Layer (Layer 2) for MAC addresses
  - Network Layer (Layer 3) uses ARP to map IPs to MACs
- Layer interaction:
  - When a system wants to send a packet to a local IP, it asks “what is your MAC?”
  - ARP resolves IP → MAC so Layer 2 can transmit frames.

Commands:
```bash
arp -n
ip neigh show
```

---

## Layer 3: Network Layer

### Task 4: ip addr show
- What it does: Lists all IP addresses (IPv4 & IPv6) assigned to interfaces.  
- Use case:
  - Troubleshooting connectivity: IP assigned correctly?
  - Checking subnets & addressing conflicts
- Layer:
  - Network Layer (Layer 3) for IP addresses
- Layer interaction:
  - Network layer encapsulates data from Transport Layer into IP packets
  - Data Link Layer wraps IP packets into frames for physical transmission

Command:
```bash
ip addr show
```

---

### Task 5: ip route show / route -n
- What it does: Displays routing table and default gateways.  
- Use case:
  - Troubleshooting “no route to host” issues
  - Checking default gateway and subnet routes
- Layer:
  - Network Layer (Layer 3)
- Layer interaction:
  - IP layer uses routing table to decide next hop
  - Data Link Layer handles delivery to the next hop MAC address

Commands:
```bash
ip route show
route -n
```

---

### Task 6: ping -c 4 <host>
- What it does: Sends ICMP echo requests to test host reachability and measures round-trip time.  
- Use case:
  - Detecting network reachability issues
  - Measuring latency
  - Detecting packet loss
- Layer:
  - Network Layer (ICMP is Layer 3 protocol)
- Layer interaction:
  - ICMP packet encapsulated in IP (Layer 3), then Ethernet frame (Layer 2), transmitted over physical medium (Layer 1)

Example:
```bash
ping -c 4 google.com
```

---

### Task 7: traceroute <host> / tracepath <host>
- What it does: Shows the path packets take from your host to a destination.  
- Use case:
  - Identifying where delays or drops occur in network
  - Troubleshooting routing or ISP issues
- Layer:
  - Network Layer (IP TTL handling)
- Layer interaction:
  - Each hop decreases TTL (network layer)
  - Data Link Layer delivers each IP packet frame by frame to next hop

Commands:
```bash
traceroute google.com
tracepath google.com
```

---

### Task 8: ping -M do -s <size>
- What it does: Tests MTU size and fragmentation; `-M do` avoids fragmentation.  
- Use case:
  - Detecting packet size issues that may block traffic
  - Troubleshooting PMTUD (Path MTU Discovery) issues
- Layer:
  - Network Layer (IP fragmentation)
- Layer interaction:
  - Large IP packet split if needed; Ethernet frames must accommodate payload

Example:
```bash
ping -M do -s 1472 google.com
```

---

### Task 9: nslookup / dig
- What it does: Resolves domain names to IP addresses.  
- Use case:
  - DNS troubleshooting
  - Verifying system can reach DNS servers
- Layer:
  - Application Layer query, but relies on Network Layer (UDP/TCP to DNS server)
- Layer interaction:
  - DNS request sent over UDP/TCP (Transport Layer) encapsulated in IP packets (Network Layer)

Commands:
```bash
nslookup example.com
dig example.com
```

---

## Layer 4: Transport Layer

### Task 10: ss -tulnp / netstat -tulnp
- What it does: Shows listening TCP/UDP ports and processes.  
- Use case:
  - Check if services are running
  - Detect port conflicts or misconfigurations
- Layer:
  - Transport Layer (TCP/UDP)
  - Network Layer (IP addresses)
- Layer interaction:
  - Transport layer encapsulates application data into TCP/UDP segments
  - Network Layer delivers to correct IP, Data Link Layer delivers frame

Commands:
```bash
ss -tulnp
netstat -tulnp
```

---

### Task 11: nc -zv <host> <port>
- What it does: Tests TCP/UDP connectivity to a specific port.  
- Use case:
  - Checking firewall or service reachability
  - Troubleshooting service availability
- Layer:
  - Transport Layer (TCP/UDP)
- Layer interaction:
  - TCP handshake occurs (SYN/ACK)
  - If network layer drops packet, connection fails

Example:
```bash
nc -zv 192.168.1.20 80
```

---

### Task 12: ss -s / ss -antp
- What it does: Shows active TCP connection states and counts.  
- Use case:
  - Detect half-open connections
  - Monitor network congestion or service issues
- Layer:
  - Transport Layer
- Layer interaction:
  - Helps understand end-to-end connection state (TCP handshake)
  - Encapsulated in IP (Network Layer) for delivery

Commands:
```bash
ss -s
ss -antp
```

---

## Advanced / Troubleshooting Tasks (13–20)

### Task 13: iptables -L -n -v / nft list ruleset
- What it does: Lists firewall rules.  
- Use case:
  - Troubleshooting blocked ports or protocols
  - Verifying access control
- Layer:
  - Network Layer (IP rules)
  - Transport Layer (TCP/UDP rules)
- Layer interaction:
  - Firewall inspects headers at Layer 3/4 before packet delivery

Commands:
```bash
sudo iptables -L -n -v
sudo nft list ruleset
```

---

### Task 14: tcpdump -i eth0 -c 10
- What it does: Captures network packets on a specific interface.  
- Use case:
  - Diagnosing unknown traffic issues
  - Debugging protocols or packet loss
- Layer:
  - Data Link Layer (captures Ethernet frames)
  - Network Layer (IP), Transport Layer (TCP/UDP)
- Layer interaction:
  - Lets you see encapsulation: Layer 2 frame contains Layer 3 packet with Layer 4 segment

Command:
```bash
sudo tcpdump -i eth0 -c 10
```

---

### Task 15: ip -6 neigh show
- What it does: Shows IPv6 neighbor cache (IPv6 version of ARP).  
- Use case:
  - Troubleshooting IPv6 connectivity
  - Detecting duplicate addresses
- Layer:
  - Data Link Layer (MAC)
  - Network Layer (IPv6)
- Layer interaction:
  - IPv6 uses neighbor discovery to map IP → MAC for local delivery

Command:
```bash
ip -6 neigh show
```

---

### Task 16: Multicast / Broadcast Testing (ping -b / ss -u -a)
- What it does: Tests broadcast or multicast reachability.  
- Use case:
  - Detecting LAN reachability
  - Testing UDP services that rely on broadcast/multicast
- Layer:
  - Network Layer (IP broadcast/multicast)
  - Transport Layer (UDP)
- Layer interaction:
  - Layer 2 delivers broadcast frame to all hosts on LAN
  - Network Layer identifies addresses for correct delivery

Examples:
```bash
ping -b 192.168.0.255
ss -u -a
```

---

### Task 17: Subnet Verification (ip addr show)
- What it does: Checks IP and subnet mask configuration.  
- Use case:
  - Detect IP misconfiguration
  - Verify two hosts are on the same subnet
- Layer:
  - Network Layer
- Layer interaction:
  - Network Layer decides if destination is local or requires gateway routing

Command:
```bash
ip addr show eth0
```

---

### Task 18: Test Routing with Specific Interface (ping -I eth0)
- What it does: Forces ping out a specific interface.  
- Use case:
  - Troubleshoot multi-interface machines
  - Check interface-specific connectivity
- Layer:
  - Network Layer (IP routing)
- Layer interaction:
  - Ensures correct Layer 3 routing
  - Layer 2 delivers frame via specified NIC

Example:
```bash
ping -I eth0 8.8.8.8
```

---

### Task 19: Verify External Connectivity (curl -I http://example.com)
- What it does: Sends HTTP HEAD request to external server.  
- Use case:
  - Check both DNS resolution and HTTP connectivity
  - Troubleshoot ISP or firewall issues
- Layer:
  - Application Layer (HTTP)
  - Transport Layer (TCP)
  - Network Layer (IP)
- Layer interaction:
  - Shows end-to-end connectivity from app → transport → network → data link

Command:
```bash
curl -I http://example.com
```

---

### Task 20: Network Statistics (netstat -i / ifconfig)
- What it does: Shows packet counts, errors, collisions per interface.  
- Use case:
  - Detect interface errors or packet loss
  - Diagnose LAN performance issues
- Layer:
  - Data Link Layer (Ethernet frames)
  - Physical Layer (link errors)
- Layer interaction:
  - Layer 1/2 errors impact Layer 3 delivery
  - Helps trace source of packet loss

Commands:
```bash
netstat -i
ifconfig
```

```sh
ubuntu:~$ ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 62:c6:a2:36:3b:cd brd ff:ff:ff:ff:ff:ff
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1454 qdisc noqueue state DOWN mode DEFAULT group default 
    link/ether 62:1a:be:19:8a:99 brd ff:ff:ff:ff:ff:ff


ubuntu:~$ ifconfig -a
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1454
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 62:1a:be:19:8a:99  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enp1s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.30.1.2  netmask 255.255.255.0  broadcast 172.30.1.255
        inet6 fe80::1995:6751:3ede:c0c5  prefixlen 64  scopeid 0x20<link>
        ether 62:c6:a2:36:3b:cd  txqueuelen 1000  (Ethernet)
        RX packets 22129  bytes 30685386 (30.6 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 6006  bytes 9663311 (9.6 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0


ubuntu:~$ ethtool eth0
netlink error: no device matches name (offset 24)
netlink error: No such device
netlink error: no device matches name (offset 24)
netlink error: No such device
netlink error: no device matches name (offset 24)
netlink error: No such device
netlink error: no device matches name (offset 24)
netlink error: No such device
netlink error: no device matches name (offset 24)
netlink error: No such device
netlink error: no device matches name (offset 24)
netlink error: No such device
netlink error: no device matches name (offset 24)
netlink error: No such device
No data available


ubuntu:~$ arp -n
Address                  HWtype  HWaddress           Flags Mask            Iface
172.30.1.1               ether   02:00:00:00:00:00   C                     enp1s0


ubuntu:~$ ip neigh show
172.30.1.1 dev enp1s0 lladdr 02:00:00:00:00:00 REACHABLE 


ubuntu:~$ ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 62:c6:a2:36:3b:cd brd ff:ff:ff:ff:ff:ff
    inet 172.30.1.2/24 brd 172.30.1.255 scope global dynamic noprefixroute enp1s0
       valid_lft 86313325sec preferred_lft 75524125sec
    inet6 fe80::1995:6751:3ede:c0c5/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1454 qdisc noqueue state DOWN group default 
    link/ether 62:1a:be:19:8a:99 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever


ubuntu:~$ ip route show
default via 172.30.1.1 dev enp1s0 proto dhcp src 172.30.1.2 metric 1002 mtu 1500 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
172.30.1.0/24 dev enp1s0 proto dhcp scope link src 172.30.1.2 metric 1002 mtu 1500 


ubuntu:~$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.30.1.1      0.0.0.0         UG    1002   0        0 enp1s0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
172.30.1.0      0.0.0.0         255.255.255.0   U     1002   0        0 enp1s0


ubuntu:~$ ping -c 4 google.com
PING google.com (142.250.185.206) 56(84) bytes of data.
64 bytes from fra16s52-in-f14.1e100.net (142.250.185.206): icmp_seq=1 ttl=117 time=4.64 ms
64 bytes from fra16s52-in-f14.1e100.net (142.250.185.206): icmp_seq=2 ttl=117 time=3.91 ms
64 bytes from fra16s52-in-f14.1e100.net (142.250.185.206): icmp_seq=3 ttl=117 time=4.10 ms
64 bytes from fra16s52-in-f14.1e100.net (142.250.185.206): icmp_seq=4 ttl=117 time=3.95 ms

--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 3.906/4.148/4.642/0.293 ms


ubuntu:~$ traceroute google.com
Command 'traceroute' not found, but can be installed with:
apt install inetutils-traceroute  # version 2:2.4-3ubuntu1, or
apt install traceroute            # version 1:2.1.5-1
ubuntu:~$ apt install traceroute
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following package was automatically installed and is no longer required:
  squashfs-tools
Use 'apt autoremove' to remove it.
The following NEW packages will be installed:
  traceroute
0 upgraded, 1 newly installed, 0 to remove and 20 not upgraded.
Need to get 60.5 kB of archives.
After this operation, 162 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu noble/universe amd64 traceroute amd64 1:2.1.5-1 [60.5 kB]
Fetched 60.5 kB in 0s (628 kB/s)
Selecting previously unselected package traceroute.
(Reading database ... 113825 files and directories currently installed.)
Preparing to unpack .../traceroute_1%3a2.1.5-1_amd64.deb ...
Unpacking traceroute (1:2.1.5-1) ...
Setting up traceroute (1:2.1.5-1) ...
update-alternatives: using /usr/bin/traceroute.db to provide /usr/bin/traceroute (traceroute) in auto mode
update-alternatives: using /usr/bin/traceroute6.db to provide /usr/bin/traceroute6 (traceroute6) in auto mode
update-alternatives: using /usr/bin/lft.db to provide /usr/bin/lft (lft) in auto mode
update-alternatives: using /usr/bin/traceproto.db to provide /usr/bin/traceproto (traceproto) in auto mode
update-alternatives: using /usr/sbin/tcptraceroute.db to provide /usr/sbin/tcptraceroute (tcptraceroute) in auto mode
Processing triggers for man-db (2.12.0-4build2) ...
Scanning processes...                                                                                                                                                                        
Scanning linux images...                                                                                                                                                                     

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.


ubuntu:~$ traceroute google.com
traceroute to google.com (142.250.185.206), 30 hops max, 60 byte packets
 1  172.30.1.1 (172.30.1.1)  0.080 ms  0.044 ms  0.040 ms
 2  10.8.0.1 (10.8.0.1)  6.024 ms  6.010 ms  5.992 ms
 3  192.168.1.1 (192.168.1.1)  5.974 ms  6.022 ms  6.005 ms
 4  irb-2.router-02.fra1.civo.io (74.220.24.3)  6.029 ms  6.015 ms  6.229 ms
 5  lag-111.ear5.Frankfurt1.Level3.net (62.67.68.101)  6.360 ms  6.343 ms  6.468 ms
 6  * * *
 7  72.14.213.242 (72.14.213.242)  5.134 ms  7.403 ms  7.311 ms
 8  192.178.109.157 (192.178.109.157)  7.350 ms 192.178.109.241 (192.178.109.241)  7.247 ms 192.178.109.157 (192.178.109.157)  7.289 ms
 9  142.250.213.213 (142.250.213.213)  7.200 ms 142.250.225.77 (142.250.225.77)  7.358 ms 142.250.213.213 (142.250.213.213)  7.333 ms
10  fra16s52-in-f14.1e100.net (142.250.185.206)  7.314 ms  7.371 ms  7.893 ms


ubuntu:~$ tracepath google.com
 1?: [LOCALHOST]                      pmtu 1500
 1:  172.30.1.1                                            0.098ms 
 1:  172.30.1.1                                            0.093ms 
 2:  172.30.1.1                                            0.160ms pmtu 1400
 2:  10.8.0.1                                              2.366ms 
 3:  192.168.1.1                                           3.085ms 
 4:  irb-2.router-02.fra1.civo.io                          3.354ms 
 5:  no reply
 6:  no reply
 7:  72.14.213.242                                         5.892ms 
 8:  no reply
 9:  no reply


ubuntu:~$ ping -M do -s 1472 google.com
PING google.com (142.250.185.206) 1472(1500) bytes of data.
ping: local error: message too long, mtu=1400
ping: local error: message too long, mtu=1400
ping: local error: message too long, mtu=1400
ping: local error: message too long, mtu=1400
ping: local error: message too long, mtu=1400
ping: local error: message too long, mtu=1400
^C
--- google.com ping statistics ---
6 packets transmitted, 0 received, +6 errors, 100% packet loss, time 5102ms


ubuntu:~$ nslookup google.com 
Server:         8.8.8.8
Address:        8.8.8.8#53

Non-authoritative answer:
Name:   google.com
Address: 142.250.185.206
Name:   google.com
Address: 2a00:1450:4001:812::200e


ubuntu:~$ dig google.com

# ;; DiG 9.18.39-0ubuntu0.24.04.2-Ubuntu google.com
# ;; global options: +cmd
# ;; Got answer:
# ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 31708
# ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

# ;; OPT PSEUDOSECTION:
# ; EDNS: version: 0, flags:; udp: 512
# ;; QUESTION SECTION:
# ;google.com.                    IN      A

# ;; ANSWER SECTION:
# google.com.             194     IN      A       142.250.185.206

# ;; Query time: 6 msec
# ;; SERVER: 8.8.8.8#53(8.8.8.8) (UDP)
# ;; WHEN: Sun Jan 11 18:16:47 UTC 2026
# ;; MSG SIZE  rcvd: 55


ubuntu:~$ netstat -tulnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1/init              
tcp        0      0 0.0.0.0:40205           0.0.0.0:*               LISTEN      1222/node           
tcp        0      0 0.0.0.0:40200           0.0.0.0:*               LISTEN      1271/kc-terminal    
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      1185/systemd-resolv 
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      1185/systemd-resolv 
tcp        0      0 127.0.0.1:39845         0.0.0.0:*               LISTEN      715/containerd      
tcp6       0      0 :::22                   :::*                    LISTEN      1/init              
tcp6       0      0 :::40305                :::*                    LISTEN      1259/runtime-info-s 
tcp6       0      0 :::40300                :::*                    LISTEN      1212/runtime-scenar 
udp        0      0 127.0.0.54:53           0.0.0.0:*                           1185/systemd-resolv 
udp        0      0 127.0.0.53:53           0.0.0.0:*                           1185/systemd-resolv 
udp        0      0 172.30.1.2:68           0.0.0.0:*                           1132/dhcpcd: [BOOTP 
udp        0      0 172.30.1.2:68           0.0.0.0:*                           455/systemd-network 
udp6       0      0 fe80::1995:6751:3ed:546 :::*                                829/dhcpcd: [DHCP6  


ubuntu:~$ ss -tulnp
Netid      State       Recv-Q      Send-Q                                Local Address:Port              Peer Address:Port      Process                                                      
udp        UNCONN      0           0                                        127.0.0.54:53                     0.0.0.0:*          users:(("systemd-resolve",pid=1185,fd=16))                  
udp        UNCONN      0           0                                     127.0.0.53%lo:53                     0.0.0.0:*          users:(("systemd-resolve",pid=1185,fd=14))                  
udp        UNCONN      0           0                                        172.30.1.2:68                     0.0.0.0:*          users:(("dhcpcd",pid=1132,fd=7))                            
udp        UNCONN      0           0                                 172.30.1.2%enp1s0:68                     0.0.0.0:*          users:(("systemd-network",pid=455,fd=21))                   
udp        UNCONN      0           0                [fe80::1995:6751:3ede:c0c5]%enp1s0:546                       [::]:*          users:(("dhcpcd",pid=829,fd=7))                             
tcp        LISTEN      0           4096                                        0.0.0.0:22                     0.0.0.0:*          users:(("sshd",pid=1189,fd=3),("systemd",pid=1,fd=90))      
tcp        LISTEN      0           511                                         0.0.0.0:40205                  0.0.0.0:*          users:(("node",pid=1222,fd=18))                             
tcp        LISTEN      0           128                                         0.0.0.0:40200                  0.0.0.0:*          users:(("kc-terminal",pid=1271,fd=12))                      
tcp        LISTEN      0           4096                                     127.0.0.54:53                     0.0.0.0:*          users:(("systemd-resolve",pid=1185,fd=17))                  
tcp        LISTEN      0           4096                                  127.0.0.53%lo:53                     0.0.0.0:*          users:(("systemd-resolve",pid=1185,fd=15))                  
tcp        LISTEN      0           4096                                      127.0.0.1:39845                  0.0.0.0:*          users:(("containerd",pid=715,fd=9))                         
tcp        LISTEN      0           4096                                           [::]:22                        [::]:*          users:(("sshd",pid=1189,fd=4),("systemd",pid=1,fd=91))      
tcp        LISTEN      0           4096                                              *:40305                        *:*          users:(("runtime-info-se",pid=1259,fd=3))                   
tcp        LISTEN      0           4096                                              *:40300                        *:*          users:(("runtime-scenari",pid=1212,fd=3))                   


ubuntu:~$ nc -zv 192.168.1.20 80
^C


ubuntu:~$ ss -s
Total: 187
TCP:   10 (estab 1, closed 0, orphaned 0, timewait 0)

Transport Total     IP        IPv6
RAW       5         1         4        
UDP       5         4         1        
TCP       10        7         3        
INET      20        12        8        
FRAG      0         0         0        


ubuntu:~$ ss -antp
State           Recv-Q          Send-Q                   Local Address:Port                     Peer Address:Port           Process                                                          
LISTEN          0               4096                           0.0.0.0:22                            0.0.0.0:*               users:(("sshd",pid=1189,fd=3),("systemd",pid=1,fd=90))          
LISTEN          0               511                            0.0.0.0:40205                         0.0.0.0:*               users:(("node",pid=1222,fd=18))                                 
LISTEN          0               128                            0.0.0.0:40200                         0.0.0.0:*               users:(("kc-terminal",pid=1271,fd=12))                          
LISTEN          0               4096                        127.0.0.54:53                            0.0.0.0:*               users:(("systemd-resolve",pid=1185,fd=17))                      
LISTEN          0               4096                     127.0.0.53%lo:53                            0.0.0.0:*               users:(("systemd-resolve",pid=1185,fd=15))                      
LISTEN          0               4096                         127.0.0.1:39845                         0.0.0.0:*               users:(("containerd",pid=715,fd=9))                             
ESTAB           0               0                           172.30.1.2:40200                    10.244.12.22:48752           users:(("kc-terminal",pid=1271,fd=13))                          
LISTEN          0               4096                              [::]:22                               [::]:*               users:(("sshd",pid=1189,fd=4),("systemd",pid=1,fd=91))          
LISTEN          0               4096                                 *:40305                               *:*               users:(("runtime-info-se",pid=1259,fd=3))                       
LISTEN          0               4096                                 *:40300                               *:*               users:(("runtime-scenari",pid=1212,fd=3))                       


ubuntu:~$ sudo iptables -L -n -v
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-USER  0    --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 DOCKER-FORWARD  0    --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DROP       0    --  !docker0 docker0  0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-BRIDGE (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER     0    --  *      docker0  0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-CT (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     0    --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED

Chain DOCKER-FORWARD (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-CT  0    --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 DOCKER-ISOLATION-STAGE-1  0    --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 DOCKER-BRIDGE  0    --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     0    --  docker0 *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-ISOLATION-STAGE-2  0    --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DROP       0    --  *      docker0  0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-USER (1 references)
 pkts bytes target     prot opt in     out     source               destination         


ubuntu:~$ sudo nft list ruleset
# Warning: table ip nat is managed by iptables-nft, do not touch!
table ip nat {
        chain DOCKER {
                iifname "docker0" counter packets 0 bytes 0 return
        }

        chain PREROUTING {
                type nat hook prerouting priority dstnat; policy accept;
                fib daddr type local counter packets 20 bytes 1200 jump DOCKER
        }

        chain OUTPUT {
                type nat hook output priority dstnat; policy accept;
                ip daddr != 127.0.0.0/8 fib daddr type local counter packets 0 bytes 0 jump DOCKER
        }

        chain POSTROUTING {
                type nat hook postrouting priority srcnat; policy accept;
                ip saddr 172.17.0.0/16 oifname != "docker0" counter packets 0 bytes 0 masquerade
        }
}
# Warning: table ip filter is managed by iptables-nft, do not touch!
table ip filter {
        chain DOCKER {
                iifname != "docker0" oifname "docker0" counter packets 0 bytes 0 drop
        }

        chain DOCKER-FORWARD {
                counter packets 0 bytes 0 jump DOCKER-CT
                counter packets 0 bytes 0 jump DOCKER-ISOLATION-STAGE-1
                counter packets 0 bytes 0 jump DOCKER-BRIDGE
                iifname "docker0" counter packets 0 bytes 0 accept
        }

        chain DOCKER-BRIDGE {
                oifname "docker0" counter packets 0 bytes 0 jump DOCKER
        }

        chain DOCKER-CT {
                oifname "docker0" ct state related,established counter packets 0 bytes 0 accept
        }

        chain DOCKER-ISOLATION-STAGE-1 {
                iifname "docker0" oifname != "docker0" counter packets 0 bytes 0 jump DOCKER-ISOLATION-STAGE-2
        }

        chain DOCKER-ISOLATION-STAGE-2 {
                oifname "docker0" counter packets 0 bytes 0 drop
        }

        chain FORWARD {
                type filter hook forward priority filter; policy accept;
                counter packets 0 bytes 0 jump DOCKER-USER
                counter packets 0 bytes 0 jump DOCKER-FORWARD
        }

        chain DOCKER-USER {
        }
}
# Warning: table ip6 nat is managed by iptables-nft, do not touch!
table ip6 nat {
        chain DOCKER {
        }

        chain PREROUTING {
                type nat hook prerouting priority dstnat; policy accept;
                fib daddr type local counter packets 0 bytes 0 jump DOCKER
        }

        chain OUTPUT {
                type nat hook output priority dstnat; policy accept;
                ip6 daddr != ::1 fib daddr type local counter packets 0 bytes 0 jump DOCKER
        }
}
table ip6 filter {
        chain DOCKER {
        }

        chain DOCKER-FORWARD {
                counter packets 0 bytes 0 jump DOCKER-CT
                counter packets 0 bytes 0 jump DOCKER-ISOLATION-STAGE-1
                counter packets 0 bytes 0 jump DOCKER-BRIDGE
        }

        chain DOCKER-BRIDGE {
        }

        chain DOCKER-CT {
        }

        chain DOCKER-ISOLATION-STAGE-1 {
        }

        chain DOCKER-ISOLATION-STAGE-2 {
        }

        chain FORWARD {
                type filter hook forward priority filter; policy accept;
                counter packets 0 bytes 0 jump DOCKER-USER
                counter packets 0 bytes 0 jump DOCKER-FORWARD
        }

        chain DOCKER-USER {
        }
}


ubuntu:~$ sudo tcpdump -i eth0 -c 10
tcpdump: eth0: No such device exists
(No such device exists)


ubuntu:~$ ip -6 neigh show
ubuntu:~$ ip addr show eth0
Device "eth0" does not exist.


ubuntu:~$ ping -I eth0 8.8.8.8
ping: SO_BINDTODEVICE eth0: No such device
ubuntu:~$ 
```

---

## Layer → Tasks Mapping

| Layer | Tasks (numbers) |
|-------|-----------------|
| Layer 1 (Physical) | 1, 2, 20 (link status, speed, errors) |
| Layer 2 (Data Link) | 1, 2, 3, 14, 15, 16, 20 (MAC, ARP, packet capture) |
| Layer 3 (Network) | 4, 5, 6, 7, 8, 9, 13, 14, 16, 17, 18, 19 |
| Layer 4 (Transport) | 10, 11, 12, 13, 14, 16, 19 |
| Layer 7 (Application) | 9, 19 |

---

## Linux Networking Troubleshooting Lab Sheet (table)

| Task | Description | Command | Layer(s) | Issues Detected | Notes |
|------|-------------|---------|----------|-----------------|-------|
| 1 | List all network interfaces and MAC addresses | `ip link show` or `ifconfig -a` | Physical (L1), Data Link (L2) | Interface up/down, hardware connectivity, MAC conflicts | Use to verify interface exists and is active |
| 2 | Check NIC speed, duplex, and driver | `ethtool eth0` | Physical (L1), Data Link (L2) | Speed mismatch, collisions, duplex errors | Critical for LAN troubleshooting |
| 3 | Display ARP table | `arp -n` or `ip neigh show` | Data Link (L2), Network (L3) | Duplicate IPs, LAN connectivity issues | Resolves IP → MAC mapping |
| 4 | Show IP addresses for interfaces | `ip addr show` | Network (L3) | Wrong IP, subnet misconfiguration | Checks IPv4 & IPv6 |
| 5 | Show routing table | `ip route show` or `route -n` | Network (L3) | No route to host, misconfigured default gateway | Identifies which interface sends traffic |
| 6 | Test connectivity and latency | `ping -c 4 google.com` | Network (L3) | Host unreachable, packet loss, latency | ICMP test, first step in troubleshooting |
| 7 | Trace route to host | `traceroute google.com` or `tracepath google.com` | Network (L3) | Routing loops, network delays, hop failures | Finds which hop drops packets |
| 8 | Test MTU and fragmentation | `ping -M do -s 1472 google.com` | Network (L3) | MTU issues, dropped packets | Adjust size to detect PMTUD problems |
| 9 | Test DNS resolution | `nslookup example.com` or `dig example.com` | Application → Network (L7→L3) | Name resolution failure, unreachable DNS | Confirms system can resolve hostnames |
| 10 | List all listening TCP/UDP ports | `ss -tulnp` or `netstat -tulnp` | Transport (L4), Network (L3) | Service not running, port conflicts | Shows PID/process using port |
| 11 | Test TCP/UDP connectivity | `nc -zv 192.168.1.20 80` | Transport (L4), Network (L3) | Firewall block, service down | Quick test if a port is reachable |
| 12 | Check active TCP connection states | `ss -s` or `ss -antp` | Transport (L4), Network (L3) | Half-open connections, congestion | Monitors connection states for troubleshooting |
| 13 | List firewall rules | `sudo iptables -L -n -v` or `sudo nft list ruleset` | Network (L3), Transport (L4) | Blocked ports, IP filtering issues | Helps debug connectivity issues caused by firewall |
| 14 | Capture packets on an interface | `sudo tcpdump -i eth0 -c 10` | Data Link (L2), Network (L3), Transport (L4) | Malformed packets, traffic issues | Can analyze packet-level flow |
| 15 | Show IPv6 neighbors | `ip -6 neigh show` | Data Link (L2), Network (L3) | Duplicate addresses, neighbor issues | IPv6 equivalent of ARP table |
| 16 | Test broadcast/multicast reachability | `ping -b 192.168.0.255` / `ss -u -a` | Network (L3), Transport (L4) | LAN communication issues, UDP service reachability | Useful for services relying on broadcast/multicast |
| 17 | Verify subnet configuration | `ip addr show eth0` | Network (L3) | Misconfigured subnet mask, overlapping networks | Confirms host and gateway are in same subnet |
| 18 | Test routing via specific interface | `ping -I eth0 google.com` | Network (L3) | Multi-interface routing issues | Ensures correct interface sends packets |
| 19 | Verify external connectivity and HTTP service | `curl -I http://example.com` | Application (L7), Transport (L4), Network (L3) | Service unreachable, DNS failure, firewall issues | Combines DNS, TCP, and HTTP checks |
| 20 | Display network statistics per interface | `netstat -i` or `ifconfig` | Data Link (L2), Physical (L1) | Packet loss, errors, collisions | Helps detect LAN-level or NIC issues |

---

## Linux Networking Mastery Lab Project

### Objective
- Gain hands-on experience with Linux networking.  
- Troubleshoot issues across all OSI layers.  
- Understand layer interactions and the impact of configuration mistakes.  
- Be able to explain any networking issue clearly, layer by layer.

### Requirements
- 2–3 Linux VMs (Ubuntu/CentOS) or cloud instances.  
- Internet connectivity (optional, for external testing).  
- Root/sudo access on Linux machines.  
- Installed tools: `net-tools`, `iproute2`, `tcpdump`, `traceroute`, `curl`, `dnsutils`, `nmap`, `netcat`, `ethtool`.

---

### Phase 0: Lab Setup & Preparation

Goal: Prepare a controlled lab environment to simulate networking issues.

Steps:
1. Create Linux VMs  
   - VM1: Client machine  
   - VM2: Web Server (HTTP/HTTPS)  
   - VM3: Internal server (SSH, DB)

2. Assign static IP addresses:
```bash
sudo ip addr add 192.168.1.10/24 dev eth0   # Client
sudo ip addr add 192.168.1.20/24 dev eth0   # Web server
sudo ip addr add 192.168.1.30/24 dev eth0   # Internal server
```

3. Bring interfaces up:
```bash
sudo ip link set eth0 up
```

4. Install essential tools:
```bash
sudo apt update
sudo apt install net-tools iproute2 tcpdump traceroute curl dnsutils nmap netcat ethtool -y
```

Learning Outcome:
- Prepare isolated network environments.  
- Familiarity with basic Linux networking tools.  
- Layer 1 (Physical) and Layer 2 (Data Link) setup understanding.

---

### Phase 1: Basic Interface and IP Configuration

Goal: Understand network interfaces, IP addresses, and connectivity at the host level.

Steps & Exercises:
1. List all interfaces:
```bash
ip link show
ifconfig -a
```
- Output: Shows all interfaces, MAC addresses, and UP/DOWN state.  
- Layer: Physical (L1) + Data Link (L2)  
- Troubleshooting Use: Check NIC status and hardware connectivity.

2. Check assigned IP addresses:
```bash
ip addr show
```
- Output: Shows IPv4 & IPv6 addresses.  
- Layer: Network (L3)  
- Troubleshooting Use: Verify correct IP assignment.

3. Bring interfaces up/down:
```bash
sudo ip link set eth0 down
sudo ip link set eth0 up
```
- Use Case: Simulate NIC failures.

4. Ping loopback & local IP:
```bash
ping 127.0.0.1
ping 192.168.1.10
```
- Layer: Network (L3)  
- Use Case: Verify TCP/IP stack and interface connectivity.

Expected Outcome:
- Interfaces should be UP.  
- Ping to local IP succeeds; ping to remote VM should fail if network misconfigured.

---

### Phase 2: LAN Troubleshooting

Goal: Understand Layer 2 (Ethernet) and Layer 3 (IP) communication in local networks.

Steps:
1. Check ARP table:
```bash
arp -n
ip neigh show
```
- Maps IP → MAC addresses for local hosts.  
- Troubleshooting Use: Detect duplicate IPs or missing ARP entries.

2. Ping broadcast address:
```bash
ping -b 192.168.1.255
```
- Purpose: Test if LAN broadcast packets reach all hosts.  
- Layer: L2 (MAC broadcast), L3 (IP broadcast)

3. MTU testing:
```bash
ping -M do -s 1472 192.168.1.20
```
- Purpose: Detect fragmentation issues; important for large packets.

4. NIC speed & duplex:
```bash
ethtool eth0
```
- Use Case: Troubleshoot performance issues or collisions.

Expected Outcome:
- Broadcast ping reaches all hosts.  
- ARP entries correctly show MAC addresses.  
- MTU tests pass if network supports large packets.

---

### Phase 3: Routing & Inter-network Troubleshooting

Goal: Understand routing between multiple subnets and interfaces.

Steps:
1. Check routing table:
```bash
ip route show
```
- Shows default gateway and routes to subnets.

2. Add static route:
```bash
sudo ip route add 192.168.2.0/24 via 192.168.1.1
```
- Use Case: Route traffic to another subnet.

3. Traceroute to destination:
```bash
traceroute 192.168.2.10
```
- Purpose: Identify hops and routing failures.

4. Force ping via specific interface:
```bash
ping -I eth0 192.168.2.10
```

Learning Outcome:
- Identify incorrect gateways or routing misconfigurations.  
- Understand interaction of Layer 3 routing with Layer 2 frame delivery.

---

### Phase 4: DNS Troubleshooting

Goal: Learn hostname resolution and DNS issues.

Steps:
1. Test DNS resolution:
```bash
nslookup example.com
dig example.com
```

2. Check resolver configuration:
```bash
cat /etc/resolv.conf
```

3. Compare hostname vs IP ping:
```bash
ping example.com
ping 93.184.216.34
```

Expected Outcome:
- DNS resolves correctly; ping by hostname matches IP.  
- Layer 7 (DNS request) travels over UDP/TCP (L4) to resolve names.

---

### Phase 5: Transport Layer Troubleshooting

Goal: Diagnose TCP/UDP connectivity issues.

Steps:
1. List listening TCP/UDP ports:
```bash
ss -tulnp
netstat -tulnp
```

2. Test connectivity to port:
```bash
nc -zv 192.168.1.20 80
```

3. Check active TCP connections:
```bash
ss -antp
```

4. Simulate service down: Stop web server, check connectivity.

Expected Outcome:
- Verify service availability and port reachability.  
- Detect blocked ports or misconfigured services.

---

### Phase 6: Firewall & Security

Goal: Troubleshoot access control and filtering issues.

Steps:
1. List firewall rules:
```bash
sudo iptables -L -n -v
sudo nft list ruleset
```

2. Allow traffic to HTTP:
```bash
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

3. Test connectivity post-rule changes.

Learning Outcome:
- Identify firewall blocks at Layer 3/4.  
- Understand how filtering affects service reachability.

---

### Phase 7: Packet Capture & Analysis

Goal: Analyze traffic to detect low-level issues.

Steps:
1. Capture packets:
```bash
sudo tcpdump -i eth0 -c 20
```

2. Filter specific traffic:
```bash
sudo tcpdump -i eth0 tcp port 80
```

3. Analyze fields: Source/Dest IP, MAC, TCP flags.

Learning Outcome:
- Detect malformed packets, retransmissions, packet loss.  
- Correlate Layer 2 → Layer 4 encapsulation issues.

---

### Phase 8: IPv6 Troubleshooting

Goal: Understand IPv6 configuration and communication.

Steps:
1. Assign IPv6 address:
```bash
sudo ip -6 addr add 2001:db8::10/64 dev eth0
```

2. Ping IPv6 host:
```bash
ping6 2001:db8::20
```

3. Check neighbor table:
```bash
ip -6 neigh show
```

Learning Outcome:
- Detect misconfigured IPv6 addresses.  
- Understand IPv6 neighbor discovery (equivalent to ARP).

---

### Phase 9: End-to-End Service Testing

Goal: Verify connectivity from application to network layer.

Steps:
1. Test HTTP connectivity:
```bash
curl -I http://192.168.1.20
```

2. Simulate failures:
- Stop web server  
- Block port via firewall  
- Change IP/subnet

Learning Outcome:
- Correlate all layers to identify where communication fails.

---

### Phase 10: Advanced Troubleshooting Scenarios

Scenarios to Practice:
1. Multi-NIC server → route traffic to correct interface  
2. Firewall blocks HTTP/HTTPS → identify and fix rules  
3. Slow web service → use tcpdump to find retransmissions  
4. DNS resolution fails → check `/etc/resolv.conf`, test with `dig`  
5. IPv6 connectivity fails → check neighbor table & routing

Learning Outcome:
- Solve real-world networking problems.  
- Explain each failure layer by layer.

---

## Project Outcome

By completing all phases:
- You’ll be able to troubleshoot any Linux networking issue.  
- Understand all OSI layers, their interactions, and practical implications.  
- Be confident explaining network issues from Physical → Application layer.  
- Have a complete lab record to reference for learning or interviews.

