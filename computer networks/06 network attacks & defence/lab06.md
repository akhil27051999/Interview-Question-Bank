# Step 6: Network Attacks and Defense

## Introduction

### Scenario
You need to understand common network attacks (MITM, ARP spoofing, port scanning, DNS spoofing) to detect and defend against them.

### What We’re Learning
Understand attack methods and learn how to defend, monitor, and detect incidents.

### Expected Outcome

- ✅ Understand MITM attacks
- ✅ Understand ARP spoofing and detection
- ✅ Understand port scanning attacks
- ✅ Understand DNS spoofing attacks
- ✅ Learn defense strategies
- ✅ Learn attack detection methods

### Instructions
All commands are run in the Killercoda terminal.  
Focus is on learning attacks for defense purposes.

---

## Man-in-the-Middle (MITM) Attacks

### Concept

```bash
cat << 'EOF'
MITM (Man-in-the-Middle) Attack:
- Attacker intercepts communication between two parties
- Can read, modify, or inject data
How it works:
1. Attacker positions between victim and target
2. Intercepts all traffic
3. Reads/modifies data
4. Forwards traffic (victim unaware)
Examples: ARP spoofing, DNS spoofing, rogue APs, SSL/TLS manipulation
EOF
```

### Flow of MITM Attack

```sh
cat << 'EOF'
MITM Attack Flow:
Normal: Victim <--> Router <--> Internet <--> Server
MITM:  Victim <--> Attacker <--> Router <--> Internet <--> Server
Attacker can:
- Read unencrypted traffic (HTTP)
- Modify data in transit
- Inject malicious content
- Steal credentials
Defense:
- Use HTTPS (encrypted traffic)
- Verify SSL/TLS certificates
- Certificate pinning
- Monitor for ARP anomalies
EOFcat << 'EOF'
ARP Spoofing (ARP Poisoning):
- Attacker sends fake ARP messages
- Maps attacker's MAC to victim's IP
- Redirects victim traffic through attacker
How it works:
1. Attacker sends ARP reply: "IP X is at MAC Y"
2. Victim updates ARP table
3. Traffic flows through attacker
4. Attacker reads/modifies traffic
Why it works:
- ARP has no authentication
- Devices trust ARP replies
Goals:
- MITM, network disruption, session hijacking
EOF

```

### Check HTTPS Usage

```sh
curl -I https://www.google.com 2>/dev/null | head -3
echo "HTTPS encrypts traffic, preventing MITM from reading data"
```

## ARP Spoofing

### Concept

```sh
cat << 'EOF'
ARP Spoofing (ARP Poisoning):
- Attacker sends fake ARP messages
- Maps attacker's MAC to victim's IP
- Redirects victim traffic through attacker
How it works:
1. Attacker sends ARP reply: "IP X is at MAC Y"
2. Victim updates ARP table
3. Traffic flows through attacker
4. Attacker reads/modifies traffic
Why it works:
- ARP has no authentication
- Devices trust ARP replies
Goals:
- MITM, network disruption, session hijacking
EOF
```

### View ARP Table

```sh
ip neigh show | head -5
```

### Detect ARP Spoofing

```sh
cat << 'EOF'
Detecting ARP Spoofing:
Signs:
1. Multiple MACs for same IP
2. Unexpected MAC changes
3. Duplicate IPs
Detection:
- Monitor ARP table: watch -n1 'ip neigh show'
- Check duplicates: ip neigh show | awk '{print $1}' | sort | uniq -d
- Compare MACs with known good devices
EOF
```

### Check for Duplicate IPs
```sh
ip neigh show | awk '{print $1}' | sort | uniq -d
```

### ARP Spoofing Defense Strategies
```sh
cat << 'EOF'
ARP Spoofing Defense:
1. Static ARP Entries: arp -s <IP> <MAC>
2. ARP Monitoring Tools: arpwatch, arp-scan
3. Network Segmentation: VLANs, limit broadcast domains
4. Encryption: HTTPS, VPN
5. Port Security: Switchport security to bind MAC to port
EOF
```

## Port Scanning Attacks
### Concept

```sh
cat << 'EOF'
Port Scanning Attack:
- Attacker scans for open ports and services
- Types: Stealth scan, Full scan, Service version detection, OS detection
Defense:
- Firewall rules (close unnecessary ports)
- IDS/IPS
- Rate limiting
- Honeypots
EOF
```

### Detect Port Scanning Patterns

```sh
cat << 'EOF'
Detecting Port Scans:
Signs:
1. Multiple connection attempts to different ports
2. Rapid connections from a single source
3. Connections to closed ports
Detection Methods:
- Firewall logs
- IDS/IPS systems
- Connection rate analysis
EOF
```

