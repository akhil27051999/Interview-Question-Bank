# 🛠️ DevOps Shell Script Readme: 30 Practical Monitoring & Maintenance Scripts

## 📁 Script Index

1. Disk usage alert
2. Process monitor & auto-restart
3. File stats (lines, words, chars)
4. Port open check
5. Archive logs older than 7 days
6. Ping list of servers
7. Date-based backup
8. Decision making from user input
9. Extract IPs from log
10. Top memory processes
11. Check user existence
12. List active services
13. Delete old files
14. Current logged-in users
15. IP address validation
16. Print even numbers
17. Bulk user creation
18. System info display
19. Monitor CPU load
20. Schedule cron job
21. Process restart if not running
22. Disk threshold alert
23. Timestamped backup
24. Top memory-consuming processes
25. Extract IPs from access log
26. Internet connectivity check
27. Count total files
28. Archive old logs
29. Lowercase file names
30. Monitor open ports


## ✅ Script Documentation

### 1. Check Disk Usage and Alert if > 80%

```bash
#!/bin/bash
THRESHOLD=80
USAGE=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//')
if [ "$USAGE" -ge "$THRESHOLD" ]; then
    echo "Disk usage is above $THRESHOLD%. Current usage: $USAGE%."
else
    echo "Disk usage is normal: $USAGE%"
fi
```

**Explanation:** Uses `df`, `awk`, `sed` to extract root disk usage and alerts if above threshold.

---

### 2. Monitor Process (e.g., nginx) and Restart if Stopped

```bash
#!/bin/bash
PROCESS="nginx"
if ! pgrep "$PROCESS" > /dev/null; then
    echo "$PROCESS not running. Restarting..."
    systemctl start "$PROCESS"
else
    echo "$PROCESS is running."
fi
```

**Explanation:** Uses `pgrep` to verify if nginx is running. Restarts it using `systemctl` if not found.

---

### 3. Count Lines, Words, and Characters in a File

```bash
#!/bin/bash
FILE="$1"
if [ -f "$FILE" ]; then
    echo "Lines: $(wc -l < "$FILE")"
    echo "Words: $(wc -w < "$FILE")"
    echo "Characters: $(wc -c < "$FILE")"
else
    echo "File does not exist."
fi
```

**Explanation:** Uses `wc` to get line, word, and char count.

---

### 4. Check if a Port is Open on localhost

```bash
#!/bin/bash
PORT=80
if nc -zv 127.0.0.1 $PORT 2>/dev/null; then
    echo "Port $PORT is open."
else
    echo "Port $PORT is closed."
fi
```

**Explanation:** Uses `nc` to check TCP port status.

---

### 5. Archive and Compress Log Files Older Than 7 Days

```bash
#!/bin/bash
LOG_DIR="/var/log/myapp"
ARCHIVE_DIR="/var/log/archive"
mkdir -p "$ARCHIVE_DIR"
find "$LOG_DIR" -name "*.log" -type f -mtime +7 -exec gzip {} \; -exec mv {}.gz "$ARCHIVE_DIR" \;
```

**Explanation:** Finds `.log` files older than 7 days, compresses them using `gzip`, and moves them to an archive directory.

---

### 6. Ping a List of Servers from a File

```bash
#!/bin/bash
while read -r server; do
    if ping -c 1 "$server" > /dev/null; then
        echo "$server is reachable"
    else
        echo "$server is not reachable"
    fi
done < servers.txt
```

**Explanation:** Reads each hostname/IP from `servers.txt` and checks connectivity.

---

### 7. Backup a Directory with Date-Based Naming

```bash
#!/bin/bash
SRC="/home/user/data"
DEST="/backup/data_$(date +%F).tar.gz"
tar -czf "$DEST" "$SRC"
echo "Backup created at $DEST"
```

**Explanation:** Compresses a directory and names it with the current date.

---

### 8. Read User Input and Make a Decision

```bash
#!/bin/bash
read -p "Enter a number: " num
if [ "$num" -gt 0 ]; then
    echo "Positive number"
elif [ "$num" -lt 0 ]; then
    echo "Negative number"
else
    echo "Zero"
fi
```

**Explanation:** Accepts a number and checks its sign.

---

### 9. Extract IP Addresses from a File

```bash
#!/bin/bash
grep -Eo '([0-9]{1,3}\.){3}[0-9]{1,3}' access.log
```

**Explanation:** Uses regex to extract IPv4 addresses from logs.

---

### 10. Top 5 Memory-Consuming Processes

```bash
#!/bin/bash
ps aux --sort=-%mem | head -6
```

**Explanation:** Lists top 5 processes using the most memory.

---

### 11. Check if a User Exists on the System

```bash
#!/bin/bash
read -p "Enter username: " username
if id "$username" &>/dev/null; then
    echo "User $username exists."
else
    echo "User $username does not exist."
fi
```

**Explanation:** Uses `id` to check for user existence.

---

### 12. List All Active Services

```bash
#!/bin/bash
echo "Active services:"
systemctl list-units --type=service --state=running
```

**Explanation:** Displays running systemd services.

---

### 13. Delete Files Older Than 30 Days

