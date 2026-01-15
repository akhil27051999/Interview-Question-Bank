# Step 7: Network Concepts Discussion and Q&A

## Introduction

### Scenario
Understanding networking concepts deeply helps you troubleshoot issues, analyze attacks, and design secure networks.  
This section covers important questions and concepts that security engineers should understand.

### What We're Learning
- Answer common networking questions
- Understand protocol layers
- Understand handshake processes (TCP and TLS)
- Understand web access workflow
- Deepen your understanding of networking fundamentals

### Expected Outcome
- ✅ Understand which OSI layers protocols operate at  
- ✅ Understand handshake processes (TCP and TLS)  
- ✅ Understand difference between mTLS and SSL/TLS  
- ✅ Understand how web access works  
- ✅ Answer common networking interview questions  

### Instructions
All commands in this lab are run in the Killercoda terminal.

---

## Common Networking Questions

### Q1: On which port does ping work?
```bash
cat << 'EOF'
ANSWER:
Ping does NOT work on a port. Why:
- Ping uses ICMP (Internet Control Message Protocol)
- ICMP is a Layer 3 (Network) protocol
- ICMP does NOT use ports (ports are Layer 4 - Transport)
- ICMP uses types and codes instead of ports

ICMP Types:
- Type 8: Echo Request (ping request)
- Type 0: Echo Reply (ping reply)
- Type 3: Destination Unreachable
- Type 11: Time Exceeded (traceroute)

Ports are for:
- TCP and UDP (Transport layer protocols)
- Not for ICMP (Network layer protocol)

Analogy:
- Ports = apartment numbers (TCP/UDP)
- ICMP = building intercom (no apartment number needed)
EOF
```

### test ping
```sh
ping -c 2 8.8.8.8
```

## Q2: HTTPS works on which OSI layer?
```sh
cat << 'EOF'
ANSWER:
HTTPS works at multiple OSI layers.

Primary Layer: Application Layer (Layer 7)
- HTTPS is an application protocol
- Used by web browsers and web servers
- Defines how web pages are transferred

HTTPS involves multiple layers:
- Layer 7 (Application): HTTP protocol
- Layer 6 (Presentation): SSL/TLS encryption/decryption
- Layer 4 (Transport): TCP protocol (port 443)
- Layer 3 (Network): IP protocol
- Layer 2 (Data Link): Ethernet frames
- Layer 1 (Physical): Cables/wireless

Key Point:
- HTTPS = HTTP (Layer 7) + SSL/TLS (Layer 6)
- Encryption happens at Presentation layer (Layer 6)
- Protocol itself is Application layer (Layer 7)
EOF
```

### Test HTTPS:
```sh
curl -I https://www.google.com 2>/dev/null | head -3
```

## Q3: SSL/TLS works on which OSI layer?
```sh
cat << 'EOF'
ANSWER:
SSL/TLS works at Presentation Layer (Layer 6)

Why:
- SSL/TLS handles encryption/decryption
- Converts data between formats (encrypted ↔ decrypted)
- Provides data translation and encryption services

Layer Breakdown:
- Layer 7 (Application): HTTP protocol (unencrypted)
- Layer 6 (Presentation): SSL/TLS encryption wraps HTTP
- Layer 4 (Transport): TCP carries encrypted data
- Layer 3 (Network): IP routing

What SSL/TLS does:
- Encrypts application data (Layer 7)
- Provides secure communication channel
- Handles certificate authentication
- Manages encryption keys

Result:
- HTTPS = HTTP (Layer 7) encrypted by SSL/TLS (Layer 6)
- Data encrypted at Layer 6, transmitted via Layer 4 (TCP)
EOF
```

### View SSL/TLS in action:
```sh
echo | openssl s_client -connect google.com:443 -servername google.com 2>/dev/null | grep -E "(Protocol|Cipher)" | head -3
```
## SSL/TLS vs mTLS
### SSL/TLS (One-Way Authentication)
```sh
cat << 'EOF'
How it works:
1. Client connects to server
2. Server presents certificate
3. Client verifies server's certificate
4. Client trusts server (one-way)
5. Encrypted communication established

Authentication:
- Server authenticates to client
- Client does NOT authenticate to server
- Client is anonymous to server

Use Case:
- Web browsing (HTTPS)
EOF
```

### mTLS (Mutual TLS / Two-Way Authentication)
```sh
cat << 'EOF'
How it works:
1. Client connects to server
2. Server presents certificate
3. Client verifies server's certificate
4. Client also presents its certificate
5. Server verifies client's certificate
6. Both parties authenticate (two-way)
7. Encrypted communication established

Authentication:
- Server authenticates to client
- Client also authenticates to server

Use Case:
- API-to-API communication
- Microservices security
- Zero Trust architectures
EOF
```

### Comparison
```sh
cat << 'EOF'
SSL/TLS (One-Way):
- Server has certificate
- Client verifies server
- Client has no certificate
- Server doesn't verify client
- Use: Web browsing, general HTTPS

mTLS (Two-Way):
- Server has certificate
- Client verifies server
- Client has certificate
- Server verifies client
- Use: API security, microservices, zero trust

Key Difference:
- TLS: Only server authenticates
- mTLS: Both server AND client authenticate

Security Benefit:
- mTLS provides stronger authentication
- Prevents unauthorized clients
- Better for service-to-service communication
EOF
```

