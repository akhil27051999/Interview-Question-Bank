# DevOps Interview Preparation Guide

## Table of Contents
- [Linux & Shell Scripting](#linux--shell-scripting)
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

## Linux & Shell Scripting

### Category 1: Linux Fundamentals

1. **What is the Linux boot process?**  
Answer:  
The Linux boot process consists of several stages:

BIOS/UEFI: Initializes hardware and runs POST

Bootloader: GRUB2 loads the kernel (located in /boot)

Kernel: Initializes devices and mounts root filesystem

Init Process: systemd (modern) or SysV init (legacy) starts

Runlevels/Targets: System services are started

Login Prompt: User can log into the system

2. **Explain Linux file system hierarchy.**  
Answer:  
Key directories in Linux:

/ - Root directory

/bin - Essential user binaries

/etc - Configuration files

/home - User home directories

/var - Variable data (logs, spool files)

/tmp - Temporary files

/usr - User programs and data

/opt - Optional software packages

/boot - Boot loader files

/dev - Device files

/proc - Process information

3. **What is the difference between hard links and soft links?**  
Answer:

Hard Link: Direct reference to inode, can't cross filesystems, deleted only when all links removed

Soft Link (Symbolic): Pointer to filename, can cross filesystems, broken if original deleted

Example:

```bash
# Create hard link
ln file.txt hardlink.txt

# Create soft link
ln -s file.txt softlink.txt
```

4. **Explain Linux process states.**  
Answer:

Running (R): Currently executing

Sleeping (S): Waiting for event

Stopped (T): Suspended by signal

Zombie (Z): Terminated but parent hasn't waited

Uninterruptible Sleep (D): Waiting for I/O, can't be killed

---

### Category 2: User & Permission Management

5. **Explain Linux file permissions (rwx).**  
Answer:  
Three permission types for three user classes:

User (u): File owner

Group (g): Group members

Others (o): Everyone else

Permissions:

r (4): Read

w (2): Write

x (1): Execute

Example:

```bash
# rwxr-xr-- = 754
chmod 754 file.txt
# Owner: rwx (7), Group: r-x (5), Others: r-- (4)
```

6. **What is the difference between su and sudo?**  
Answer:

su: Switch user, requires target user's password

sudo: Execute command as another user (usually root), requires user's own password and sudo privileges

Example:

```bash
# Switch to root (needs root password)
su -

# Run command as root (needs user's password)
sudo apt update
```

7. **How to manage users and groups in Linux?**  
Answer:  
User Management:

```bash
# Create user
useradd -m -s /bin/bash john

# Set password
passwd john

# Modify user
usermod -aG sudo john

# Delete user
userdel -r john
```

Group Management:

```bash
# Create group
groupadd developers

# Add user to group
usermod -aG developers john

# Check groups
groups john
```

8. **What is umask and how does it work?**  
Answer:  
umask sets default permissions for new files and directories.

Calculation:

Files: 666 - umask

Directories: 777 - umask

Example:

```bash
# Set umask to 022
umask 022
# Files: 666-022=644 (rw-r--r--)
# Directories: 777-022=755 (rwxr-xr-x)
```

---

### Category 3: Process Management

9. **How to monitor and manage processes?**  
Answer:  
Monitoring:

```bash
# List all processes
ps aux

# Real-time monitoring
top
htop

# Process tree
pstree
```

Management:

```bash
# Kill process by PID
kill 1234

# Force kill
kill -9 1234

# Kill by name
pkill nginx

# Kill all matching processes
killall java
```

10. **Explain signals in Linux with examples.**  
Answer:  
Common signals:

SIGHUP (1): Hangup, reload configuration

SIGINT (2): Interrupt (Ctrl+C)

SIGKILL (9): Force terminate (cannot be caught)

SIGTERM (15): Graceful termination (default for kill)

SIGSTOP (19): Pause process

SIGCONT (18): Continue paused process

11. **What are foreground and background processes?**  
Answer:

Foreground: Runs in current terminal, user must wait

Background: Runs independently, user can continue working

Example:

```bash
# Run in background
./script.sh &

# Move running job to background
Ctrl+Z
bg

# Bring background job to foreground
fg
```

12. **How to find and kill a process using a specific port?**  
Answer:

```bash
# Find process using port 8080
lsof -i :8080
# or
netstat -tulpn | grep 8080
# or
ss -tulpn | grep 8080

# Kill process using port 8080
fuser -k 8080/tcp
```

---

### Category 4: Disk & Filesystem Management

13. **How to check disk usage in Linux?**  
Answer:

```bash
# Disk space by filesystem
df -h

# Disk usage by directory
du -sh /path/to/directory

# Detailed disk usage
du -h --max-depth=1 /home

# Find large files
find / -type f -size +100M 2>/dev/null
```

14. **Explain Linux filesystem types.**  
Answer:  
Common filesystems:

ext4: Default for most Linux distributions

XFS: High performance, good for large files

Btrfs: Copy-on-write, snapshots, compression

ZFS: Advanced features, data integrity

tmpfs: In-memory filesystem

swap: Used for virtual memory

15. **How to mount and unmount filesystems?**  
Answer:

```bash
# Mount filesystem
mount /dev/sdb1 /mnt/data

# Mount with options
mount -t ext4 -o defaults,noatime /dev/sdb1 /mnt/data

# Unmount
umount /mnt/data

# Mount all from fstab
mount -a

# Check mounted filesystems
mount
# or
findmnt
```

16. **What is LVM and its components?**  
Answer:  
Logical Volume Manager provides abstraction layer over physical storage.

Components:

Physical Volume (PV): Physical disk or partition

Volume Group (VG): Collection of PVs

Logical Volume (LV): Partition created from VG

Example:

```bash
# Create physical volume
pvcreate /dev/sdb

# Create volume group
vgcreate myvg /dev/sdb

# Create logical volume
lvcreate -L 10G -n mylv myvg

# Format and mount
mkfs.ext4 /dev/myvg/mylv
mount /dev/myvg/mylv /mnt/data
```

---

### Category 5: Networking

17. **How to check network configuration?**  
Answer:

```bash
# Network interfaces
ip addr show
# or
ifconfig

# Routing table
ip route show
# or
route -n

# Network statistics
ss -tulpn
# or
netstat -tulpn

# DNS resolution
nslookup google.com
dig google.com
```

18. **How to troubleshoot network connectivity?**  
Answer:

```bash
# Check connectivity
ping google.com

# Trace route
traceroute google.com
mtr google.com

# Check DNS
nslookup google.com

# Check specific port
telnet google.com 80
nc -v google.com 80

# Check firewall
iptables -L
```

19. **Explain common network configuration files.**  
Answer:

/etc/hosts: Static hostname resolution

/etc/resolv.conf: DNS configuration

/etc/nsswitch.conf: Name service switch configuration

/etc/sysconfig/network-scripts/: Network scripts (RHEL/CentOS)

/etc/netplan/: Network configuration (Ubuntu)

20. **How to use tcpdump for network analysis?**  
Answer:

```bash
# Capture all traffic on eth0
tcpdump -i eth0

# Capture HTTP traffic
tcpdump -i eth0 port 80

# Capture traffic to/from specific IP
tcpdump -i eth0 host 192.168.1.100

# Save to file
tcpdump -i eth0 -w capture.pcap

# Read from file
tcpdump -r capture.pcap
```

---

### Category 6: Package Management

21. **Compare apt, yum, and dpkg.**  
Answer:

apt (Debian/Ubuntu): High-level package management

dpkg (Debian/Ubuntu): Low-level package installation

yum/dnf (RHEL/CentOS): Red Hat package management

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

22. **How to find which package provides a specific file?**  
Answer:

```bash
# Debian/Ubuntu
dpkg -S /bin/ls

# RHEL/CentOS
rpm -qf /bin/ls

# Search package repository
apt-file search filename
yum provides filename
```

23. **How to create a simple Debian package?**  
Answer:

```bash
# Create directory structure
mkdir -p myapp/DEBIAN
mkdir -p myapp/usr/local/bin

# Create control file
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

### Category 7: Shell Scripting Fundamentals

24. **What is a shebang line and why is it important?**  
Answer:  
The shebang (#!) tells the system which interpreter to use.

Examples:

```bash
#!/bin/bash           # Use bash shell
#!/usr/bin/python3    # Use Python 3
#!/bin/sh             # Use system shell

# Without execute permission
bash script.sh

# With execute permission
chmod +x script.sh
./script.sh
```

25. **Explain different types of variables in shell scripting.**  
Answer:

```bash
#!/bin/bash

# Local variables
name="John"
count=10

# Environment variables (exported)
export DATABASE_URL="postgresql://localhost/mydb"

# Special variables
echo "Script name: $0"
echo "First argument: $1"
echo "All arguments: $@"
echo "Number of arguments: $#"
echo "Process ID: $$"
echo "Exit status: $?"
```

26. **How to handle command-line arguments?**  
Answer:

```bash
#!/bin/bash

# Basic arguments
echo "First argument: $1"
echo "Second argument: $2"

# Loop through all arguments
for arg in "$@"; do
    echo "Argument: $arg"
done

# Using getopts for options
while getopts "u:p:" opt; do
    case $opt in
        u) username="$OPTARG" ;;
        p) password="$OPTARG" ;;
        *) echo "Usage: $0 -u username -p password" ;;
    esac
