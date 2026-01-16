## Validate IP address from a File

**Purpose:**
  1. Extract IPv4 addresses from text files or logs.
  2. Remove duplicates, count occurrences, and validate real IPv4 ranges.
  3. Generate random IP addresses for testing or simulation.
  4. Useful for log analysis, network monitoring, and DevOps automation.

**Use Cases:**
  - `Log analysis`: Extract source/destination IPs from firewall, web, or system logs.
  - `Security monitoring`: Identify repeated or suspicious IP addresses.
  - `Automation & testing`: Generate random IPs for scripts, network simulations, or mock data.
  - `DevOps scripting`: Quickly summarize network traffic patterns from multiple servers.
    
```sh
ubuntu:~$ cat ips.txt 
User connected from 192.168.1.10
Failed login attempt from 10.0.0.5
Ping request from 172.16.5.23
Suspicious traffic detected from 8.8.8.8
Another request from 192.168.1.10
Invalid access from 203.0.113.45
Server contacted 54.85.23.111 for updates
Random text without IP

# ---------------- Basic extraction (most common) ----------------
ubuntu:~$ grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' ips.txt
192.168.1.10
10.0.0.5
172.16.5.23
8.8.8.8
192.168.1.10
203.0.113.45
54.85.23.111

# ---------------- Extract UNIQUE IP addresses only ----------------
ubuntu:~$ grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' ips.txt | sort -u       
10.0.0.5
172.16.5.23
192.168.1.10
203.0.113.45
54.85.23.111
8.8.8.8

# ---------------- Count how many times each IP appears ----------------
ubuntu:~$ grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' ips.txt | sort | uniq -c
      1 10.0.0.5
      1 172.16.5.23
      2 192.168.1.10
      1 203.0.113.45
      1 54.85.23.111
      1 8.8.8.8

# ---------------- Validate only REAL IPv4 addresses (0–255) ----------------
ubuntu:~$ grep -oE '\b((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\b' ips.txt
192.168.1.10
10.0.0.5
172.16.5.23
8.8.8.8
192.168.1.10
203.0.113.45
54.85.23.111

# ---------------- (Optional) Generate a file with random IPs automatically ----------------
ubuntu:~$ for i in {1..10}; do
  echo "$((RANDOM%256)).$((RANDOM%256)).$((RANDOM%256)).$((RANDOM%256))"
done > random_ips.txt

ubuntu:~$ ls   
backup_status.txt  filesystem  ips.txt  random_ips.txt  validate-ip.sh

ubuntu:~$ cat random_ips.txt 
61.225.91.79
255.231.194.222
185.107.83.15
17.89.204.229
52.225.157.57
58.76.65.183
153.48.6.176
134.191.215.40
8.17.93.93
140.205.225.213
```
