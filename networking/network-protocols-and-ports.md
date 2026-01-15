# Step 4: Network Protocols and Ports

## Introduction

**Scenario:** You need to understand network protocols (TCP, UDP, HTTP, HTTPS, DNS) and ports to analyze network traffic, identify services, and detect suspicious activity. Understanding protocols is essential for security analysis.

**What we’re learning:**

* TCP vs UDP
* Common ports and services
* HTTP / HTTPS
* DNS protocol
* Service identification by port
* Protocol security implications

## Expected Outcomes

* ✅ Understand TCP and UDP protocols
* ✅ Know common ports and their services
* ✅ Understand HTTP and HTTPS
* ✅ Understand DNS protocol
* ✅ Identify services by port numbers
* ✅ Understand protocol security implications

---

## Instructions

> **Note:** All commands in this lab are run in the **Killercoda terminal**.

---

## TCP vs UDP

### TCP (Transmission Control Protocol)

```bash
cat << 'EOF'
TCP (Transmission Control Protocol)

Characteristics:
- Connection-oriented (establishes connection first)
- Reliable (guarantees delivery)
- Ordered (data arrives in order)
- Error-checked (retransmits lost packets)
- Slower than UDP
- Uses handshake (SYN, SYN-ACK, ACK)

Use Cases:
- Web browsing (HTTP/HTTPS)
- Email (SMTP)
- File transfers (FTP)
- Remote access (SSH)

Analogy:
- Like a phone call: both sides confirm communication
EOF
```

### UDP (User Datagram Protocol)

```bash
cat << 'EOF'
UDP (User Datagram Protocol)

Characteristics:
- Connectionless (no connection established)
- Unreliable (no delivery guarantee)
- Unordered (data may arrive out of order)
- No error checking
- Faster than TCP
- No handshake

Use Cases:
- DNS queries
- Video streaming
- Online gaming
- VoIP
- DHCP

Analogy:
- Like shouting: you send it and hope it's heard
EOF
```

### TCP vs UDP Comparison

```bash
cat << 'EOF'
TCP vs UDP Comparison

TCP:
✓ Reliable delivery
✓ Ordered data
✓ Error checking
✗ Slower
✗ More overhead

Use TCP when data integrity is critical.

UDP:
✓ Fast
✓ Low overhead
✗ No delivery guarantee
✗ No ordering

Use UDP when speed matters more than reliability.
EOF
```

### View Active Connections

```bash
# View TCP connections
ss -tn | head -10

# View UDP connections
ss -un | head -10
```

---

## Understanding Ports

### What Are Ports?

```bash
cat << 'EOF'
PORTS

Definition:
- Numbers that identify specific services on a device

Range:
- 0–65535

Format:
- IP_ADDRESS:PORT (e.g., 192.168.1.1:80)

Port Categories:
- Well-known: 0–1023 (require root privileges)
- Registered: 1024–49151
- Dynamic/Ephemeral: 49152–65535

Common Ports:
- 22  : SSH
- 80  : HTTP
- 443 : HTTPS
- 53  : DNS
- 25  : SMTP
- 3306: MySQL
- 3389: RDP
EOF
```

### View Listening Ports

```bash
ss -tlnp 2>/dev/null | head -10 || netstat -tlnp 2>/dev/null | head -10
```

### Identify Services by Port

```bash
cat << 'EOF'
COMMON PORTS AND SERVICES

Web:
- 80   : HTTP
- 443  : HTTPS
- 8080 : HTTP alternate

Remote Access:
- 22   : SSH
- 23   : Telnet (insecure)
- 3389 : RDP

Email:
- 25   : SMTP
- 110  : POP3
- 143  : IMAP
- 587  : SMTP (encrypted submission)

Databases:
- 3306 : MySQL
- 5432 : PostgreSQL
- 1433 : MSSQL
- 27017: MongoDB

DNS:
- 53   : DNS (TCP/UDP)

File Transfer:
- 21   : FTP
- 22   : SFTP
- 69   : TFTP
EOF
```

### Check a Specific Port

```bash
ss -tlnp | grep ":22 " || echo "SSH (port 22) not listening"
```

---

## HTTP and HTTPS

### HTTP

```bash
cat << 'EOF'
HTTP (Hypertext Transfer Protocol)

Port: 80
Protocol: TCP
Purpose: Transfers web pages (unencrypted)

Security Issues:
- Plaintext data
- Can be intercepted
- Vulnerable to MITM attacks

Example Request:
GET /index.html HTTP/1.1
Host: example.com
EOF
```

### HTTPS

```bash
cat << 'EOF'
HTTPS (HTTP Secure)

Port: 443
Protocol: TCP
Purpose: Encrypted web traffic

Security Benefits:
- Data encryption
- Server authentication
- Data integrity
- Prevents MITM attacks

Modern Standard:
- All web traffic should use HTTPS
EOF
```

### Test HTTP / HTTPS

```bash
curl -I http://httpbin.org/get 2>/dev/null | head -5 || echo "HTTP test failed"

curl -I https://www.google.com 2>/dev/null | head -5
```