done
```

27. **Explain different types of quotes in shell.**  
Answer:

Single quotes ('): Preserve literal value of all characters

Double quotes ("): Allow variable and command substitution

Backticks (`) or $(): Command substitution

Example:

```bash
name="John"

echo 'Hello $name'      # Output: Hello $name
echo "Hello $name"      # Output: Hello John
echo "Today is $(date)" # Output: Today is [current date]
```

---

### Category 8: Advanced Shell Scripting

28. **How to use arrays in shell scripting?**  
Answer:

```bash
#!/bin/bash

# Declare array
fruits=("apple" "banana" "cherry")

# Access elements
echo "First fruit: ${fruits[0]}"
echo "All fruits: ${fruits[@]}"

# Array length
echo "Number of fruits: ${#fruits[@]}"

# Loop through array
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done

# Associative arrays (bash 4+)
declare -A colors
colors["red"]="#FF0000"
colors["green"]="#00FF00"
echo "Red: ${colors[red]}"
```

29. **Explain process substitution.**  
Answer:  
Process substitution allows using command output as temporary files.

Example:

```bash
# Compare outputs of two commands
diff <(ls /bin) <(ls /usr/bin)

# Feed while loop with command output
while read line; do
    echo "Processing: $line"
done < <(grep "error" /var/log/syslog)
```

