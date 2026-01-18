## The Forensic Analyst: Log Mining
**A web server in your network has reported suspicious activity. You have been handed a raw access.log file and need to identify potential security threats quickly.**

**In this lab, you will step into the shoes of a Security Engineer. You will move beyond simple text searching and learn how to perform high-speed data extraction.**

### Your Toolkit:
  - `grep`: The "Global Regular Expression Print." You'll use this to filter out the noise and find specific error codes.
  - `awk`: A powerful pattern-scanning and processing language. You'll use this to treat your text file like a database and extract specific columns (like IP addresses).
  - `Pipes (|)`: To chain these tools together for complex analysis.
    
### Your Objectives:
  - Isolate all failed login attempts.
  - Extract a list of unique IP addresses that are hitting the server.
  - Calculate which IP address has made the most requests.

### Step 1: Preparing the Log File
**Before we analyze, we need data. We will create a simulated access.log representing traffic to a website.**

```sh
ubuntu:~$ cat << 'EOF' > /var/log/access.log
192.168.1.1 - [10/Oct/2023:13:55:36] "GET /index.html" 200
10.0.0.5 - [10/Oct/2023:13:56:01] "POST /login" 401
192.168.1.1 - [10/Oct/2023:13:56:10] "GET /styles.css" 200
172.16.0.45 - [10/Oct/2023:13:57:05] "GET /admin" 403
10.0.0.5 - [10/Oct/2023:13:57:12] "POST /login" 401
10.0.0.5 - [10/Oct/2023:13:58:01] "POST /login" 200
192.168.1.20 - [10/Oct/2023:13:59:45] "GET /logo.png" 200
EOF

ubuntu:~$ cat /var/log/access.log 
192.168.1.1 - [10/Oct/2023:13:55:36] "GET /index.html" 200
10.0.0.5 - [10/Oct/2023:13:56:01] "POST /login" 401
192.168.1.1 - [10/Oct/2023:13:56:10] "GET /styles.css" 200
172.16.0.45 - [10/Oct/2023:13:57:05] "GET /admin" 403
10.0.0.5 - [10/Oct/2023:13:57:12] "POST /login" 401
10.0.0.5 - [10/Oct/2023:13:58:01] "POST /login" 200
192.168.1.20 - [10/Oct/2023:13:59:45] "GET /logo.png" 200
```

### Step 2: Finding Failures with Grep
**grep is perfect for finding specific lines. Let's find all "Unauthorized" (401) or "Forbidden" (403) attempts.**

  1. Search for 401 errors
  2. Use Extended Regex to find both 401 and 403 errors
  3. Count the failures

```sh
ubuntu:~$ grep "401" /var/log/access.log
10.0.0.5 - [10/Oct/2023:13:56:01] "POST /login" 401
10.0.0.5 - [10/Oct/2023:13:57:12] "POST /login" 401

ubuntu:~$ grep -E "401|403" /var/log/access.log
10.0.0.5 - [10/Oct/2023:13:56:01] "POST /login" 401
172.16.0.45 - [10/Oct/2023:13:57:05] "GET /admin" 403
10.0.0.5 - [10/Oct/2023:13:57:12] "POST /login" 401

ubuntu:~$ grep -cE "401|403" /var/log/access.log
3
```

### Step 3: Isolating Data with Awk
**While grep finds lines, awk finds columns. By default, awk uses spaces as separators.**
  1. Extract only the IP addresses (Column 1)
  2. Extract IPs and their Status Codes (Column 1 and Column 6)
  3. Filter and Extract: Find lines with "401" and print only the IP
     
```
ubuntu:~$ awk '{print $1}' /var/log/access.log
192.168.1.1
10.0.0.5
192.168.1.1
172.16.0.45
10.0.0.5
10.0.0.5
192.168.1.20

ubuntu:~$ awk '{print $1 " -> " $6}' /var/log/access.log
192.168.1.1 -> 200
10.0.0.5 -> 401
192.168.1.1 -> 200
172.16.0.45 -> 403
10.0.0.5 -> 401
10.0.0.5 -> 200
192.168.1.20 -> 200

ubuntu:~$ awk '$6 == "401" {print $1}' /var/log/access.log
10.0.0.5
10.0.0.5
```

### Step 4: Advanced Log Reporting
**Let's combine awk , sort , and uniq to find out which IP is the most "chatty."**
  1. Identify the top attacker
  2. The "If" Statement in Awk: Only print the IP if the status code is NOT 200
     
```sh
ubuntu:~$ awk '{print $1}' /var/log/access.log | sort | uniq -c | sort -nr
      3 10.0.0.5
      2 192.168.1.1
      1 192.168.1.20
      1 172.16.0.45

ubuntu:~$ awk '$6 != "200" {print "securit alert: " $1}' /var/log/access.log
securit alert: 10.0.0.5
securit alert: 172.16.0.45
securit alert: 10.0.0.5
ubuntu:~$
```