### View Suspicious Connections
```sh
ss -tn | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head -5
```

## DNS Spoofing

### Concept

```sh
cat << 'EOF'
DNS Spoofing (DNS Poisoning):
- Attacker provides fake DNS responses
- Redirects traffic to malicious sites
How it works:
1. Victim queries "example.com"
2. Attacker returns malicious IP
3. Victim connects to attacker's server
Defense:
- Trusted DNS servers (8.8.8.8, 1.1.1.1)
- DNS over HTTPS (DoH) or TLS (DoT)
- Verify SSL/TLS certificates
- Monitor DNS queries
EOF
```

### Test DNS Resolution
```sh
echo "Testing DNS resolution:"
for site in google.com github.com; do
  RESOLVED=$(host $site 2>/dev/null | grep "has address" | head -1)
  echo "$site: $RESOLVED"
done
```

## Defense Strategies

```sh
cat << 'EOF'
Network Defense Strategies:
1. Encryption: HTTPS, VPN, encrypt sensitive data
2. Network Segmentation: Subnets, firewall rules, limit lateral movement
3. Monitoring: Log traffic, IDS/IPS, anomaly alerts
4. Access Control: Close unused ports, strong authentication, least privilege
5. Regular Assessment: Authorized port scans, vulnerability scans, SSL/TLS testing, audits
EOF
```

### Defense Checklist Script

```sh
cat > defense-checklist.sh << 'EOF'
#!/bin/bash
echo "=== Network Defense Checklist ==="
echo ""
echo "1. Open Ports:" $(ss -tlnp 2>/dev/null | wc -l) "ports listening (should be minimal)"
echo ""
echo "2. HTTPS Usage:" $(curl -I https://www.google.com > /dev/null 2>&1 && echo "✓ HTTPS working" || echo "✗ HTTPS failed")
echo ""
echo "3. DNS Resolution:" $(host google.com > /dev/null 2>&1 && echo "✓ DNS working" || echo "✗ DNS failed")
echo ""
echo "4. ARP Table:" $(ip neigh show | wc -l) "ARP entries (monitor for changes)"
EOF

chmod +x defense-checklist.sh
./defense-checklist.sh
```

## Practice Exercise: Attack Detection Script

```sh
cat > detect-attacks.sh << 'EOF'
#!/bin/bash
echo "=== Attack Detection Checks ==="
echo ""
echo "1. ARP Table Duplicates:"
DUPLICATES=$(ip neigh show | awk '{print $1}' | sort | uniq -d)
if [ -z "$DUPLICATES" ]; then
  echo "✓ No duplicate IPs in ARP table"
else
  echo "⚠ Potential ARP spoofing: $DUPLICATES"
fi
echo ""
echo "2. Suspicious Connections:"
ss -tn | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head -3
echo ""
echo "3. DNS Verification:"
host google.com | grep "has address" | head -1
EOF

chmod +x detect-attacks.sh
./detect-attacks.sh
```

### Key Learning Points

- MITM Attack: Intercepts communication between two parties
- ARP Spoofing: Fake ARP messages redirect traffic
- Port Scanning: Maps attack surface via open ports
- DNS Spoofing: Provides fake DNS responses
- Defense: Encryption, monitoring, segmentation, access control
- Detection: Monitor ARP table, connection patterns, DNS queries
- Best Practices: Use encrypted protocols, verify certificates, limit open ports

# Script

## Network Attack Detection Checks 

```sh
ubuntu:~$ cat detect-attacks.sh 
#!/bin/bash
echo "=== Attack Detection Checks ==="
echo ""
echo "1. ARP Table Duplicates:"
DUPLICATES=$(ip neigh show | awk '{print $1}' | sort | uniq -d)
if [ -z "$DUPLICATES" ]; then
    echo "✓ No duplicate IPs in ARP table"
else
    echo "⚠ Potential ARP spoofing: $DUPLICATES"
fi
echo ""
echo "2. Suspicious Connections:"
ss -tn | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head -3
echo ""
echo "3. DNS Verification:"
host google.com | grep "has address" | head -1

ubuntu:~$ chmod +x detect-attacks.sh
ubuntu:~$ ./detect-attacks.sh
=== Attack Detection Checks ===

1. ARP Table Duplicates:
✓ No duplicate IPs in ARP table

2. Suspicious Connections:
      1 Address
      1 10.244.4.181
      1 10.244.3.183

3. DNS Verification:
google.com has address 74.125.130.100
```