30. **How to create functions in shell scripts?**  
Answer:

```bash
#!/bin/bash

# Function definition
greet() {
    local name=$1
    echo "Hello, $name!"
}

# Function with return value
is_even() {
    local num=$1
    if (( num % 2 == 0 )); then
        return 0  # true
    else
        return 1  # false
    fi
}

# Call functions
greet "Alice"
greet "Bob"

# Use return value
if is_even 10; then
    echo "10 is even"
else
    echo "10 is odd"
fi
```

---

### Category 9: Text Processing

31. **How to use grep, sed, and awk effectively?**  
Answer:  
grep examples:

```bash
# Basic search
grep "error" file.log

# Case insensitive
grep -i "error" file.log

# Recursive search
grep -r "TODO" /path/to/code

# Inverse match
grep -v "success" file.log

# Count matches
grep -c "pattern" file.log
```

sed examples:

```bash
# Replace first occurrence per line
sed 's/old/new/' file.txt

# Replace all occurrences
sed 's/old/new/g' file.txt

# Delete lines containing pattern
sed '/pattern/d' file.txt

# In-place edit
sed -i 's/old/new/g' file.txt
```

awk examples:

```bash
# Print specific column
awk '{print $1}' file.txt

# Print lines where column matches
awk '$3 > 100 {print $1, $3}' file.txt

# Custom field separator
awk -F',' '{print $2}' file.csv

# Built-in variables
awk '{print "Line:", NR, "Columns:", NF}' file.txt
```