```bash
#!/bin/bash
TARGET_DIR="/tmp/oldfiles"
find "$TARGET_DIR" -type f -mtime +30 -exec rm -f {} \;
echo "Deleted files older than 30 days from $TARGET_DIR"
```

**Explanation:** Uses `find` to clean up old files.

---

### 14. Get Current Logged-In Users

```bash
#!/bin/bash
echo "Currently logged-in users:"
who
```

**Explanation:** Uses `who` command.

---

### 15. Validate IP Address Format

```bash
#!/bin/bash
read -p "Enter IP address: " ip
if [[ $ip =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]]; then
    echo "Valid IP address format"
else
    echo "Invalid IP address format"
fi
```

**Explanation:** Checks input with IPv4 regex.

---

### 16. Print Even Numbers from 1 to 20

```bash
#!/bin/bash
for i in {1..20}; do
    if (( i % 2 == 0 )); then
        echo $i
    fi
done
```

**Explanation:** Loop with modulo to print evens.

---

### 17. Create Multiple User Accounts from a File

```bash
#!/bin/bash
while read -r user; do
    sudo useradd "$user" && echo "Created user: $user"
done < users.txt
```

**Explanation:** Batch user creation from list.

---

### 18. Display System Information

```bash
#!/bin/bash
echo "Hostname: $(hostname)"
echo "OS: $(uname -o)"
echo "Kernel: $(uname -r)"
echo "Architecture: $(uname -m)"
```

**Explanation:** Shows basic system info.

---

### 19. Monitor CPU Load Average

```bash
#!/bin/bash
echo "CPU Load Average:"
uptime | awk -F'load average:' '{ print $2 }'
```

**Explanation:** Extracts load average.

---

### 20. Schedule a Job Using Cron

```bash
#!/bin/bash
COMMAND="/path/to/script.sh"
SCHEDULE="0 5 * * *"
(crontab -l ; echo "$SCHEDULE $COMMAND") | crontab -
echo "Cron job scheduled: $COMMAND at $SCHEDULE"
```

**Explanation:** Adds a job to crontab.

---

### 21. Check and Restart Process if Not Running

```bash
#!/bin/bash
PROCESS="nginx"
if ! pgrep "$PROCESS" > /dev/null; then
    echo "$PROCESS is not running. Restarting..."
    sudo systemctl start "$PROCESS"
else
    echo "$PROCESS is running."
fi
```

**Explanation:** Verifies process with `pgrep` and restarts using `systemctl` if missing.

---

### 22. Monitor Disk Usage and Alert if Above Threshold

```bash
#!/bin/bash
THRESHOLD=80
USAGE=$(df / | grep / | awk '{print $5}' | sed 's/%//')
if [ "$USAGE" -gt "$THRESHOLD" ]; then
    echo "Disk usage is above $THRESHOLD%! Current: $USAGE%"
    # Here, add email or alert logic
fi
```

**Explanation:** Uses `df` and `awk` to monitor disk usage.

---

### 23. Backup Directory with Timestamp

```bash
#!/bin/bash
SRC="/var/www/html"
DEST="/backup/html_$(date +%F_%H-%M-%S).tar.gz"
tar -czf "$DEST" "$SRC"
echo "Backup created at $DEST"
```

**Explanation:** Archives with a timestamp to avoid overwriting backups.

---

### 24. Display Top 5 Memory-Consuming Processes

```bash
#!/bin/bash
echo "Top 5 Memory-Consuming Processes:"
ps aux --sort=-%mem | head -6
```

**Explanation:** Shows memory-heavy processes for resource monitoring.

---

### 25. Extract All IP Addresses from a Log File

```bash
#!/bin/bash
LOG="/var/log/nginx/access.log"
grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' "$LOG" | sort | uniq
```

**Explanation:** Uses regex to extract and deduplicate IPs.

---

### 26. Check Internet Connectivity

```bash
#!/bin/bash
ping -c 1 8.8.8.8 &> /dev/null
if [ $? -eq 0 ]; then
    echo "Internet is reachable."
else
    echo "No internet connection."
fi
```

**Explanation:** Pings Google DNS to validate internet.

---

### 27. Count Total Number of Files in a Directory

```bash
#!/bin/bash
DIR="/var/log"
COUNT=$(find "$DIR" -type f | wc -l)
echo "Total files in $DIR: $COUNT"
```

**Explanation:** Recursively counts all files.

---

### 28. Archive Logs Older Than 7 Days

```bash
#!/bin/bash
LOG_DIR="/var/log/myapp"
ARCHIVE="/var/log/myapp_archive"
mkdir -p "$ARCHIVE"
find "$LOG_DIR" -name "*.log" -mtime +7 -exec mv {} "$ARCHIVE" \;
echo "Archived old logs."
```

**Explanation:** Moves `.log` files older than 7 days to archive.

---

### 29. Convert All Filenames in Directory to Lowercase

```bash
#!/bin/bash
for f in *; do
    mv "$f" "$(echo $f | tr 'A-Z' 'a-z')"
done
```

**Explanation:** Batch-renames files in lowercase for consistency.

---

### 30. Monitor Open Ports on the System

```bash
#!/bin/bash
echo "Open ports:"
sudo netstat -tuln | grep LISTEN
```

**Explanation:** Lists listening TCP/UDP ports.
