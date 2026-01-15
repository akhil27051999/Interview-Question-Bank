# Step 5: Network Security Tools

## Introduction

**Scenario:** You need to analyze network traffic, test SSL/TLS configurations, and perform basic port scanning to identify security issues. These tools are essential for security assessments and investigations.

**What we're learning:**

* Capture and analyze traffic using `tcpdump`
* Test SSL/TLS configurations using `sslscan`
* Perform basic port scanning using `netcat`
* Understand security implications of traffic patterns

**Expected Outcome:**

* ✅ Capture network traffic with tcpdump
* ✅ Test SSL/TLS configurations with sslscan
* ✅ Understand SSL/TLS versions and cipher suites
* ✅ Perform basic port scanning
* ✅ Analyze traffic for security issues

---

## Instructions

> **Note:** All commands in this lab are run in the Killercoda terminal.

---

## Environment Setup

### Install Required Tools

```bash
sudo apt update
sudo apt install -y tcpdump sslscan netcat-openbsd
```

### Create Practice Directory

```bash
export RANDOM_3CHAR=$(openssl rand -hex 3 | cut -c1-3)
export PRACTICE_DIR="$HOME/peachycloudsecurity-tools-${RANDOM_3CHAR}"
mkdir -p "$PRACTICE_DIR"
cd "$PRACTICE_DIR"
echo "Practice directory: $PRACTICE_DIR"
```

---

## tcpdump Basics

### What is tcpdump?

`tcpdump` captures raw network packets for troubleshooting and security analysis.

**Common Use Cases**

* Incident response
* Traffic inspection
* Detecting scans or exfiltration

**Basic Syntax**

```bash
tcpdump [options] [filter]
```

**Common Options**

* `-i` → Interface
* `-n` → No DNS resolution
* `-c` → Packet count
* `-w` → Write to file
* `-r` → Read from file

---

### View Available Interfaces

```bash
ip link show | grep -E "^[0-9]+:" | head -5
```

---

### Capture Sample Packets

```bash
sudo timeout 3 tcpdump -i any -n -c 5
```

---

### tcpdump Output Explained

```
10:30:45.123456 192.168.1.10.54321 > 8.8.8.8.53: UDP, length 32
```

* Timestamp
* Source IP & port
* Destination IP & port
* Protocol
* Packet size

---

### Common tcpdump Filters

```bash
# Protocols
sudo tcpdump icmp
sudo tcpdump tcp
sudo tcpdump udp

# Host
sudo tcpdump host 8.8.8.8

# Port
sudo tcpdump port 80
sudo tcpdump port 443

# Network
sudo tcpdump net 192.168.1.0/24
```

---

### Save and Read PCAP Files

```bash
sudo tcpdump -i any -c 50 -w capture.pcap
sudo tcpdump -r capture.pcap
```

---

## SSL/TLS Testing with sslscan

### What is sslscan?

Tests SSL/TLS configurations for:

* Protocol versions
* Cipher suites
* Known vulnerabilities

---

### Scan a Public Server

```bash
sslscan google.com | head -30
```

---

### SSL/TLS Versions

| Version       | Status       |
| ------------- | ------------ |
| SSLv2 / SSLv3 | ❌ Insecure   |
| TLS 1.0 / 1.1 | ⚠ Deprecated |
| TLS 1.2       | ✅ Secure     |
| TLS 1.3       | ✅ Best       |

---

### Cipher Suite Naming

Format:

```
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
```

* Key Exchange → ECDHE (preferred)
* Encryption → AES / CHACHA20
* Hash → SHA256 / SHA384

---

### Weak vs Strong Ciphers

❌ Weak:

* RC4
* 3DES
* NULL

✅ Strong:

* AES-GCM
* CHACHA20-POLY1305
* ECDHE

---

### Quick SSL Health Check

```bash
sslscan github.com | grep -E "Protocols|Preferred|Heartbleed" | head -10
```

---

## Port Scanning Basics

### What is Port Scanning?

Identifies open services and attack surface.

⚠ **Only scan systems you own or have permission for**

---

### Scan Ports with netcat

```bash
nc -zv google.com 80 443
```

---

### View Local Listening Ports

```bash
ss -tlnp | awk '{print $4}' | cut -d: -f2 | sort -u | head -10
```

---

### Port States

* **Open** → Service available
* **Closed** → No service
* **Filtered** → Firewall blocking

Security Note: Open ports = attack surface

---

## Traffic Analysis Basics

### What to Look For

🚩 Suspicious Patterns:

* Unusual ports
* Repeated connection attempts
* Large outbound transfers
* Cleartext credentials

✅ Normal Patterns:

* HTTPS traffic
* Established connections
* Regular DNS queries

---

### View Active Connections

```bash
ss -tunp | head -10
```

---

### Connection State Summary

```bash
ss -ant | awk '{print $1}' | sort | uniq -c
```

---

## Practice Exercise: Security Scan Script

```bash
cat > security-scan.sh << 'EOF'
#!/bin/bash

echo "=== Network Security Assessment ==="
echo ""

echo "1. Listening Ports:"
ss -tlnp | head -5

echo ""
echo "2. Active Connections:"
ss -tn | head -5

echo ""
echo "3. SSL/TLS Test (google.com):"
sslscan google.com | grep -E "Protocols|Preferred" | head -3
EOF

chmod +x security-scan.sh
./security-scan.sh
```

---

## Key Learning Points

* `tcpdump` → Packet-level traffic visibility
* `sslscan` → TLS & cipher security checks
* Netcat → Lightweight port scanning
* TLS 1.2/1.3 + strong ciphers are mandatory
* Traffic patterns reveal attacks

---

## Security Relevance

* Detect scanning & lateral movement
* Identify weak encryption
* Reduce attack surface
* Support incident response & forensics

---

## Summary

You now know how to:

* Capture and inspect live traffic
* Validate SSL/TLS security
* Identify exposed services
* Analyze traffic from a security perspective

This is **core SOC, Cloud Security, and IR skillset**.