32. **How to parse CSV files in shell?**  
Answer:

```bash
#!/bin/bash

# Using awk
awk -F',' '{
    print "Name:", $1, "Age:", $2
}' data.csv

# Using IFS with while loop
while IFS=',' read -r name age email; do
    echo "Name: $name, Age: $age, Email: $email"
done < data.csv

# Skip header
{
    read header  # skip header line
    while IFS=',' read -r name age email; do
        echo "Processing: $name"
    done
} < data.csv
```

33. **How to validate and sanitize input in scripts?**  
Answer:

```bash
#!/bin/bash

# Check if argument is provided
if [ $# -eq 0 ]; then
    echo "Usage: $0 <filename>"
    exit 1
fi

# Validate file exists
filename=$1
if [ ! -f "$filename" ]; then
    echo "Error: File $filename not found"
    exit 1
fi

# Validate numeric input
read -p "Enter age: " age
if ! [[ "$age" =~ ^[0-9]+$ ]]; then
    echo "Error: Age must be a number"
    exit 1
fi

# Sanitize input
username=$(echo "$1" | tr -d '[:space:]' | tr '[:upper:]' '[:lower:]')
```

---

### Category 10: Error Handling & Debugging

34. **How to handle errors in shell scripts?**  
Answer:

```bash
#!/bin/bash

# Exit on error
set -e

# Exit on undefined variables
set -u

# Exit on pipe failure
set -o pipefail

# Trap errors
trap 'echo "Error at line $LINENO"; exit 1' ERR

# Check command success
if ! command; then
    echo "Command failed"
    exit 1
fi

# Log errors with timestamp
error() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] ERROR: $*" >&2
}

# Usage
error "Something went wrong"
```

35. **How to debug shell scripts?**  
Answer:

```bash
#!/bin/bash

# Print commands before execution
set -x

# Print line numbers
PS4='+ $LINENO: '

# Debug specific section
set -x
# code to debug
set +x

# Use debug function
debug() {
    if [ "$DEBUG" = "true" ]; then
        echo "DEBUG: $*" >&2
    fi
}

# Usage
DEBUG=true
debug "Variable value: $variable"
```

36. **How to create log files from scripts?**  
Answer:

```bash
#!/bin/bash

# Log file
LOG_FILE="/var/log/myscript.log"

# Log function
log() {
    local level=$1
    shift
    local message=$*
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $level: $message" | tee -a "$LOG_FILE"
}

# Usage
log "INFO" "Script started"
log "ERROR" "Something went wrong"

# Redirect all output
{
    echo "Starting process..."
    some_command
    echo "Process completed"
} >> "$LOG_FILE" 2>&1
```

---

### Category 11: Performance Monitoring

37. **How to monitor system performance?**  
Answer:

```bash
# CPU usage
top
htop
mpstat

# Memory usage
free -h
vmstat

# Disk I/O
iostat
iotop

# Network
iftop
nethogs

# Overall system
sar
dstat
```

38. **How to find and resolve high CPU usage?**  
Answer:

```bash
# Find high CPU processes
ps aux --sort=-%cpu | head -10

# Monitor specific process
top -p PID

# Get detailed process info
cat /proc/PID/status

# Check for runaway processes
pidstat 1 5

# Analyze with perf
perf top
```