---

## DNS Protocol

### Understanding DNS

```bash
cat << 'EOF'
DNS (Domain Name System)

Port: 53 (TCP & UDP)
Purpose: Converts domain names to IP addresses

Record Types:
- A     : IPv4 address
- AAAA  : IPv6 address
- CNAME : Alias
- MX    : Mail exchange
- TXT   : Verification / metadata

Security Risks:
- DNS spoofing
- DNS tunneling
- DNS amplification attacks
EOF
```

### DNS Queries

```bash
# Basic DNS query
dig google.com +short || host google.com | head -1

# Detailed DNS query
dig google.com | head -15

# Record types
echo "A record:"; dig google.com A +short | head -1
echo "AAAA record:"; dig google.com AAAA +short | head -1 || echo "No IPv6"
```

---

## Port Scanning Basics

```bash
cat << 'EOF'
PORT SCANNING

Purpose:
- Identify open ports and services

Security Perspective:
- Attackers scan for targets
- Defenders scan for exposure

Common Tools:
- nmap
- netcat (nc)
- telnet

Open ports = Attack surface
EOF
```

### Basic Port Test

```bash
timeout 2 bash -c 'echo > /dev/tcp/google.com/80' && echo "Port 80 open" || echo "Port blocked or filtered"
```

### View Local Listening Ports

```bash
echo "Listening ports:"; ss -tlnp | awk '{print $4}' | cut -d: -f2 | sort -u | head -10
```

---

## Protocol Security

```bash
cat << 'EOF'
INSECURE PROTOCOLS (Avoid):
- HTTP (80)
- FTP (21)
- Telnet (23)

SECURE PROTOCOLS (Use):
- HTTPS (443)
- SSH (22)
- SFTP (22)
- SMTPS (465/587)

Best Practices:
- Use encryption
- Close unused ports
- Restrict access with firewalls
- Monitor and scan regularly
EOF
```

---

## Practice Exercise

### Protocol Analysis Script

```bash
cat > protocol-check.sh << 'EOF'
#!/bin/bash

echo "=== Protocol and Port Analysis ==="
echo
echo "1. Listening TCP Ports:"
ss -tlnp | head -5 || echo "ss not available"

echo
echo "2. Active TCP Connections:"
ss -tn | head -5

echo
echo "3. DNS Test:"
host google.com > /dev/null 2>&1 && echo "✓ DNS working" || echo "✗ DNS failed"

echo
echo "4. HTTPS Test:"
curl -I https://www.google.com > /dev/null 2>&1 && echo "✓ HTTPS working" || echo "✗ HTTPS failed"
EOF

chmod +x protocol-check.sh
./protocol-check.sh
```

---

## Summary

You’ve learned:

* TCP vs UDP behavior
* Common ports and services
* HTTP vs HTTPS security
* DNS protocol and risks
* Port scanning basics
* Protocol security best practices

These skills are **critical for SOC analysis, cloud security, and real-world incident investigation**.

# Script

## Check Protocol Script
```sh
ubuntu:~$ vi protocol-check.sh 
ubuntu:~$ cat protocol-check.sh 
#!/bin/bash
echo "=== Protocol and Port Analysis ==="
echo ""
echo "1. Listening TCP Ports:"
ss -tlnp 2>/dev/null | head -5 || echo "ss not available"
echo ""
echo "2. Active TCP Connections:"
ss -tn | head -5
echo ""
echo "3. DNS Test:"
host google.com > /dev/null 2>&1 && echo "✓ DNS working" || echo "✗ DNS failed"
echo ""
echo "4. HTTPS Test:"
curl -I https://www.google.com > /dev/null 2>&1 && echo "✓ HTTPS working" || echo "✗ HTTPS failed"

ubuntu:~$ chmod +x protocol-check.sh
ubuntu:~$ ./protocol-check.sh
=== Protocol and Port Analysis ===

1. Listening TCP Ports:
State  Recv-Q Send-Q Local Address:Port  Peer Address:PortProcess                                                
LISTEN 0      128          0.0.0.0:40200      0.0.0.0:*    users:(("kc-terminal",pid=1281,fd=12))                
LISTEN 0      511          0.0.0.0:40205      0.0.0.0:*    users:(("node",pid=1232,fd=18))                       
LISTEN 0      4096         0.0.0.0:22         0.0.0.0:*    users:(("sshd",pid=1199,fd=3),("systemd",pid=1,fd=93))
LISTEN 0      4096       127.0.0.1:33677      0.0.0.0:*    users:(("containerd",pid=625,fd=7))                   

2. Active TCP Connections:
State Recv-Q Send-Q Local Address:Port  Peer Address:Port Process
ESTAB 0      0         172.30.1.2:40205 10.244.4.181:42778       
ESTAB 0      0         172.30.1.2:40200 10.244.3.183:39122       

3. DNS Test:
✓ DNS working

4. HTTPS Test:
✓ HTTPS working
ubuntu:~$ 
```