## TCP Three-Way Handshake
```sh
cat << 'EOF'
Purpose: Establish reliable connection before data transfer

Step 1: SYN (Synchronize)
- Client sends SYN packet to server
- Client: "I want to connect, my sequence number is X"
- Client state: SYN_SENT

Step 2: SYN-ACK (Synchronize-Acknowledge)
- Server responds with SYN-ACK
- Server: "I acknowledge X, my sequence number is Y"
- Server state: SYN_RECEIVED

Step 3: ACK (Acknowledge)
- Client sends ACK
- Connection established
- Both states: ESTABLISHED

Total: 3 packets exchanged
EOF
```

### Visualize TCP handshake:
```sh
cat << 'EOF'
Client                    Server
  |                         |
  |-------- SYN ----------->|
  |                         |
  |<----- SYN-ACK ----------|
  |                         |
  |-------- ACK ----------->|
  |                         |
  |    CONNECTION ESTABLISHED|
EOF
```

### View TCP connections:
```sh
ss -tn | head -5
```

## TLS Handshake:

```sh
cat << 'EOF'
Purpose: Establish encrypted connection and verify server

Step 1: ClientHello
- Client sends supported TLS versions and cipher suites
- Sends random number (ClientRandom)

Step 2: ServerHello
- Server selects version and cipher
- Sends certificate
- Sends random number (ServerRandom)

Step 3: Key Exchange
- Client verifies server certificate
- Generates pre-master secret
- Encrypts pre-master with server's public key
- Sends encrypted pre-master

Step 4: Session Keys
- Both sides derive session keys from ClientRandom, ServerRandom, pre-master

Step 5: Ready
- Exchange "Change Cipher Spec" and "Finished"
- Encrypted communication begins
EOF
```

### Visualize TLS handshake:

```sh
cat << 'EOF'
Client                    Server
  |                         |
  |----- Client Hello ----->|
  |                         |
  |<---- Server Hello ------|
  |                         |
  |-- Encrypted Key ------->|
  |                         |
  |<-- Change Cipher -------|
  |-- Change Cipher ------->|
  |                         |
  |<-- Finished ------------|
  |-- Finished ----------->|
  |                         |
  |<=== ENCRYPTED DATA ====>|
EOF
```

### Test TLS handshake:
```sh
echo | timeout 3 openssl s_client -connect google.com:443 -servername google.com 2>/dev/null | grep -E "(Protocol|Cipher|Verify)" | head -5
```

## Symmetric vs Asymmetric Encryption

### Symmetric Encryption

```sh
cat << 'EOF'
- Uses the SAME key for encryption and decryption
- Fast and efficient for large data
- Key must be shared securely

Example: AES, 3DES, ChaCha20
EOF
```

### Asymmetric Encryption

```sh
cat << 'EOF'
- Uses a pair of keys: public and private
- Public key can be shared
- Private key must be secret
- Used for key exchange and digital signatures

Example: RSA, ECC
EOF
```

### TLS Combination
```sh
cat << 'EOF'
TLS Handshake (Asymmetric):
- Exchanges session key securely using server's public key

TLS Data Transfer (Symmetric):
- Encrypts actual data with session key (AES)
- Fast and efficient
EOF
```

### Protocol Layer Summary
```sh
cat << 'EOF'
Layer 7 (Application): HTTP, HTTPS, DNS, SSH, FTP
Layer 6 (Presentation): SSL/TLS, data compression, translation
Layer 4 (Transport): TCP, UDP (ports)
Layer 3 (Network): IP, ICMP, routing protocols
Layer 2 (Data Link): Ethernet, MAC, ARP
Layer 1 (Physical): Cables, wireless, electrical/optical signals
EOF
```

### Key Learning Points
- Ping: ICMP (Layer 3), no ports
- HTTPS: Application Layer (7) with Presentation Layer (6) encryption
- SSL/TLS: Presentation Layer (6)
- TLS vs mTLS: One-way vs two-way authentication
- TCP: Three-way handshake
- TLS handshake: Establishes encryption and verifies server
- Symmetric: Fast, same key
- Asymmetric: Public/private, secure key exchange
- Public Key: Can share
- Private Key: Must keep secret
- TLS uses both: Asymmetric handshake, symmetric data
- Web access flow: DNS → TCP → TLS → HTTP Request → HTTP Response → Rendering
  
### Security Relevance
- Layer knowledge helps detect attacks
- Handshake understanding identifies anomalies
- mTLS offers stronger authentication
- Cipher analysis reveals weak configurations
- Understanding traffic flow aids network troubleshooting

### Practice Questions
```sh
cat << 'EOF'
1. Which port does ping use? None - ICMP, Layer 3
2. HTTPS OSI Layer? Application Layer 7 (encryption at Layer 6)
3. SSL/TLS OSI Layer? Presentation Layer 6
4. TLS vs mTLS? TLS=server auth, mTLS=both auth
5. TCP handshake steps? 3 (SYN, SYN-ACK, ACK)
6. Symmetric encryption? Same key encrypt/decrypt
7. Asymmetric encryption? Public/private keys
8. Public vs Private key? Public shared, Private secret
9. TLS encryption types? Asymmetric for handshake, symmetric for data
10. Web access process? DNS → TCP → TLS → HTTP Request → HTTP Response → Render
EOF
```