39. **How to troubleshoot memory issues?**  
Answer:

```bash
# Check memory usage
free -h
cat /proc/meminfo

# Find memory-hungry processes
ps aux --sort=-%mem | head -10

# Check swap usage
swapon --show
vmstat 1 5

# Check for memory leaks
valgrind --leak-check=yes ./program

# Clear page cache (if safe)
echo 3 > /proc/sys/vm/drop_caches
```

---

### Category 12: Automation & Cron Jobs

40. **How to schedule tasks with cron?**  
Answer:

```bash
# Edit crontab
crontab -e

# Cron syntax: minute hour day month day_of_week command
# * * * * * command_to_execute
# │ │ │ │ │
# │ │ │ │ └── Day of week (0-7, both 0 and 7 are Sunday)
# │ │ │ └──── Month (1-12)
# │ │ └────── Day of month (1-31)
# │ └──────── Hour (0-23)
# └────────── Minute (0-59)

# Examples
0 * * * * /path/to/script.sh          # Every hour
0 2 * * * /path/to/backup.sh          # Daily at 2 AM
*/5 * * * * /path/to/monitor.sh       # Every 5 minutes
0 0 * * 0 /path/to/weekly-task.sh     # Weekly on Sunday
```

41. **How to create systemd services?**  
Answer:

```bash
# Create service file
sudo nano /etc/systemd/system/myservice.service

# Service file content
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

# Enable and start service
sudo systemctl daemon-reload
sudo systemctl enable myservice
sudo systemctl start myservice
sudo systemctl status myservice
```

---

### Category 13: Security & Hardening

42. **How to secure a Linux server?**  
Answer:

```bash
# Update system
apt update && apt upgrade

# Configure firewall
ufw enable
ufw allow ssh
ufw allow 80/tcp
ufw allow 443/tcp

# Disable root SSH login
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config

# Use key-based authentication
ssh-keygen -t rsa -b 4096

# Install fail2ban
apt install fail2ban

# Set up automatic security updates
apt install unattended-upgrades
```

43. **How to audit file permissions and security?**  
Answer:

```bash
# Find world-writable files
find / -type f -perm -o+w 2>/dev/null

# Find files with SUID/SGID
find / -type f \( -perm -4000 -o -perm -2000 \) 2>/dev/null

# Check for unauthorized users
cat /etc/passwd | cut -d: -f1

# Audit SSH keys
ls -la ~/.ssh/

# Check sudo privileges
sudo -l
```

---

### Category 14: Real-world Scripting Scenarios

44. **How to create a backup script?**  
Answer:

```bash
#!/bin/bash

# Backup script with rotation
BACKUP_DIR="/backup"
SOURCE_DIRS=("/home" "/etc" "/var/log")
RETENTION_DAYS=7
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_$DATE.tar.gz"

# Create backup
tar -czf "$BACKUP_DIR/$BACKUP_FILE" "${SOURCE_DIRS[@]}" 2>/dev/null

# Check if backup succeeded
if [ $? -eq 0 ]; then
    echo "Backup completed: $BACKUP_FILE"
    
    # Remove old backups
    find "$BACKUP_DIR" -name "backup_*.tar.gz" -mtime +$RETENTION_DAYS -delete
else
    echo "Backup failed!" >&2
    exit 1
fi
```

45. **How to create a log analysis script?**  
Answer:

