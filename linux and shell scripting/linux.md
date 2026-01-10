# DevOps Interview Preparation Guide — Linux & Shell Scripting

## Table of Contents
- [Category 1: Linux Fundamentals](#category-1-linux-fundamentals)
- [Category 2: User & Permission Management](#category-2-user--permission-management)
- [Category 3: Process Management](#category-3-process-management)
- [Category 4: Disk & Filesystem Management](#category-4-disk--filesystem-management)
- [Category 5: Networking](#category-5-networking)
- [Category 6: Package Management](#category-6-package-management)
- [Category 7: Shell Scripting Fundamentals](#category-7-shell-scripting-fundamentals)
- [Category 8: Advanced Shell Scripting](#category-8-advanced-shell-scripting)
- [Category 9: Text Processing](#category-9-text-processing)
- [Category 10: Error Handling & Debugging](#category-10-error-handling--debugging)
- [Category 11: Performance Monitoring](#category-11-performance-monitoring)
- [Category 12: Automation & Cron Jobs](#category-12-automation--cron-jobs)
- [Category 13: Security & Hardening](#category-13-security--hardening)
- [Category 14: Real-world Scripting Scenarios](#category-14-real-world-scripting-scenarios)

---

## Category 1: Linux Fundamentals

### Q1 — What is the Linux boot process?
**Answer**

The Linux boot process consists of several stages:

- **BIOS/UEFI**: Initializes hardware and runs POST.  
- **Bootloader**: GRUB2 (commonly) loads the kernel (files in /boot).  
- **Kernel**: Initializes devices and mounts the root filesystem.  
- **Init process**: systemd (modern) or SysV init (legacy) starts system services.  
- **Runlevels/Targets**: System services are started according to targets.  
- **Login prompt**: User can log into the system.

---

### Q2 — Explain Linux filesystem hierarchy.
**Answer**

Key directories and purpose:

- `/` — Root directory  
- `/bin` — Essential user binaries  
- `/etc` — Configuration files  
- `/home` — User home directories  
- `/var` — Variable data (logs, spool files)  
- `/tmp` — Temporary files  
- `/usr` — User programs and data  
- `/opt` — Optional software packages  
- `/boot` — Boot loader files  
- `/dev` — Device files  
- `/proc` — Process and kernel information (virtual filesystem)

---

### Q3 — What is the difference between hard links and soft links?
**Answer**

- **Hard link**: Direct reference to the same inode. Cannot cross filesystems. File remains until all hard links removed.  
- **Soft link (symbolic)**: Pointer to a filename (path). Can cross filesystems; becomes broken if the original is deleted.

Examples:
```bash
# Create hard link
ln file.txt hardlink.txt

# Create soft link
ln -s file.txt softlink.txt
```
```sh
# ------------------- Create the base setup -------------------

user@568d442825b6: mkdir -p /opt/app/bin /etc/app /var/log/app /backup
user@568d442825b6: echo "console.log('Hello App')" > /opt/app/bin/app.js
user@568d442825b6: echo "DB=prod" > /etc/app/app.conf
user@568d442825b6: echo "App started" > /var/log/app/app.log


# ------------------- Create a HARD LINK for config backup -------------------

user@568d442825b6:~$ ln /etc/app/app.conf /backup/app.conf.hard

# Check inode numbers: 
user@568d442825b6:~$ ls -li /etc/app/app.conf /backup/app.conf.hard
543386 -rw-rw-r-- 2 user user 8 Jan 10 12:20 /backup/app.conf.hard
543386 -rw-rw-r-- 2 user user 8 Jan 10 12:20 /etc/app/app.conf

# Observation: Both have same inode number and same data on disk


# ------------------- Create a SOFT LINK (SYMLINK) for logs -------------------

user@568d442825b6:~$ ln -s /var/log/app/app.log /opt/app/app.log
user@568d442825b6:~$ ls -l /opt/app/app.log
lrwxrwxrwx 1 user user 20 Jan 10 12:26 /opt/app/app.log -> /var/log/app/app.log

# 1. Different inode
# 2. Symlink stores path, not data
# 3. Arrow (->) shows target


# ------------------- Modify data & observe behavior -------------------

user@568d442825b6:~$ echo "New entry" >> /backup/app.conf.hard

# Now check the main file
user@568d442825b6:~$ cat /etc/app/app.conf
DB=prod
New entry

# Observation: Changes reflect everywhere because both point to the same inode. ✔


# ------------------- Delete original file and test links -------------------

# Delete original config file 
user@568d442825b6:~$ rm /etc/app/app.conf
user@568d442825b6:~$ cat /backup/app.conf.hard
DB=prod
New entry

# Observation: 
# ✔ File still exists
# ✔ Data is safe
# ✔ Inode still valid

# Delete original log file
user@568d442825b6:~$ rm /var/log/app/app.log
user@568d442825b6:~$ cat /opt/app/app.log
cat: /opt/app/app.log: No such file or directory

# Observation: 
#  ❌ Broken link
#  ❌ “No such file or directory”


# ------------------- Move original file -------------------
user@568d442825b6:~$ mv /backup/app.conf.hard /backup/app.conf.moved
user@568d442825b6:~$ ls -li /backup/app.conf.moved 
543386 -rw-rw-r-- 1 user user 18 Jan 10 12:28 /backup/app.conf.moved

# Observation: Hard link still works (same inode)


user@568d442825b6:~$ mv /var/log/app /var/log/app_new
mv: cannot move '/var/log/app' to '/var/log/app_new': Permission denied
# ❌ Symlink breaks because path changed


# ------------------- Filesystem boundary test -------------------

user@568d442825b6:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay          29G  4.3G   25G  15% /
tmpfs            64M     0   64M   0% /dev
tmpfs            64M  876K   64M   2% /run
tmpfs           4.0M     0  4.0M   0% /run/lock
shm              64M     0   64M   0% /dev/shm
/dev/root        29G  4.3G   25G  15% /var/log/amazon/ssm
tmpfs           1.9G     0  1.9G   0% /proc/acpi
tmpfs           1.9G     0  1.9G   0% /proc/scsi
tmpfs           1.9G     0  1.9G   0% /sys/firmware
tmpfs           1.9G   20K  1.9G   1% /tmp
tmpfs            52M  8.0K   52M   1% /run/user/1001

user@568d442825b6:~$ ln /etc/app/app.conf /tmp/app.conf.hard
ln: failed to access '/etc/app/app.conf': No such file or directory

# ❌ If /etc and /tmp are on different filesystems → fails


# ------------------- Permissions behavior -------------------

user@568d442825b6:~$ chmod 000 /backup/app.conf.moved

# Hard link respects file permissions

user@568d442825b6:~$ cat /backup/app.conf.moved
cat: /backup/app.conf.moved: Permission denied

# Soft link permissions don’t matter — target file permissions apply

user@568d442825b6:~$ 
```
```

---

### Q4 — Explain Linux process states.
**Answer**

Common process states:

- **Running (R)**: Currently executing.  
- **Sleeping (S)**: Waiting for an event or resource.  
- **Stopped (T)**: Suspended by signal (e.g., SIGSTOP).  
- **Zombie (Z)**: Terminated but parent has not reaped it.  
- **Uninterruptible Sleep (D)**: Waiting for I/O; cannot be killed until I/O completes.

---

## Category 2: User & Permission Management

### Q5 — Explain Linux file permissions (rwx).
**Answer**

Permissions are specified for three classes: user (u), group (g), others (o).

- `r` = read (4)  
- `w` = write (2)  
- `x` = execute (1)

Example:
```bash
# rwxr-xr-- = 754
chmod 754 file.txt
# Owner: rwx (7), Group: r-x (5), Others: r-- (4)
```

---

### Q6 — What is the difference between su and sudo?
**Answer**

- `su`: Switch user; requires the target user's password (common use: `su -` to become root).  
- `sudo`: Execute a command as another user (usually root) using the invoking user's password and configured sudo privileges.

Examples:
```bash
# Switch to root (needs root password)
su -

# Run command as root (needs user's password)
sudo apt update
```

---

### Q7 — How to manage users and groups in Linux?
**Answer**

User examples:
```bash
# Create user
useradd -m -s /bin/bash john

# Set password
passwd john

# Add user to sudo group
usermod -aG sudo john

# Delete user and home
userdel -r john
```

Group examples:
```bash
# Create group
groupadd developers

# Add user to group
usermod -aG developers john

# Check groups for a user
groups john
```

---

### Q8 — What is umask and how does it work?
**Answer**

`umask` sets default permissions for newly created files and directories.

- Files default: `666 - umask`  
- Directories default: `777 - umask`

Example:
```bash
# Set umask to 022
umask 022
# Files: 666-022=644  (rw-r--r--)
# Dirs:  777-022=755  (rwxr-xr-x)
```

---

## Category 3: Process Management

### Q9 — How to monitor and manage processes?
**Answer**

Monitoring:
```bash
ps aux            # List all processes
top / htop        # Real-time monitoring
pstree            # Process tree view
```

Management:
```bash
kill 1234         # Send SIGTERM to PID 1234
kill -9 1234      # Force kill (SIGKILL)
pkill nginx       # Kill processes by name
killall java      # Kill all processes named java
```

---

### Q10 — Explain signals in Linux with examples.
**Answer**

Common signals:
- `SIGHUP (1)`: Hangup — often used to reload config.  
- `SIGINT (2)`: Interrupt (Ctrl+C).  
- `SIGKILL (9)`: Force terminate (cannot be caught).  
- `SIGTERM (15)`: Graceful termination (default for `kill`).  
- `SIGSTOP (19)`: Pause process.  
- `SIGCONT (18)`: Continue paused process.

Example:
```bash
kill -HUP 1234   # Send SIGHUP to PID 1234
```

---

### Q11 — What are foreground and background processes?
**Answer**

- **Foreground**: Process runs in the current terminal; blocks it until finished.  
- **Background**: Runs independently; terminal remains usable.

Examples:
```bash
# Run in background
./script.sh &

# Suspend foreground job and move to background
Ctrl+Z
bg

# Bring background job to foreground
fg
```

---

### Q12 — How to find and kill a process using a specific port?
**Answer**
```bash
# Find process using port 8080
lsof -i :8080

# Alternative
netstat -tulpn | grep 8080
ss -tulpn | grep 8080

# Kill process using port 8080
fuser -k 8080/tcp
```

---

## Category 4: Disk & Filesystem Management

### Q13 — How to check disk usage in Linux?
**Answer**
```bash
# Disk space by filesystem
df -h

# Disk usage by directory
du -sh /path/to/directory

# Detailed per-subdir
du -h --max-depth=1 /home

# Find large files
find / -type f -size +100M 2>/dev/null
```

---

### Q14 — Explain Linux filesystem types.
**Answer**

Common filesystems:

- `ext4` — Default for many distros.  
- `XFS` — High performance, good for large files.  
- `btrfs` — Copy-on-write, snapshots, compression.  
- `ZFS` — Advanced features, data integrity.  
- `tmpfs` — In-memory filesystem.  
- `swap` — Virtual memory.

---

### Q15 — How to mount and unmount filesystems?
**Answer**
```bash
# Mount filesystem
mount /dev/sdb1 /mnt/data

# Mount with options
mount -t ext4 -o defaults,noatime /dev/sdb1 /mnt/data

# Unmount
umount /mnt/data

# Mount all from /etc/fstab
mount -a

# Check mounted filesystems
findmnt
```

---

### Q16 — What is LVM and its components?
**Answer**

Logical Volume Manager (LVM) provides flexible volume management:

- **PV (Physical Volume)** — Physical disk or partition.  
- **VG (Volume Group)** — Pool of PVs.  
- **LV (Logical Volume)** — Volume created from VG, formatted and mounted.

Example:
```bash
pvcreate /dev/sdb
vgcreate myvg /dev/sdb
lvcreate -L 10G -n mylv myvg
mkfs.ext4 /dev/myvg/mylv
mount /dev/myvg/mylv /mnt/data
```

---

## Category 5: Networking

### Q17 — How to check network configuration?
**Answer**
```bash
# Interfaces and IPs
ip addr show       # or ifconfig

# Routing table
ip route show      # or route -n

# Listening ports and sockets
ss -tulpn          # or netstat -tulpn

# DNS resolution
nslookup google.com
dig google.com
```

---

### Q18 — How to troubleshoot network connectivity?
**Answer**
```bash
# Basic connectivity
ping google.com

# Trace route
traceroute google.com
mtr google.com

# DNS
nslookup google.com

# Check TCP port reachability
telnet google.com 80
nc -v google.com 80

# Check firewall rules
iptables -L
```

---

### Q19 — Explain common network configuration files.
**Answer**

- `/etc/hosts` — Static hostname resolution.  
- `/etc/resolv.conf` — DNS resolver configuration.  
- `/etc/nsswitch.conf` — Name service switch order.  
- `/etc/sysconfig/network-scripts/` — RHEL/CentOS network scripts.  
- `/etc/netplan/` — Ubuntu (netplan) configuration.

---

### Q20 — How to use tcpdump for network analysis?
**Answer**
```bash
# Capture all traffic on eth0 (requires sudo)
tcpdump -i eth0

# Capture HTTP traffic
tcpdump -i eth0 port 80

# Capture traffic to/from a specific IP
tcpdump -i eth0 host 192.168.1.100

# Save capture to file
tcpdump -i eth0 -w capture.pcap

# Read from file
tcpdump -r capture.pcap
```

---

## Category 6: Package Management

### Q21 — Compare apt, yum, and dpkg.
**Answer**

- `apt` (Debian/Ubuntu) — High-level package management (`apt install`, `apt update`).  
- `dpkg` — Low-level package tool for Debian (`dpkg -i package.deb`).  
- `yum` / `dnf` (RHEL/CentOS/Fedora) — Red Hat family package management (`yum install`, `dnf install`).  
- `rpm` — Low-level RPM tool (`rpm -i package.rpm`).

Examples:
```bash
# Ubuntu/Debian
apt update
apt install nginx
dpkg -i package.deb

# RHEL/CentOS
yum update
yum install nginx
rpm -i package.rpm
```

---

### Q22 — How to find which package provides a specific file?
**Answer**
```bash
# Debian/Ubuntu
dpkg -S /bin/ls

# RHEL/CentOS
rpm -qf /bin/ls

# Search package repo for filename
apt-file search filename
yum provides filename
```

---

### Q23 — How to create a simple Debian package?
**Answer**
```bash
# Directory structure
mkdir -p myapp/DEBIAN
mkdir -p myapp/usr/local/bin

# Control file
cat > myapp/DEBIAN/control << EOF
Package: myapp
Version: 1.0
Section: utils
Priority: optional
Architecture: amd64
Maintainer: John Doe <john@example.com>
Description: My sample application
 A simple demonstration package.
EOF

# Build package
dpkg-deb --build myapp
```

---

## Category 7: Shell Scripting Fundamentals

### Q24 — What is a shebang line and why is it important?
**Answer**

The shebang (`#!`) specifies the interpreter to execute the script.

Examples:
```bash
#!/bin/bash
#!/usr/bin/python3
#!/bin/sh
```

Usage:
```bash
# Make script executable
chmod +x script.sh
./script.sh
# Or run with interpreter
bash script.sh
```

---

### Q25 — Explain different types of variables in shell scripting.
**Answer**

- **Local variables**:
```bash
name="John"
count=10
```
- **Environment variables** (exported):
```bash
export DATABASE_URL="postgresql://localhost/mydb"
```
- **Special variables**:
```bash
$0  # script name
$1  # first argument
$@  # all arguments
$#  # number of arguments
$$  # PID of script
$?  # exit status of last command
```

---

### Q26 — How to handle command-line arguments?
**Answer**
```bash
# Access positional args
echo "First arg: $1"

# Loop through args
for arg in "$@"; do
    echo "Arg: $arg"
done

# Using getopts for options
while getopts "u:p:" opt; do
    case $opt in
        u) username="$OPTARG" ;;
        p) password="$OPTARG" ;;
        *) echo "Usage: $0 -u user -p pass" ;;
    esac
done
```

---

### Q27 — Explain different types of quotes in shell.
**Answer**

- **Single quotes `'...'`**: Preserve literal value of enclosed characters.  
- **Double quotes `"..."`**: Allow variable and command substitution.  
- **Command substitution**: `$(...)` or backticks `` `...` ``.

Examples:
```bash
name="John"
echo 'Hello $name'       # prints: Hello $name
echo "Hello $name"       # prints: Hello John
echo "Today is $(date)"  # prints current date
```

---

## Category 8: Advanced Shell Scripting

### Q28 — How to use arrays in shell scripting?
**Answer**
```bash
# Indexed array
fruits=("apple" "banana" "cherry")
echo "${fruits[0]}"      # apple
echo "${fruits[@]}"      # all elements
echo "${#fruits[@]}"     # length

# Loop
for f in "${fruits[@]}"; do
  echo "Fruit: $f"
done

# Associative arrays (bash 4+)
declare -A colors
colors["red"]="#FF0000"
echo "${colors[red]}"
```

---

### Q29 — Explain process substitution.
**Answer**

Process substitution uses `<(...)` or `>(...)` to treat command output as a file.

Examples:
```bash
# Compare outputs of two commands
diff <(ls /bin) <(ls /usr/bin)

# Read command output in a loop
while read line; do
  echo "Line: $line"
done < <(grep "error" /var/log/syslog)
```

---

### Q30 — How to create functions in shell scripts?
**Answer**
```bash
# Function definition
greet() {
  local name=$1
  echo "Hello, $name!"
}

is_even() {
  local n=$1
  if (( n % 2 == 0 )); then
    return 0
  else
    return 1
  fi
}

greet "Alice"
if is_even 10; then echo "even"; fi
```

---

## Category 9: Text Processing

### Q31 — How to use grep, sed, and awk effectively?
**Answer**

`grep` examples:
```bash
grep "error" file.log
grep -i "error" file.log     # case-insensitive
grep -r "TODO" /path         # recursive
grep -v "success" file.log   # invert match
grep -c "pattern" file.log   # count matches
```

`sed` examples:
```bash
sed 's/old/new/' file.txt          # replace first occurrence per line
sed 's/old/new/g' -i file.txt      # replace all occurrences in-place
sed '/pattern/d' file.txt          # delete lines with pattern
```

`awk` examples:
```bash
awk '{print $1}' file.txt
awk '$3 > 100 {print $1, $3}' file.txt
awk -F',' '{print $2}' file.csv
awk '{print "Line:", NR, "Cols:", NF}' file.txt
```

---

### Q32 — How to parse CSV files in shell?
**Answer**
```bash
# Using awk
awk -F',' '{print "Name:", $1, "Age:", $2}' data.csv

# Using while IFS
while IFS=',' read -r name age email; do
  echo "Name: $name, Age: $age, Email: $email"
done < data.csv

# Skip header example
{ read header; while IFS=',' read -r name age email; do ...; done } < data.csv
```

---

### Q33 — How to validate and sanitize input in scripts?
**Answer**
```bash
# Check argument provided
if [ $# -eq 0 ]; then
  echo "Usage: $0 <filename>"
  exit 1
fi

# Validate file exists
file=$1
if [ ! -f "$file" ]; then
  echo "File not found"
  exit 1
fi

# Numeric validation
read -p "Enter age: " age
if ! [[ "$age" =~ ^[0-9]+$ ]]; then
  echo "Age must be numeric"
  exit 1
fi

# Simple sanitize
username=$(echo "$1" | tr -d '[:space:]' | tr '[:upper:]' '[:lower:]')
```

---

## Category 10: Error Handling & Debugging

### Q34 — How to handle errors in shell scripts?
**Answer**
```bash
#!/bin/bash
set -e                # exit on error
set -u                # treat unset vars as error
set -o pipefail       # pipeline returns non-zero if any command fails

trap 'echo "Error at line $LINENO"; exit 1' ERR

# Example check
if ! some_command; then
  echo "Command failed" >&2
  exit 1
fi
```

Logging helper:
```bash
error() { echo "[$(date)] ERROR: $*" >&2; }
error "Something went wrong"
```

---

### Q35 — How to debug shell scripts?
**Answer**
```bash
# Enable verbose mode
set -x
PS4='+ $LINENO: '

# Turn on/off around debug section
set -x
# debug code
set +x

# Conditional debug function
debug() { [ "${DEBUG:-false}" = "true" ] && echo "DEBUG: $*" >&2; }
DEBUG=true; debug "variable=$var"
```

---

### Q36 — How to create log files from scripts?
**Answer**
```bash
LOG_FILE="/var/log/myscript.log"

log() {
  echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

log "Script started"

# Redirect block output
{
  echo "Starting process..."
  some_command
  echo "Completed"
} >> "$LOG_FILE" 2>&1
```

---

## Category 11: Performance Monitoring

### Q37 — How to monitor system performance?
**Answer**
- CPU: `top`, `htop`, `mpstat`  
- Memory: `free -h`, `vmstat`  
- Disk I/O: `iostat`, `iotop`  
- Network: `iftop`, `nethogs`  
- Overall: `sar`, `dstat`

---

### Q38 — How to find and resolve high CPU usage?
**Answer**
```bash
ps aux --sort=-%cpu | head -10
top -p <PID>
cat /proc/<PID>/status
pidstat 1 5
perf top   # advanced profiling
```
Investigate runaway processes, optimize code, or limit resources (cgroups).

---

### Q39 — How to troubleshoot memory issues?
**Answer**
```bash
free -h
cat /proc/meminfo
ps aux --sort=-%mem | head -10
swapon --show
vmstat 1 5
# Clear page cache if safe (requires care)
echo 3 > /proc/sys/vm/drop_caches
```
For leaks, use tools like `valgrind` for native binaries.

---

## Category 12: Automation & Cron Jobs

### Q40 — How to schedule tasks with cron?
**Answer**

Edit crontab with `crontab -e`.

Cron format:
```
# ┌──────── minute (0-59)
# │ ┌────── hour (0-23)
# │ │ ┌──── day of month (1-31)
# │ │ │ ┌── month (1-12)
# │ │ │ │ ┌ day of week (0-7) (0 or 7 = Sun)
# │ │ │ │ │
# * * * * * command
```

Examples:
```cron
0 * * * * /path/to/script.sh       # hourly
0 2 * * * /path/to/backup.sh       # daily 2:00 AM
*/5 * * * * /path/to/monitor.sh    # every 5 minutes
0 0 * * 0 /path/to/weekly.sh       # weekly on Sunday
```

---

### Q41 — How to create systemd services?
**Answer**
Create `/etc/systemd/system/myservice.service`:
```ini
[Unit]
Description=My Custom Service
After=network.target

[Service]
Type=simple
User=myuser
ExecStart=/usr/local/bin/myscript.sh
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```
Enable & start:
```bash
sudo systemctl daemon-reload
sudo systemctl enable myservice
sudo systemctl start myservice
sudo systemctl status myservice
```

---

## Category 13: Security & Hardening

### Q42 — How to secure a Linux server?
**Answer**

Key steps:
- Keep system updated:
```bash
apt update && apt upgrade
```
- Configure firewall (`ufw` example):
```bash
ufw enable
ufw allow ssh
ufw allow 80/tcp
ufw allow 443/tcp
```
- Disable root SSH login and use key-based auth:
```bash
# /etc/ssh/sshd_config
PermitRootLogin no
```
- Install `fail2ban` and enable unattended upgrades:
```bash
apt install fail2ban unattended-upgrades
```

---

### Q43 — How to audit file permissions and security?
**Answer**
```bash
# World-writable files
find / -type f -perm -o+w 2>/dev/null

# SUID/SGID files
find / -type f \( -perm -4000 -o -perm -2000 \) 2>/dev/null

# List users
cut -d: -f1 /etc/passwd

# Audit SSH keys
ls -la ~/.ssh/

# Check sudoers
sudo -l
```

---

## Category 14: Real-world Scripting Scenarios

### Q44 — How to create a backup script?
**Answer**
```bash
#!/bin/bash
BACKUP_DIR="/backup"
SOURCE_DIRS=("/home" "/etc" "/var/log")
RETENTION_DAYS=7
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_$DATE.tar.gz"

tar -czf "$BACKUP_DIR/$BACKUP_FILE" "${SOURCE_DIRS[@]}" 2>/dev/null

if [ $? -eq 0 ]; then
  echo "Backup completed: $BACKUP_FILE"
  find "$BACKUP_DIR" -name "backup_*.tar.gz" -mtime +$RETENTION_DAYS -delete
else
  echo "Backup failed!" >&2
  exit 1
fi
```

---

### Q45 — How to create a log analysis script?
**Answer**
```bash
#!/bin/bash
LOG_FILE="/var/log/nginx/access.log"
REPORT="/tmp/access_report_$(date +%Y%m%d).txt"

{
  echo "NGINX Access Log Report - $(date)"
  echo "Total requests: $(wc -l < "$LOG_FILE")"
  echo -e "\nTop 10 IPs:"
  awk '{print $1}' "$LOG_FILE" | sort | uniq -c | sort -nr | head -10
  echo -e "\nTop 10 URLs:"
  awk '{print $7}' "$LOG_FILE" | sort | uniq -c | sort -nr | head -10
  echo -e "\nHTTP Response codes:"
  awk '{print $9}' "$LOG_FILE" | sort | uniq -c | sort -nr
  echo -e "\nRequests per hour:"
  awk '{print $4}' "$LOG_FILE" | cut -d: -f1,2 | uniq -c
} > "$REPORT"

echo "Report generated: $REPORT"
```

---

### Q46 — How to create a system monitoring script?
**Answer**
```bash
#!/bin/bash
THRESHOLD_CPU=80
THRESHOLD_MEM=90
THRESHOLD_DISK=85
ALERT_EMAIL="admin@example.com"

CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
MEM_USAGE=$(free | awk '/Mem/ {printf "%.0f", $3/$2 * 100}')
DISK_USAGE=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')

ALERT=""
[ "$CPU_USAGE" -gt "$THRESHOLD_CPU" ] && ALERT+="CPU high: ${CPU_USAGE}%\n"
[ "$MEM_USAGE" -gt "$THRESHOLD_MEM" ] && ALERT+="Mem high: ${MEM_USAGE}%\n"
[ "$DISK_USAGE" -gt "$THRESHOLD_DISK" ] && ALERT+="Disk high: ${DISK_USAGE}%\n"

if [ -n "$ALERT" ]; then
  echo -e "$ALERT" | mail -s "System Alert - $(hostname)" "$ALERT_EMAIL"
fi
```

---

### Q47 — How to create a user management script?
**Answer**
```bash
#!/bin/bash
USER_FILE="/tmp/users.csv"
LOG="/var/log/user_management.log"

log() { echo "[$(date)] $*" >> "$LOG"; }

create_user() {
  user=$1; groups=$2
  if id "$user" &>/dev/null; then log "User $user exists"; return 1; fi
  useradd -m -s /bin/bash "$user"
  pass=$(openssl rand -base64 12)
  echo "$user:$pass" | chpasswd
  IFS=',' read -ra garr <<< "$groups"
  for g in "${garr[@]}"; do usermod -aG "$g" "$user"; done
  log "Created $user groups:$groups"
  echo "User:$user Pass:$pass Groups:$groups"
}

[ -f "$USER_FILE" ] || { echo "User file not found"; exit 1; }
while IFS=',' read -r user groups; do
  [[ "$user" == "username" || -z "$user" ]] && continue
  create_user "$user" "$groups"
done < "$USER_FILE"
```

---

### Q48 — How to create a Docker container management script?
**Answer**
```bash
#!/bin/bash
manage() {
  case $1 in
    start) docker-compose up -d ;;
    stop)  docker-compose down ;;
    restart) docker-compose restart ;;
    backup)
      docker exec mydb sh -c 'mysqldump -u root -p$MYSQL_ROOT_PASSWORD $MYSQL_DATABASE' > /backup/db_$(date +%F).sql
      ;;
    logs) docker-compose logs -f ;;
    update)
      docker-compose pull
      docker-compose down
      docker-compose up -d
      ;;
    *) echo "Usage: $0 {start|stop|restart|backup|logs|update}" ; exit 1;;
  esac
}
docker info &>/dev/null || { echo "Docker not running"; exit 1; }
manage "$1"
```

---

### Q49 — How to create an SSL certificate monitoring script?
**Answer**
```bash
#!/bin/bash
DOMAINS=("example.com")
THRESHOLD=30
ALERT="admin@example.com"

for d in "${DOMAINS[@]}"; do
  exp=$(echo | openssl s_client -servername "$d" -connect "$d:443" 2>/dev/null \
       | openssl x509 -noout -dates | grep notAfter | cut -d= -f2)
  [ -z "$exp" ] && echo "Could not fetch $d" && continue
  days=$(( ( $(date -d "$exp" +%s) - $(date +%s) ) / 86400 ))
  echo "Domain: $d expires in $days days"
  if [ "$days" -le "$THRESHOLD" ]; then
    echo "SSL for $d expires in $days days" | mail -s "SSL Alert $d" "$ALERT"
  fi
done
```

---

### Q50 — How to create a database maintenance script?
**Answer**
```bash
#!/bin/bash
DB_HOST=localhost; DB_USER=admin; DB_PASS=pass; DB_NAME=mydb
BACKUP_DIR=/backup/db; RETENTION=7

backup() {
  ts=$(date +%F_%H%M%S)
  mkdir -p "$BACKUP_DIR"
  if mysqldump -h "$DB_HOST" -u "$DB_USER" -p"$DB_PASS" "$DB_NAME" > "$BACKUP_DIR/backup_$ts.sql"; then
    gzip "$BACKUP_DIR/backup_$ts.sql"
    echo "Backup created: $BACKUP_DIR/backup_$ts.sql.gz"
  else
    echo "Backup failed" >&2; return 1
  fi
}

optimize() {
  tables=$(mysql -h "$DB_HOST" -u "$DB_USER" -p"$DB_PASS" -e "SHOW TABLES" "$DB_NAME" | tail -n +2 | tr '\n' ',')
  mysql -h "$DB_HOST" -u "$DB_USER" -p"$DB_PASS" -e "OPTIMIZE TABLE $tables" "$DB_NAME"
}

cleanup() {
  find "$BACKUP_DIR" -name "backup_*.sql.gz" -mtime +$RETENTION -delete
}

case $1 in
  backup) backup ;;
  optimize) optimize ;;
  cleanup) cleanup ;;
  full) backup; optimize; cleanup ;;
  *) echo "Usage: $0 {backup|optimize|cleanup|full}"; exit 1 ;;
esac
```