```bash
#!/bin/bash

# Log analysis script
LOG_FILE="/var/log/nginx/access.log"
REPORT_FILE="/tmp/access_report_$(date +%Y%m%d).txt"

{
    echo "=== NGINX Access Log Report ==="
    echo "Generated: $(date)"
    echo "==============================="
    
    # Total requests
    echo "Total requests: $(wc -l < "$LOG_FILE")"
    
    # Top IP addresses
    echo -e "\nTop 10 IP addresses:"
    awk '{print $1}' "$LOG_FILE" | sort | uniq -c | sort -nr | head -10
    
    # Most accessed URLs
    echo -e "\nTop 10 URLs:"
    awk '{print $7}' "$LOG_FILE" | sort | uniq -c | sort -nr | head -10
    
    # Response codes
    echo -e "\nHTTP Response codes:"
    awk '{print $9}' "$LOG_FILE" | sort | uniq -c | sort -nr
    
    # Requests per hour
    echo -e "\nRequests per hour:"
    awk '{print $4}' "$LOG_FILE" | cut -d: -f1,2 | uniq -c
} > "$REPORT_FILE"

echo "Report generated: $REPORT_FILE"
```

46. **How to create a system monitoring script?**  
Answer:

```bash
#!/bin/bash

# System monitoring script
THRESHOLD_CPU=80
THRESHOLD_MEM=90
THRESHOLD_DISK=85
ALERT_EMAIL="admin@example.com"

# Get system metrics
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
MEM_USAGE=$(free | grep Mem | awk '{printf("%.2f"), $3/$2 * 100}')
DISK_USAGE=$(df / | awk 'NR==2 {print $5}' | cut -d'%' -f1)

# Check thresholds
ALERT=""
if (( $(echo "$CPU_USAGE > $THRESHOLD_CPU" | bc -l) )); then
    ALERT="CPU usage is high: ${CPU_USAGE}%\\n"
fi

if (( $(echo "$MEM_USAGE > $THRESHOLD_MEM" | bc -l) )); then
    ALERT="${ALERT}Memory usage is high: ${MEM_USAGE}%\\n"
fi

if [ "$DISK_USAGE" -gt "$THRESHOLD_DISK" ]; then
    ALERT="${ALERT}Disk usage is high: ${DISK_USAGE}%\\n"
fi

# Send alert if needed
if [ -n "$ALERT" ]; then
    echo -e "$ALERT" | mail -s "System Alert - $(hostname)" "$ALERT_EMAIL"
    echo "Alert sent: $ALERT"
fi
```

47. **How to create a user management script?**  
Answer:

```bash
#!/bin/bash

# User management script
USER_FILE="/tmp/users.csv"
LOG_FILE="/var/log/user_management.log"

log() {
    echo "[$(date)] $1" >> "$LOG_FILE"
}

create_user() {
    local username=$1
    local groups=$2
    
    if id "$username" &>/dev/null; then
        log "User $username already exists"
        return 1
    fi
    
    # Create user
    useradd -m -s /bin/bash "$username"
    
    # Set random password
    password=$(openssl rand -base64 12)
    echo "$username:$password" | chpasswd
    
    # Add to groups
    IFS=',' read -ra group_array <<< "$groups"
    for group in "${group_array[@]}"; do
        usermod -aG "$group" "$username"
    done
    
    log "Created user $username with groups: $groups"
    echo "Username: $username"
    echo "Password: $password"
    echo "Groups: $groups"
    echo "---"
}

# Main script
if [ ! -f "$USER_FILE" ]; then
    echo "Error: User file $USER_FILE not found"
    exit 1
fi

# Process user file
while IFS=',' read -r username groups; do
    # Skip header and empty lines
    [[ "$username" == "username" || -z "$username" ]] && continue
    
    create_user "$username" "$groups"
done < "$USER_FILE"
```

48. **How to create a Docker container management script?**  
Answer:

```bash
#!/bin/bash

# Docker container management
CONTAINER_NAME="myapp"
IMAGE_NAME="myapp:latest"
BACKUP_DIR="/backup/docker"

manage_container() {
    case $1 in
        start)
            docker-compose up -d
            ;;
        stop)
            docker-compose down
            ;;
        restart)
            docker-compose restart
            ;;
        backup)
            mkdir -p "$BACKUP_DIR"
            docker exec "$CONTAINER_NAME" bash -c 'mysqldump -u root -p$MYSQL_ROOT_PASSWORD $MYSQL_DATABASE' > "$BACKUP_DIR/backup_$(date +%Y%m%d).sql"
            ;;
        logs)
            docker-compose logs -f
            ;;
        update)
            docker-compose pull
            docker-compose down
            docker-compose up -d
            ;;
        *)
            echo "Usage: $0 {start|stop|restart|backup|logs|update}"
            exit 1
            ;;
    esac
}

# Check if Docker is running
if ! docker info &>/dev/null; then
    echo "Error: Docker is not running"
    exit 1
fi

manage_container "$1"
```

49. **How to create a SSL certificate monitoring script?**  
Answer:

```bash
#!/bin/bash

# SSL certificate monitor
DOMAINS=("example.com" "api.example.com" "app.example.com")
DAYS_THRESHOLD=30
ALERT_EMAIL="admin@example.com"

check_ssl() {
    local domain=$1
    local port=${2:-443}
    
    # Get certificate expiration
    expiration=$(echo | openssl s_client -servername "$domain" -connect "$domain:$port" 2>/dev/null | \
                 openssl x509 -noout -dates | grep 'notAfter' | cut -d= -f2)
    
    if [ -z "$expiration" ]; then
        echo "Error: Could not get certificate for $domain"
        return 1
    fi
    
    # Convert to timestamp
    exp_timestamp=$(date -d "$expiration" +%s)
    current_timestamp=$(date +%s)
    days_until_expire=$(( (exp_timestamp - current_timestamp) / 86400 ))
    
    echo "Domain: $domain - Expires in: $days_until_expire days"
    
    if [ "$days_until_expire" -le "$DAYS_THRESHOLD" ]; then
        echo "ALERT: SSL certificate for $domain expires in $days_until_expire days" | \
        mail -s "SSL Certificate Alert - $domain" "$ALERT_EMAIL"
    fi
}

# Check all domains
for domain in "${DOMAINS[@]}"; do
    check_ssl "$domain"
done
```

50. **How to create a database maintenance script?**  
Answer:

```bash
#!/bin/bash

# Database maintenance script
DB_HOST="localhost"
DB_USER="admin"
DB_PASS="password"
DB_NAME="myapp"
BACKUP_DIR="/backup/database"
RETENTION_DAYS=7

# Function to run MySQL query
mysql_query() {
    mysql -h "$DB_HOST" -u "$DB_USER" -p"$DB_PASS" -e "$1" "$DB_NAME"
}

# Backup database
backup_database() {
    local timestamp=$(date +%Y%m%d_%H%M%S)
    local backup_file="$BACKUP_DIR/backup_${timestamp}.sql"
    
    mkdir -p "$BACKUP_DIR"
    
    if mysqldump -h "$DB_HOST" -u "$DB_USER" -p"$DB_PASS" "$DB_NAME" > "$backup_file"; then
        echo "Backup created: $backup_file"
        
        # Compress backup
        gzip "$backup_file"
        echo "Backup compressed: ${backup_file}.gz"
    else
        echo "Backup failed!" >&2
        return 1
    fi
}

# Optimize tables
optimize_tables() {
    echo "Optimizing tables..."
    mysql_query "OPTIMIZE TABLE $(mysql_query "SHOW TABLES" | tail -n +2 | tr '\n' ',')"
}

# Clean up old backups
cleanup_backups() {
    find "$BACKUP_DIR" -name "backup_*.sql.gz" -mtime +$RETENTION_DAYS -delete
    echo "Old backups cleaned up"
}

# Main execution
case $1 in
    backup)
        backup_database
        ;;
    optimize)
        optimize_tables
        ;;
    cleanup)
        cleanup_backups
        ;;
    full)
        backup_database
        optimize_tables
        cleanup_backups
        ;;
    *)
        echo "Usage: $0 {backup|optimize|cleanup|full}"
        exit 1
        ;;
esac
```
