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

# Category 1: Linux Fundamentals

## Q1 — What is the Linux boot process?
**Answer**

The Linux boot process consists of several stages:

- **BIOS/UEFI**: Initializes hardware and runs POST.  
- **Bootloader**: GRUB2 (commonly) loads the kernel (files in /boot).  
- **Kernel**: Initializes devices and mounts the root filesystem.  
- **Init process**: systemd (modern) or SysV init (legacy) starts system services.  
- **Runlevels/Targets**: System services are started according to targets.  
- **Login prompt**: User can log into the system.

## Q2 — Explain Linux filesystem hierarchy.
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

## Q3 — What is the difference between hard links and soft links?
**Answer**

- **Hard link**: Direct reference to the same inode. Cannot cross filesystems. File remains until all hard links removed.  
- **Soft link (symbolic)**: Pointer to a filename (path). Can cross filesystems; becomes broken if the original is deleted.

`Examples:`
```bash
# Create hard link
ln file.txt hardlink.txt

# Create soft link
ln -s file.txt softlink.txt
```

## Practice Lab : Hard links and Soft links

```bash

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

# Observation: Changes reflect everywhere because both point to the same inode. ✓


# ------------------- Delete original file and test links -------------------

# Delete original config file 
user@568d442825b6:~$ rm /etc/app/app.conf
user@568d442825b6:~$ cat /backup/app.conf.hard
DB=prod
New entry

# Observation: 
# ✓ File still exists
# ✓ Data is safe
# ✓ Inode still valid

# Delete original log file
user@568d442825b6:~$ rm /var/log/app/app.log
user@568d442825b6:~$ cat /opt/app/app.log
cat: /opt/app/app.log: No such file or directory

# Observation: 
#  ✕ Broken link
#  ✕ “No such file or directory”


# ------------------- Move original file -------------------

user@568d442825b6:~$ mv /backup/app.conf.hard /backup/app.conf.moved
user@568d442825b6:~$ ls -li /backup/app.conf.moved 
543386 -rw-rw-r-- 1 user user 18 Jan 10 12:28 /backup/app.conf.moved

# Observation: Hard link still works (same inode)


user@568d442825b6:~$ mv /var/log/app /var/log/app_new
mv: cannot move '/var/log/app' to '/var/log/app_new': Permission denied
# ✕ Symlink breaks because path changed


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

# ✕ If /etc and /tmp are on different filesystems → fails


# ------------------- Permissions behavior -------------------

user@568d442825b6:~$ chmod 000 /backup/app.conf.moved

# ✓ Hard link respects file permissions

user@568d442825b6:~$ cat /backup/app.conf.moved
cat: /backup/app.conf.moved: Permission denied

# ✕ Soft link permissions don’t matter — target file permissions apply 

```

## Q4 — Explain Linux process states.
**Answer**

Common process states:

- **Running (R)**: Currently executing.  
- **Sleeping (S)**: Waiting for an event or resource.  
- **Stopped (T)**: Suspended by signal (e.g., SIGSTOP).  
- **Zombie (Z)**: Terminated but parent has not reaped it.  
- **Uninterruptible Sleep (D)**: Waiting for I/O; cannot be killed until I/O completes.

# Category 2: User & Permission Management

## Q5 — Explain Linux file permissions (rwx).
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

## Q6 — What is the difference between su and sudo?
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

## Q7 — How to manage users and groups in Linux?
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

## Q8 — What is umask and how does it work?
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

## Practice Lab : User & Permission Management

```sh

# ------------------- User and Group Setup -------------------

user@6cd6e0189f89:~$ whoami
user
user@6cd6e0189f89:~$ sudo useradd appuser
[sudo] password for user: 

# → Creates a new user called appuser.
# → By default, no password is set → login via su will fail.
# → This user will simulate a service account (like a Node.js app).


user@6cd6e0189f89:~$ sudo groupadd webgroup

# → Creates a new group called webgroup.
# → You can use this group to manage shared permissions.

user@6cd6e0189f89:~$ sudo usermod -aG webgroup appuser

# → Adds appuser to the webgroup.
# → -aG = append to supplementary groups.


#  ------------------- Verify User and Group existance -------------------

user@6cd6e0189f89:~$ cat /etc/passwd | grep appuser
appuser:x:1001:1001::/home/appuser:/bin/sh
user@6cd6e0189f89:~$ cat /etc/group | grep webgroup
webgroup:x:1002:appuser

# Observation: Users and groups set up correctly.


# ------------------- Create Application Directory Structure -------------------

user@6cd6e0189f89:~$ sudo mkdir -p /var/www/myapp/logs
user@6cd6e0189f89:~$ cd /var/www/myapp/logs
user@6cd6e0189f89:/var/www/myapp/logs$


# ------------------- Create Log File and Set incorrect permissions (simulate issue) -------------------

user@6cd6e0189f89:/var/www/myapp/logs$ sudo touch /var/www/myapp/logs/app.log
user@6cd6e0189f89:/var/www/myapp/logs$ sudo chown root:root /var/www/myapp/logs/app.log
user@6cd6e0189f89:/var/www/myapp/logs$ sudo chmod 644 /var/www/myapp/logs/app.log

# Permissions: 644 → rw- owner, r-- group, r-- others.

user@6cd6e0189f89:/var/www/myapp/logs$ ls
app.log
user@6cd6e0189f89:/var/www/myapp/logs$ echo "logs from app" > app.log 
bash: app.log: Permission denied

# Fails because:
# → Current user = user
# → File owned by root
# → Others only have read → no write permission


# ------------------- Changes owner to appuser, group = webgroup -------------------

user@6cd6e0189f89:/var/www/myapp/logs$ sudo chown appuser:webgroup /var/www/myapp/logs/app.log
user@6cd6e0189f89:/var/www/myapp/logs$ ls -l /var/www/myapp/logs/app.log
-rw-r--r-- 1 appuser webgroup 0 Jan 10 13:12 /var/www/myapp/logs/app.log


# ------------------- Changes file permissions → (664 = rw-rw-r--) -------------------

user@6cd6e0189f89:/var/www/myapp/logs$ sudo chmod 664 /var/www/myapp/logs/app.log
user@6cd6e0189f89:/var/www/myapp/logs$ echo "logs from app" > app.log 
bash: app.log: Permission denied

# Still fails! because, The directory /var/www/myapp/logs is owned by root with 755 permissions,
# So normal users cannot create/write files in it or redirect.

user@6cd6e0189f89:~$ whoami
user
user@6cd6e0189f89:/var/www/myapp/logs$ su appuser # Switch user 
Password: 
su: Authentication failure
user@6cd6e0189f89:/var/www/myapp/logs$ sudo -u appuser whoami
appuser
user@6cd6e0189f89:/var/www/myapp/logs$ sudo passwd -S appuser
appuser L 01/10/2026 0 99999 7 -1

# Fails because appuser has no password (passwd -S appuser shows L = locked)

user@6cd6e0189f89:/var/www/myapp/logs$ sudo -u appuser echo "logs from app" >> app.log
bash: app.log: Permission denied


# ------------------- tee runs as root, appends to the file -------------------

user@6cd6e0189f89:/var/www/myapp/logs$ echo "logs from app" | sudo tee -a app.log
logs from app
user@6cd6e0189f89:/var/www/myapp/logs$ cat app.log 
logs from app
```

# Category 3: Process Management

## Q9 — How to monitor and manage processes?
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


## Q10 — Explain signals in Linux with examples.
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

## Q11 — What are foreground and background processes?
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

## Q12 — How to find and kill a process using a specific port?
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

## Process Management

**Process:** A process is a running instance of a program.
When a program (like ls, bash, nginx) is executed, the OS loads it into memory and assigns it a Process ID (PID).    
**Each process has:**
  - PID (Process ID)
  - PPID (Parent Process ID)
  - CPU & memory usage
  - Open files
  - Environment variables
  - State (running, sleeping, stopped, zombie)

| Aspect         | Parent Process                         | Child Process                 |
| -------------- | -------------------------------------- | ----------------------------- |
| Definition     | Process that creates another process   | Process created by a parent   |
| PID            | Has its own PID                        | Has its own PID               |
| PPID           | PPID points to its own parent          | PPID points to parent process |
| Creation       | Creates child using `fork()`           | Created by parent             |
| Execution      | May continue running                   | Executes assigned task        |
| Responsibility | Reaps child process (prevents zombies) | Exits after completion        |
| Example        | `bash`                                 | `ls`, `yes`                   |


## Practice Lab: Production Linux Server Under High Load, Troubleshoot it.
  - A web application
  - A log-processing script
  - A misbehaving CPU-intensive job
  - A hung process

### Users complain:
  1. Server is slow
  2. CPU usage is high
  3. Some processes don’t terminate normally

---
#### 1. Identify the Top CPU Consumer and Lower Its Priority

```sh

# ------------------- Start a CPU-hog (if not already running) -------------------

user@28cd0a24aa57:~$ yes > /dev/null &
[1] 347

# ------------------- To display a detailed, comprehensive overview of all running processes on the system -------------------

user@28cd0a24aa57:~$ ps -aux
bad data in /proc/uptime
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   1136   640 ?        Ss   18:32   0:00 /sbin/docker-init -- /usr/bin/tini -- /usr/local/bin/judge/setup.sh
root          59  0.0  0.0   2504  1408 ?        S    18:32   0:00 /usr/bin/tini -- /usr/local/bin/judge/setup.sh
root          60  0.0  0.0   3984  3072 ?        S    18:32   0:00 /bin/bash /usr/local/bin/judge/setup.sh
root          70  0.0  0.4 1757316 15820 ?       Ssl  18:32   0:00 /ecs-execute-command-fe6f4bc4-4cf3-4cf1-bad6-3803f0dd0922/amazon-ssm-agent
root          95  0.0  0.0   4524  2944 ?        S    18:32   0:00 su - user -c ttyd -W -p 7681 --ping-interval 45 -t fontSize=16 bash
user          97  0.0  0.0   2616  1664 ?        Ss   18:32   0:00 -sh -c ttyd -W -p 7681 --ping-interval 45 -t fontSize=16 bash
user         100  0.0  0.0  10120  1280 ?        Sl   18:32   0:00 ttyd -W -p 7681 --ping-interval 45 -t fontSize 16 bash
root         125  0.0  0.6 1840724 25592 ?       Sl   18:32   0:00 /ecs-execute-command-fe6f4bc4-4cf3-4cf1-bad6-3803f0dd0922/ssm-agent-worker
root         137  0.0  0.0   2552  1408 ?        S    18:32   0:00 tail -f /dev/null
user         138  0.0  0.0   4116  3328 pts/0    Ss+  18:32   0:00 bash
user         141  0.0  0.0   2644  1792 pts/0    S+   18:32   0:00 script -a -q -f -o 500M /mnt/.script_logs/script.log --timing=/dev/null -c bash
user         142  0.0  0.0   2616  1536 pts/1    Ss   18:32   0:00 sh -c bash
user         143  0.0  0.0   4248  3328 pts/1    S    18:32   0:00 bash
user         347  0.0  0.0   2516  1024 pts/1    R    18:44   1:33 yes
user         423  0.0  0.0   5900  2816 pts/1    R+   18:46   0:00 ps -aux

# ------------------- Identify the top CPU process -------------------

user@28cd0a24aa57:~$ top
top - 18:45:03 up 0 min,  0 users,  load average: 0.49, 0.13, 0.04
Tasks:  15 total,   2 running,  13 sleeping,   0 stopped,   0 zombie
%Cpu(s): 14.4 us, 36.3 sy,  0.0 ni, 49.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   3785.6 total,    359.2 free,    357.4 used,   3069.0 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   3128.8 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND                                                                                                          
    347 user      20   0    2516   1024   1024 R 100.0   0.0   0:36.61 yes                                                                                                              
      1 root      20   0    1136    640    640 S   0.0   0.0   0:00.04 docker-init                                                                                                      
     59 root      20   0    2504   1408   1408 S   0.0   0.0   0:00.01 tini                                                                                                             
     60 root      20   0    3984   3072   2816 S   0.0   0.1   0:00.00 setup.sh                                                                                                         
     70 root      20   0 1757316  15820   9600 S   0.0   0.4   0:00.07 amazon-ssm-agen                                                                                                  
     95 root      20   0    4524   2944   2688 S   0.0   0.1   0:00.00 su                                                                                                               
     97 user      20   0    2616   1664   1664 S   0.0   0.0   0:00.00 sh                                                                                                               
    100 user      20   0   10120   1280   1024 S   0.0   0.0   0:00.02 ttyd                                                                                                             
    125 root      20   0 1840724  25592  15616 S   0.0   0.7   0:00.09 ssm-agent-worke                                                                                                  
    137 root      20   0    2552   1408   1408 S   0.0   0.0   0:00.01 tail                                                                                                             
    138 user      20   0    4116   3328   2944 S   0.0   0.1   0:00.00 bash                                                                                                             
    141 user      20   0    2644   1792   1664 S   0.0   0.0   0:00.00 script                                                                                                           
    142 user      20   0    2616   1536   1536 S   0.0   0.0   0:00.00 sh                                                                                                               
    143 user      20   0    4248   3328   2816 S   0.0   0.1   0:00.00 bash                                                                                                             
    361 user      20   0    6116   3072   2688 R   0.0   0.1   0:00.00

user@28cd0a24aa57:~$ ps -eo pid,ppid,%mem,comm,%cpu --sort=-%cpu | head
    PID    PPID %MEM COMMAND         %CPU
      1       0  0.0 docker-init      0.0
     59       1  0.0 tini             0.0
     60      59  0.0 setup.sh         0.0
     70       0  0.4 amazon-ssm-agen  0.0
     95      60  0.0 su               0.0
     97      95  0.0 sh               0.0
    100      97  0.0 ttyd             0.0
    125      70  0.6 ssm-agent-worke  0.0
    137      60  0.0 tail             0.0

user@28cd0a24aa57:~$ ps -o pid,ni,comm -p 347
    PID  NI COMMAND
    347   0 yes

# ------------------- Lower its priority (increase nice value) -------------------

user@28cd0a24aa57:~$ renice 15 -p 347
347 (process ID) old priority 0, new priority 15

# Verify:

user@28cd0a24aa57:~$ ps -o pid,ni,comm -p 347
    PID  NI COMMAND
    347  15 yes

# → Lower priority = higher nice value
# → Prevents CPU hog from starving other processes
```

---
#### 2. Gracefully Stop a Process, Then Force Kill It

```sh

# ------------------- Start a test process -------------------

user@28cd0a24aa57:~$ sleep 600 &
[3] 588

# ------------------- List out running jobs -------------------

user@28cd0a24aa57:~$ jobs -l
[1]    347 Running                 yes > /dev/null &
[2]-   422 Running                 yes > /dev/null &
[3]+   588 Running                 sleep 600 &

# ------------------- To display a detailed, comprehensive overview of all running processes on the system -------------------

user@28cd0a24aa57:~$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   1136   640 ?        Ss   18:32   0:00 /sbin/docker-init -- /usr/bin/tini -- /usr/local/bin/judge/setup.sh
root          59  0.0  0.0   2504  1408 ?        S    18:32   0:00 /usr/bin/tini -- /usr/local/bin/judge/setup.sh
root          60  0.0  0.0   3984  3072 ?        S    18:32   0:00 /bin/bash /usr/local/bin/judge/setup.sh
root          70  0.0  0.4 1757316 15948 ?       Ssl  18:32   0:00 /ecs-execute-command-fe6f4bc4-4cf3-4cf1-bad6-3803f0dd0922/amazon-ssm-agent
root          95  0.0  0.0   4524  2944 ?        S    18:32   0:00 su - user -c ttyd -W -p 7681 --ping-interval 45 -t fontSize=16 bash
user          97  0.0  0.0   2616  1664 ?        Ss   18:32   0:00 -sh -c ttyd -W -p 7681 --ping-interval 45 -t fontSize=16 bash
user         100  0.0  0.0  10188  1280 ?        Rl   18:32   0:00 ttyd -W -p 7681 --ping-interval 45 -t fontSize 16 bash
root         125  0.0  0.6 1840724 25592 ?       Sl   18:32   0:00 /ecs-execute-command-fe6f4bc4-4cf3-4cf1-bad6-3803f0dd0922/ssm-agent-worker
root         137  0.0  0.0   2552  1408 ?        S    18:32   0:00 tail -f /dev/null
user         138  0.0  0.0   4116  3328 pts/0    Ss+  18:32   0:00 bash
user         141  0.0  0.0   2644  1792 pts/0    S+   18:32   0:00 script -a -q -f -o 500M /mnt/.script_logs/script.log --timing=/dev/null -c bash
user         142  0.0  0.0   2616  1536 pts/1    Ss   18:32   0:00 sh -c bash
user         143  0.0  0.0   4248  3328 pts/1    S    18:32   0:00 bash
user         347  0.0  0.0   2516  1024 pts/1    RN   18:44   9:43 yes
user         422  0.0  0.0   2516  1152 pts/1    R    18:45   8:13 yes
user         588  0.0  0.0   2516  1280 pts/1    S    18:53   0:00 sleep 600
user         589  0.0  0.0   5900  2816 pts/1    R+   18:54   0:00 ps aux

# ------------------- Graceful stop (SIGTERM – signal 15) -------------------

user@28cd0a24aa57:~$ kill 588
user@28cd0a24aa57:~$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   1136   640 ?        Ss   18:32   0:00 /sbin/docker-init -- /usr/bin/tini -- /usr/local/bin/judge/setup.sh
root          59  0.0  0.0   2504  1408 ?        S    18:32   0:00 /usr/bin/tini -- /usr/local/bin/judge/setup.sh
root          60  0.0  0.0   3984  3072 ?        S    18:32   0:00 /bin/bash /usr/local/bin/judge/setup.sh
root          70  0.0  0.4 1757316 15948 ?       Ssl  18:32   0:00 /ecs-execute-command-fe6f4bc4-4cf3-4cf1-bad6-3803f0dd0922/amazon-ssm-agent
root          95  0.0  0.0   4524  2944 ?        S    18:32   0:00 su - user -c ttyd -W -p 7681 --ping-interval 45 -t fontSize=16 bash
user          97  0.0  0.0   2616  1664 ?        Ss   18:32   0:00 -sh -c ttyd -W -p 7681 --ping-interval 45 -t fontSize=16 bash
user         100  0.0  0.0  10120  1280 ?        Sl   18:32   0:00 ttyd -W -p 7681 --ping-interval 45 -t fontSize 16 bash
root         125  0.0  0.6 1840724 25592 ?       Sl   18:32   0:00 /ecs-execute-command-fe6f4bc4-4cf3-4cf1-bad6-3803f0dd0922/ssm-agent-worker
root         137  0.0  0.0   2552  1408 ?        S    18:32   0:00 tail -f /dev/null
user         138  0.0  0.0   4116  3328 pts/0    Ss+  18:32   0:00 bash
user         141  0.0  0.0   2644  1792 pts/0    S+   18:32   0:00 script -a -q -f -o 500M /mnt/.script_logs/script.log --timing=/dev/null -c bash
user         142  0.0  0.0   2616  1536 pts/1    Ss   18:32   0:00 sh -c bash
user         143  0.0  0.0   4248  3328 pts/1    S    18:32   0:00 bash
user         347  0.0  0.0   2516  1024 pts/1    RN   18:44  10:06 yes
user         422  0.0  0.0   2516  1152 pts/1    R    18:45   8:36 yes
user         612  0.0  0.0   5900  2816 pts/1    R+   18:54   0:00 ps aux
[3]+  Terminated              sleep 600

# ------------------- Check the process after kill -------------------

user@28cd0a24aa57:~$ ps -p 588
    PID TTY          TIME CMD
user@28cd0a24aa57:~$ ps -p 347
    PID TTY          TIME CMD
    347 pts/1    00:10:32 yes

# ------------------- Force kill (SIGKILL – signal 9) -------------------

user@28cd0a24aa57:~$ kill -9 588
bash: kill: (588) - No such process

# ------------------- Refernece for how to force kill a existing process -------------------

user@28cd0a24aa57:~$ kill -9 422

user@28cd0a24aa57:~$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   1136   640 ?        Ss   18:32   0:00 /sbin/docker-init -- /usr/bin/tini -- /usr/local/bin/judge/setup.sh
root          59  0.0  0.0   2504  1408 ?        S    18:32   0:00 /usr/bin/tini -- /usr/local/bin/judge/setup.sh
root          60  0.0  0.0   3984  3072 ?        S    18:32   0:00 /bin/bash /usr/local/bin/judge/setup.sh
root          70  0.0  0.4 1757316 15948 ?       Ssl  18:32   0:00 /ecs-execute-command-fe6f4bc4-4cf3-4cf1-bad6-3803f0dd0922/amazon-ssm-agent
root          95  0.0  0.0   4524  2944 ?        S    18:32   0:00 su - user -c ttyd -W -p 7681 --ping-interval 45 -t fontSize=16 bash
user          97  0.0  0.0   2616  1664 ?        Ss   18:32   0:00 -sh -c ttyd -W -p 7681 --ping-interval 45 -t fontSize=16 bash
user         100  0.0  0.0  10120  1280 ?        Rl   18:32   0:00 ttyd -W -p 7681 --ping-interval 45 -t fontSize 16 bash
root         125  0.0  0.6 1840724 25592 ?       Sl   18:32   0:00 /ecs-execute-command-fe6f4bc4-4cf3-4cf1-bad6-3803f0dd0922/ssm-agent-worker
root         137  0.0  0.0   2552  1408 ?        S    18:32   0:00 tail -f /dev/null
user         138  0.0  0.0   4116  3328 pts/0    Ss+  18:32   0:00 bash
user         141  0.0  0.0   2644  1792 pts/0    S+   18:32   0:00 script -a -q -f -o 500M /mnt/.script_logs/script.log --timing=/dev/null -c bash
user         142  0.0  0.0   2616  1536 pts/1    Ss   18:32   0:00 sh -c bash
user         143  0.0  0.0   4248  3328 pts/1    S    18:32   0:00 bash
user         347  0.0  0.0   2516  1024 pts/1    RN   18:44  11:08 yes
user         660  0.0  0.0   5900  2816 pts/1    R+   18:55   0:00 ps aux
[2]+  Killed                  yes > /dev/null
```

---
#### 3. Find a Zombie Process and Fix It

```sh

# ------------------- Create a zombie process -------------------

user@28cd0a24aa57:~$ bash -c 'sleep 1 & exit'

user@28cd0a24aa57:~$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   1136   640 ?        Ss   18:32   0:00 /sbin/docker-init -- /usr/bin/tini -- /usr/local/bin/judge/setup.sh
root          59  0.0  0.0   2504  1408 ?        S    18:32   0:00 /usr/bin/tini -- /usr/local/bin/judge/setup.sh
root          60  0.0  0.0   3984  3072 ?        S    18:32   0:00 /bin/bash /usr/local/bin/judge/setup.sh
root          70  0.0  0.4 1757316 15948 ?       Ssl  18:32   0:00 /ecs-execute-command-fe6f4bc4-4cf3-4cf1-bad6-3803f0dd0922/amazon-ssm-agent
root          95  0.0  0.0   4524  2944 ?        S    18:32   0:00 su - user -c ttyd -W -p 7681 --ping-interval 45 -t fontSize=16 bash
user          97  0.0  0.0   2616  1664 ?        Ss   18:32   0:00 -sh -c ttyd -W -p 7681 --ping-interval 45 -t fontSize=16 bash
user         100  0.0  0.0  10124  1280 ?        Rl   18:32   0:00 ttyd -W -p 7681 --ping-interval 45 -t fontSize 16 bash
root         125  0.0  0.6 1840724 25592 ?       Sl   18:32   0:00 /ecs-execute-command-fe6f4bc4-4cf3-4cf1-bad6-3803f0dd0922/ssm-agent-worker
root         137  0.0  0.0   2552  1408 ?        S    18:32   0:00 tail -f /dev/null
user         138  0.0  0.0   4116  3328 pts/0    Ss+  18:32   0:00 bash
user         141  0.0  0.0   2644  1792 pts/0    R+   18:32   0:00 script -a -q -f -o 500M /mnt/.script_logs/script.log --timing=/dev/null -c bash
user         142  0.0  0.0   2616  1536 pts/1    Ss   18:32   0:00 sh -c bash
user         143  0.0  0.0   4248  3328 pts/1    S    18:32   0:00 bash
user         347  0.0  0.0   2516  1024 pts/1    RN   18:44  12:30 yes
user         691  0.0  0.0   5900  2560 pts/1    R+   18:57   0:00 ps aux

# ------------------- Identify zombie -------------------

user@28cd0a24aa57:~$ ps aux | awk '$8 ~ /Z/ { print }'
user@28cd0a24aa57:~$ ps -eo pid,ppid,state,cmd | grep Z
    733     143 S grep --color=auto Z

user@28cd0a24aa57:~$ kill -9 733   
bash: kill: (733) - No such process
# ✕ Zombies cannot be killed directly 

# ------------------- We must kill the parent process. -------------------

user@28cd0a24aa57:~$ kill -9 143

bad data in /proc/uptime
bad data in /proc/uptime
bad data in /proc/uptime
bad data in /proc/uptime
bad data in /proc/uptime
bad data in /proc/uptime
Killed

user@28cd0a24aa57:~$ ps aux | grep Z
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
user         802  0.0  0.0   3312  1664 pts/0    S+   19:00   0:00 grep --color=auto Z

# Note:
# → Zombies exist because parent didn’t reap child
# → Killing parent cleans them up

```

---
#### 4. Move a Stopped Job to Background and Foreground

```sh

# ------------------- Move a Stopped Job to Background and Foreground -------------------

user@28cd0a24aa57:~$ sleep 300

# Suspend it using Ctrl + z
^Z                                       
[1]+  Stopped                 sleep 300  

# ------------------- Move to background -------------------

user@28cd0a24aa57:~$ bg %1
[1]+ sleep 300 &

# ------------------- List the jobs running -------------------

user@28cd0a24aa57:~$ jobs -l
[1]+   884 Running                 sleep 300 &

# ------------------- Move to foreground -------------------

user@28cd0a24aa57:~$ fg %1
sleep 300
^Z
[1]+  Stopped                 sleep 300

# ------------------- Job : 2 -------------------

user@28cd0a24aa57:~$ sleep 200 &
[2] 898

user@28cd0a24aa57:~$ fg %1
sleep 300
^Z      
[1]+  Stopped                 sleep 300

user@28cd0a24aa57:~$ fg %2
sleep 200
^Z
[2]+  Stopped                 sleep 200
```

# Category 4: Disk & Filesystem Management

## Q13 — How to check disk usage in Linux?
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

## Q14 — Explain Linux filesystem types.
**Answer**

#### Linux Filesystems:

| Filesystem    | Type         | Description                                                                             | Common Use Case                   | Example               |
| ------------- | ------------ | --------------------------------------------------------------------------------------- | --------------------------------- | --------------------- |
| **ext4**      | Disk FS      | Most widely used Linux filesystem. Supports journaling, large files, and fast recovery. | Root (`/`) filesystem, data disks | Ubuntu root partition |
| **ext3**      | Disk FS      | Older version of ext4 with journaling support.                                          | Legacy Linux systems              | Older servers         |
| **ext2**      | Disk FS      | No journaling → faster but unsafe after crash.                                          | Boot partitions, USB drives       | `/boot` partition     |
| **XFS**       | Disk FS      | High-performance filesystem, excellent for large files.                                 | Databases, media servers          | RHEL default FS       |
| **Btrfs**     | Disk FS      | Advanced FS with snapshots, compression, and subvolumes.                                | Modern Linux, backups             | SUSE Linux            |
| **FAT32**     | Disk FS      | Simple, widely supported filesystem.                                                    | USB drives, cameras               | Pen drives            |
| **NTFS**      | Disk FS      | Windows filesystem. Read/write supported in Linux.                                      | Dual-boot systems                 | Windows C: drive      |
| **OverlayFS** | Virtual FS   | Combines multiple filesystems into one.                                                 | Containers (Docker)               | Docker image layers   |
| **tmpfs**     | Memory FS    | Stored in RAM, very fast, cleared on reboot.                                            | Temporary files                   | `/tmp`, `/run`        |
| **procfs**    | Virtual FS   | Kernel process information.                                                             | Process monitoring                | `/proc/cpuinfo`       |
| **sysfs**     | Virtual FS   | Kernel and hardware info.                                                               | Device management                 | `/sys/class/net`      |
| **devtmpfs**  | Virtual FS   | Device files created dynamically.                                                       | Device access                     | `/dev/sda`            |
| **NFS**       | Network FS   | Access files over network.                                                              | Shared storage                    | `/mnt/nfs`            |


## Q15 — How to mount and unmount filesystems?
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

## Q16 — What is LVM and its components?
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

## Disk, Partition, Filesystem & Mount

| Concept        | What it is                         | Linux Example                          | Simple Example (Analogy)    | Practical Example                              |
| -------------- | ---------------------------------- | -------------------------------------- | --------------------------- | ---------------------------------------------- |
| **Disk**       | Physical or virtual storage device | `/dev/sda`, `/dev/sdb`, `/dev/nvme0n1` |  A warehouse                | A 100GB cloud disk attached to an EC2 instance |
| **Partition**  | Logical division of a disk         | `/dev/sda1`, `/dev/sdb1`               |  Rooms inside a warehouse   | One partition for OS, one for logs             |
| **Filesystem** | Structure that organizes data      | `ext4`, `xfs`                          |  Filing system in a room    | ext4 used to store Linux files                 |
| **Mount**      | Linking filesystem to a directory  | `/mnt/data`, `/var/log`                |  Door to access a room      | Mount disk to `/var/log`                       |

```text
Real Disk (Docker VM /dev/vda)
         │
         ├─ Contains Filesystem (ext4) where we can create files
         │
         └─ /tmp/virtualdisk.img  ← This file is just a “container” for practice
                 │
                 └─ Loop Device (/dev/loop0)
                       │
                       └─ Partition (/dev/loop0p1)
                             │
                             └─ Filesystem (ext4)
                                   │
                                   └─ Mount (/mnt/virtualdisk) → create files here

```

## Practice Lab : Disk & Filesystem Management

**You are a Linux/System Engineer managing a web server. /var is filling up due to logs and a new disk has been attached to the server**

  1. Partition the disk
  2. Create a filesystem
  3. Mount it persistently
  4. Move application data
  5. Handle disk usage issues
  6. Simulate and recover from common failures
     
---
#### 1. Create a “virtual disk” file

```sh

# ------------------- We create a file that acts as a disk. Later we can format it and mount it. -------------------

ubuntu:~$ sudo dd if=/dev/zero of=/tmp/virtualdisk.img bs=1M count=100
100+0 records in
100+0 records out
104857600 bytes (105 MB, 100 MiB) copied, 0.0402209 s, 2.6 GB/s

# Explaination:
# dd creates a blank file of 100MB
# bs=1M count=100 → 100 * 1MB = 100MB
# /tmp/virtualdisk.img → our “virtual disk”


# ------------------- Create a loop device → Loop device allows Linux to treat a file as a block device. -------------------

ubuntu:~$ sudo losetup -fP /tmp/virtualdisk.img

ubuntu:~$ losetup -a
/dev/loop0: [64769]:1094 (/tmp/virtualdisk.img)

# Explanation:
# /dev/loop0 now acts like a real disk
# We can partition, format, and mount it

```
---
#### 2. Partition the virtual disk

```sh

# ------------------- Use fdisk to create a single partition. -------------------

ubuntu:~$ sudo fdisk /dev/loop0

Welcome to fdisk (util-linux 2.39.3).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS (MBR) disklabel with disk identifier 0x038f6069.

Command (m for help): m

Help:

  DOS (MBR)
   a   toggle a bootable flag
   b   edit nested BSD disklabel
   c   toggle the dos compatibility flag

  Generic
   d   delete a partition
   F   list free unpartitioned space
   l   list known partition types
   n   add a new partition
   p   print the partition table
   t   change a partition type
   v   verify the partition table
   i   print information about a partition

  Misc
   m   print this menu
   u   change display/entry units
   x   extra functionality (experts only)

  Script
   I   load disk layout from sfdisk script file
   O   dump disk layout to sfdisk script file

  Save & Exit
   w   write table to disk and exit
   q   quit without saving changes

  Create a new label
   g   create a new empty GPT partition table
   G   create a new empty SGI (IRIX) partition table
   o   create a new empty MBR (DOS) partition table
   s   create a new empty Sun partition table

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): e
Partition number (1-4, default 1): 1
First sector (2048-204799, default 2048): 2048
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-204799, default 204799): 204799

Created a new partition 1 of type 'Extended' and of size 99 MiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.


# ------------------- Check partition -------------------

ubuntu:~$ lsblk
NAME      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0       7:0    0  100M  0 loop 
`-loop0p1 259:1    0    1K  0 part 
vda       253:0    0   20G  0 disk 
|-vda1    253:1    0   19G  0 part /
|-vda14   253:14   0    4M  0 part 
|-vda15   253:15   0  106M  0 part /boot/efi
`-vda16   259:0    0  913M  0 part /boot

ubuntu:~$ sudo mkfs.ext4 /dev/loop0p1
mke2fs 1.47.0 (5-Feb-2023)
mkfs.ext4: inode_size (256) * inodes_count (0) too big for a
        filesystem with 0 blocks, specify higher inode_ratio (-i)
        or lower inode count (-N).

# Explanation:
# We created loop0p1 → partition 1 on the virtual disk
# Concept is exactly same as /dev/vdb1 on real VM
```
---
#### 3. Create a filesystem

```sh

# ------------------- Format the partition with ext4. -------------------

ubuntu:~$ sudo mkfs.ext4 /dev/loop0
mke2fs 1.47.0 (5-Feb-2023)
Found a dos partition table in /dev/loop0
Proceed anyway? (y,N) y
Discarding device blocks: done                            
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done

# Explanation:
# Now /dev/loop0p1 is an ext4 filesystem
# Can store files like any real disk
```
---
#### 4. Mount and unmount the filesystem

```sh

# ------------------- Mount partition to a directory to use it. -------------------

ubuntu:~$ sudo mkdir /mnt/virtualdisk
ubuntu:~$ sudo mount /dev/loop0 /mnt/virtualdisk

# ------------------- Check mount: -------------------

ubuntu:~$ df -h /mnt/virtualdisk
Filesystem      Size  Used Avail Use% Mounted on
/dev/loop0       90M   24K   83M   1% /mnt/virtualdisk

ubuntu:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
loop0     7:0    0  100M  0 loop /mnt/virtualdisk
vda     253:0    0   20G  0 disk 
|-vda1  253:1    0   19G  0 part /
|-vda14 253:14   0    4M  0 part 
|-vda15 253:15   0  106M  0 part /boot/efi
`-vda16 259:0    0  913M  0 part /boot

# Explanation:
# df -hT shows filesystem type and size
# /mnt/virtualdisk is now ready for practice
```
---
#### 5. Unmount and cleanup

```sh

# ------------------- After practice, unmount and detach loop device. -------------------

ubuntu:~$ umount /mnt/virtualdisk && losetup -d /dev/loop0

ubuntu:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda     253:0    0   20G  0 disk 
|-vda1  253:1    0   19G  0 part /
|-vda14 253:14   0    4M  0 part 
|-vda15 253:15   0  106M  0 part /boot/efi
`-vda16 259:0    0  913M  0 part /boot

# Explanation:
# /mnt/virtualdisk is unmounted
# Loop device is detached
# File removed — lab is clean
```
---
#### 6. Troubleshooting disk usage in Linux

```sh

# ------------------- Disk space by filesystem -------------------

user@b6dcf056802d:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay          29G  7.9G   22G  28% /
tmpfs            64M     0   64M   0% /dev
tmpfs            64M  876K   64M   2% /run
tmpfs           4.0M     0  4.0M   0% /run/lock
shm              64M     0   64M   0% /dev/shm
/dev/root        29G  7.9G   22G  28% /var/log/amazon/ssm
tmpfs           1.9G     0  1.9G   0% /proc/acpi
tmpfs           1.9G     0  1.9G   0% /proc/scsi
tmpfs           1.9G     0  1.9G   0% /sys/firmware
tmpfs           1.9G   20K  1.9G   1% /tmp
tmpfs            52M  8.0K   52M   1% /run/user/1001

# ------------------- Disk usage by directory -------------------

user@b6dcf056802d:~$ sudo du -sh /var/*
4.0K    /var/backups
1.7M    /var/cache
48M     /var/lib
4.0K    /var/local
0       /var/lock
1.1G    /var/log
4.0K    /var/mail
4.0K    /var/opt
0       /var/run
4.0K    /var/spool
12K     /var/tmp

user@b6dcf056802d:~$ sudo du -sh /var/log/*
0       /var/log/README
12K     /var/log/alternatives.log
28K     /var/log/amazon
80K     /var/log/apt
1.1G    /var/log/big.log
60K     /var/log/bootstrap.log
0       /var/log/btmp
180K    /var/log/dpkg.log
8.0K    /var/log/journal
0       /var/log/lastlog
4.0K    /var/log/private
0       /var/log/wtmp

# ------------------- Detailed per-subdir -------------------

user@b6dcf056802d:~$ sudo du -h --max-depth=1 /home
16K     /home/ubuntu
2.1G    /home/user
2.1G    /home

# ------------------- Find large files than 100M -------------------

user@b6dcf056802d:~$ sudo find /var/log -type f -name "*.log" -size +100M -exec ls -lh {} \; 2>/dev/null
-rw-r--r-- 1 root root 1.0G Jan 11 15:34 /var/log/big.log

# ------------------- Find files less than 100M -------------------

user@b6dcf056802d:~$ sudo find /var/log -type f -name "*.log" -size -100M -exec ls -lh {} \; 2>/dev/null
-rw-r--r-- 1 root root 20K Jan 11 15:29 /var/log/apt/history.log
-rw-r----- 1 root adm 43K Jan 11 15:29 /var/log/apt/term.log
-rw-r--r-- 1 root root 59K Apr 15  2025 /var/log/bootstrap.log
-rw-r--r-- 1 root root 8.9K Jun  6  2025 /var/log/alternatives.log
-rw-r--r-- 1 root root 177K Jan 11 15:29 /var/log/dpkg.log
-rw-r--r-- 1 root root 9.8K Jan 11 15:44 /var/log/amazon/ssm/amazon-ssm-agent.log
```

### Real-World Troubleshooting Scenarios (Simulated)

| Problem           | Cause              | Resolution                              |
| ----------------- | ------------------ | --------------------------------------- |
| Cannot write file | Mounted read-only  | `mount -o remount,rw /mnt/virtualdisk`  |
| Disk full         | Filled loopback FS | Check `df -h` and delete files          |
| Cannot mount      | Partition missing  | `lsblk` or recreate partition           |
| Permission denied | Wrong ownership    | `sudo chown user:user /mnt/virtualdisk` |

### Command Summary

| Command   | Purpose           |
| --------- | ----------------- |
| lsblk     | View disks        |
| df        | Disk usage        |
| du        | Find space hogs   |
| fdisk     | Partition disk    |
| mkfs      | Create filesystem |
| mount     | Attach filesystem |
| fstab     | Persistent mount  |
| fsck      | Repair FS         |
| resize2fs | Resize FS         |


# Category 5: Networking

## Q17 — How to check network configuration?
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

### 1. Checking Network configurations

```sh

# ------------------- Interfaces and IPs -------------------

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
       valid_lft 86310789sec preferred_lft 75521589sec
    inet6 fe80::1995:6751:3ede:c0c5/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1454 qdisc noqueue state DOWN group default 
    link/ether 62:1a:be:19:8a:99 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever

# ------------------- Routing table -------------------

ubuntu:~$ ip route show
default via 172.30.1.1 dev enp1s0 proto dhcp src 172.30.1.2 metric 1002 mtu 1500 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
172.30.1.0/24 dev enp1s0 proto dhcp scope link src 172.30.1.2 metric 1002 mtu 1500

# ------------------- Listening ports and sockets -------------------

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

# ------------------- DNS resolution -------------------

ubuntu:~$ nslookup google.com
Server:         8.8.8.8
Address:        8.8.8.8#53

Non-authoritative answer:
Name:   google.com
Address: 142.250.185.206
Name:   google.com
Address: 2a00:1450:4001:812::200e

ubuntu:~$ dig google.com

# ; <<>> DiG 9.18.39-0ubuntu0.24.04.2-Ubuntu <<>> google.com
# ;; global options: +cmd
# ;; Got answer:
# ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11992
# ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

# ;; OPT PSEUDOSECTION:
# ; EDNS: version: 0, flags:; udp: 512
# ;; QUESTION SECTION:
# ;google.com.                    IN      A

# ;; ANSWER SECTION:
# google.com.             156     IN      A       142.250.185.206

# ;; Query time: 5 msec
# ;; SERVER: 8.8.8.8#53(8.8.8.8) (UDP)
# ;; WHEN: Sun Jan 11 18:55:39 UTC 2026
# ;; MSG SIZE  rcvd: 55
```

## Q18 — How to troubleshoot network connectivity?
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

```sh

# ------------------- Basic connectivity -------------------

ubuntu:~$ ping -c 4 google.com
PING google.com (142.250.185.206) 56(84) bytes of data.
64 bytes from fra16s52-in-f14.1e100.net (142.250.185.206): icmp_seq=1 ttl=117 time=3.96 ms
64 bytes from fra16s52-in-f14.1e100.net (142.250.185.206): icmp_seq=2 ttl=117 time=4.16 ms
64 bytes from fra16s52-in-f14.1e100.net (142.250.185.206): icmp_seq=3 ttl=117 time=3.88 ms
64 bytes from fra16s52-in-f14.1e100.net (142.250.185.206): icmp_seq=4 ttl=117 time=4.56 ms

--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 3.881/4.140/4.559/0.262 ms

# ------------------- traceroute google.com -------------------

ubuntu:~$ traceroute google.com
traceroute to google.com (142.250.185.206), 30 hops max, 60 byte packets
 1  172.30.1.1 (172.30.1.1)  0.100 ms  0.074 ms  0.063 ms
 2  10.8.0.1 (10.8.0.1)  5.990 ms  5.969 ms  5.989 ms
 3  192.168.1.1 (192.168.1.1)  5.950 ms  5.936 ms  5.931 ms
 4  irb-2.router-02.fra1.civo.io (74.220.24.3)  5.923 ms  5.892 ms  5.882 ms
 5  lag-111.ear5.Frankfurt1.Level3.net (62.67.68.101)  7.218 ms  7.064 ms  8.096 ms
 6  * * *
 7  72.14.213.242 (72.14.213.242)  4.171 ms  8.588 ms  8.557 ms
 8  192.178.109.241 (192.178.109.241)  8.525 ms 192.178.109.157 (192.178.109.157)  8.604 ms 192.178.109.241 (192.178.109.241)  12.359 ms
 9  142.250.213.213 (142.250.213.213)  8.460 ms  8.489 ms  8.414 ms
10  fra16s52-in-f14.1e100.net (142.250.185.206)  8.482 ms  8.438 ms  12.053 ms

# ------------------- DNS -------------------

ubuntu:~$ nslookup google.com
Server:         8.8.8.8
Address:        8.8.8.8#53

Non-authoritative answer:
Name:   google.com
Address: 142.250.185.206
Name:   google.com
Address: 2a00:1450:4001:812::200e

# ------------------- Check TCP port reachability -------------------

ubuntu:~$ telnet google.com 80
Trying 142.250.185.206...
Connected to google.com.
Escape character is '^]'.
^]
Connection closed by foreign host.

ubuntu:~$ nc -v google.com 80
Connection to google.com (142.250.185.206) 80 port [tcp/http] succeeded!
^C

# ------------------- Check firewall rules -------------------

ubuntu:~$ iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
DOCKER-USER  all  --  anywhere             anywhere            
DOCKER-FORWARD  all  --  anywhere             anywhere            

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         

Chain DOCKER (1 references)
target     prot opt source               destination         
DROP       all  --  anywhere             anywhere            

Chain DOCKER-BRIDGE (1 references)
target     prot opt source               destination         
DOCKER     all  --  anywhere             anywhere            

Chain DOCKER-CT (1 references)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED

Chain DOCKER-FORWARD (1 references)
target     prot opt source               destination         
DOCKER-CT  all  --  anywhere             anywhere            
DOCKER-ISOLATION-STAGE-1  all  --  anywhere             anywhere            
DOCKER-BRIDGE  all  --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere            

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
target     prot opt source               destination         
DOCKER-ISOLATION-STAGE-2  all  --  anywhere             anywhere            

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
target     prot opt source               destination         
DROP       all  --  anywhere             anywhere            

Chain DOCKER-USER (1 references)
target     prot opt source               destination         
ubuntu:~$
```

## Q19 — Explain common network configuration files.
**Answer**

- `/etc/hosts` — Static hostname resolution.  
- `/etc/resolv.conf` — DNS resolver configuration.  
- `/etc/nsswitch.conf` — Name service switch order.  
- `/etc/sysconfig/network-scripts/` — RHEL/CentOS network scripts.  
- `/etc/netplan/` — Ubuntu (netplan) configuration.


## Q20 — How to use tcpdump for network analysis?
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

# Category 6: Package Management

## Q21 — Compare apt, yum, and dpkg.
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

## Q22 — How to find which package provides a specific file?
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

## Q23 — How to create a simple Debian package?
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
### Practice Lab : Package & Service Management

- Learn package management with apt (Debian/Ubuntu)
- Learn systemd service management using systemctl
- Practice troubleshooting package and service issues

```sh

# ------------------- Check installed packages -------------------

ubuntu:~$ dpkg -l | grep curl
ii  curl                             8.5.0-2ubuntu10.6                       amd64        command line tool for transferring data with URL syntax
ii  libcurl3t64-gnutls:amd64         8.5.0-2ubuntu10.6                       amd64        easy-to-use client-side URL transfer library (GnuTLS flavour)
ii  libcurl4t64:amd64                8.5.0-2ubuntu10.6                       amd64        easy-to-use client-side URL transfer library (OpenSSL flavour)

# Search for a package

ubuntu:~$ apt search git
# Finds available packages matching a keyword.

# ------------------- Check package info and dependencies -------------------

ubuntu:~$ apt show git
Package: git
Version: 1:2.43.0-1ubuntu7.3
Priority: optional
Section: vcs
Origin: Ubuntu
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Original-Maintainer: Jonathan Nieder <jrnieder@gmail.com>
Bugs: https://bugs.launchpad.net/ubuntu/+filebug
Installed-Size: 22.2 MB
Provides: git-completion, git-core
Depends: libc6 (>= 2.38), libcurl3t64-gnutls (>= 7.56.1),
...

ubuntu:~$ apt depends git
git
  Depends: libc6 (>= 2.38)
  Depends: libcurl3t64-gnutls (>= 7.56.1)
  Depends: libexpat1 (>= 2.0.1)
  Depends: libpcre2-8-0 (>= 10.34)
  Depends: zlib1g (>= 1:1.2.2)
  Depends: perl...
 
ubuntu:~$ sudo apt update && sudo apt install git -y 2>/dev/null
Hit:1 http://archive.ubuntu.com/ubuntu noble InRelease
Get:2 http://archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]    
Get:3 http://archive.ubuntu.com/ubuntu noble-backports InRelease [126 kB]             
Hit:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Get:5 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 Components [175 kB]
Get:6 http://archive.ubuntu.com/ubuntu noble-updates/universe amd64 Components [378 kB]
Get:7 http://archive.ubuntu.com/ubuntu noble-updates/restricted amd64 Components [212 B]
Get:8 http://archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 Components [940 B]
Get:9 http://archive.ubuntu.com/ubuntu noble-backports/main amd64 Components [7280 B]
Get:10 http://archive.ubuntu.com/ubuntu noble-backports/universe amd64 Components [10.5 kB]
Get:11 http://archive.ubuntu.com/ubuntu noble-backports/restricted amd64 Components [212 B]
Get:12 http://archive.ubuntu.com/ubuntu noble-backports/multiverse amd64 Components [212 B]
Fetched 824 kB in 1s (819 kB/s)                                           
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
20 packages can be upgraded. Run 'apt list --upgradable' to see them.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
git is already the newest version (1:2.43.0-1ubuntu7.3).
git set to manually installed.
The following package was automatically installed and is no longer required:
  squashfs-tools
Use 'sudo apt autoremove' to remove it.
0 upgraded, 0 newly installed, 0 to remove and 20 not upgraded.

# ------------------- Remove a package -------------------

ubuntu:~$ sudo apt remove git -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  git-man liberror-perl squashfs-tools
Use 'sudo apt autoremove' to remove them.
The following packages will be REMOVED:
  git ubuntu-server
0 upgraded, 0 newly installed, 2 to remove and 20 not upgraded.
After this operation, 22.2 MB disk space will be freed.
(Reading database ... 113845 files and directories currently installed.)
Removing ubuntu-server (1.539.2) ...
Removing git (1:2.43.0-1ubuntu7.3) ...

# ------------------- Update system -------------------

ubuntu:~$ sudo apt update && sudo apt upgrade -y 2>dev/null
Hit:1 http://archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://archive.ubuntu.com/ubuntu noble-updates InRelease             
Hit:3 http://archive.ubuntu.com/ubuntu noble-backports InRelease           
Get:4 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Get:5 http://security.ubuntu.com/ubuntu noble-security/main amd64 Components [21.6 kB]
Get:6 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Components [71.4 kB]
Get:7 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Components [212 B]
Get:8 http://security.ubuntu.com/ubuntu noble-security/multiverse amd64 Components [212 B]
Fetched 220 kB in 1s (240 kB/s)                                         
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
20 packages can be upgraded. Run 'apt list --upgradable' to see them.
bash: dev/null: No such file or directory
ubuntu:~$ sudo systemctl status apache2
Unit apache2.service could not be found.

# ------------------- systemd/ Service Management with systemctl -------------------

ubuntu:~$ sudo systemctl start apache2
ubuntu:~$ sudo systemctl stop apache2
ubuntu:~$ sudo systemctl restart apache2
ubuntu:~$ sudo systemctl status apache2
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/apache2.service; enabled; preset: enabled)
     Active: active (running) since Sun 2026-01-11 19:17:01 UTC; 9s ago
       Docs: https://httpd.apache.org/docs/2.4/
    Process: 3035 ExecStart=/usr/sbin/apachectl start (code=exited, status=0/SUCCESS)
   Main PID: 3038 (apache2)
      Tasks: 55 (limit: 2237)
     Memory: 5.1M (peak: 5.1M)
        CPU: 18ms
     CGroup: /system.slice/apache2.service
             ├─3038 /usr/sbin/apache2 -k start
             ├─3040 /usr/sbin/apache2 -k start
             └─3041 /usr/sbin/apache2 -k start

Jan 11 19:17:01 ubuntu systemd[1]: Starting apache2.service - The Apache HTTP Server...
Jan 11 19:17:01 ubuntu apachectl[3037]: AH00558: apache2: Could not reliably determine the servers fully qualified domain name, using 127.0.0.1. Set the 'ServerName' directive globally to suppress this message
Jan 11 19:17:01 ubuntu systemd[1]: Started apache2.service - The Apache HTTP Server.
```
### Commands Summanry

| Concept                 | Command                                   | Use Case                          |
| ----------------------- | ----------------------------------------- | --------------------------------- |
| Check installed package | `dpkg -l`, `rpm -q`                       | Confirm if a package exists       |
| Search package          | `apt search`, `yum search`                | Find package name before install  |
| Install package         | `apt install`, `yum install`              | Install software and dependencies |
| Remove package          | `apt remove/purge`, `yum remove`          | Clean unwanted software           |
| Update packages         | `apt update && apt upgrade`, `yum update` | Keep system up-to-date            |
| Service status          | `systemctl status <service>`              | Check if service is running       |
| Start/Stop/Restart      | `systemctl start/stop/restart`            | Control services                  |
| Enable/Disable          | `systemctl enable/disable`                | Start services at boot            |
| Troubleshoot service    | `journalctl -xe`, `configtest`            | Find reason service failed        |


# Category 7: Shell Scripting Fundamentals

## Q24 — What is a shebang line and why is it important?
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

## Q25 — Explain different types of variables in shell scripting.
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
  
| Variable | Meaning                          |
| -------- | -------------------------------- |
| `$0`     | Script name                      |
| `$1`     | First argument                   |
| `$2`     | Second argument                  |
| `$#`     | Number of arguments              |
| `$@`     | All arguments                    |
| `$*`     | All arguments as a single string |
| `$?`     | Exit status                      |
| `$$`     | Script PID                       |
| `$!`     | Last background PID              |

## Q26 — How to handle command-line arguments?
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

## Q27 — Explain different types of quotes in shell.
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

# Category 8: Advanced Shell Scripting

## Q28 — How to use arrays in shell scripting?
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

## Q29 — Explain process substitution.
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

## Q30 — How to create functions in shell scripts?
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

# Category 9: Text Processing

## Q31 — How to use grep, sed, and awk effectively?
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

## Q32 — How to parse CSV files in shell?
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

## Q33 — How to validate and sanitize input in scripts?
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

### Practice Lab: Text Processing for Log Analysis & Report Generation

- Filter logs using grep
- Clean/transform logs using sed
- Analyze & generate reports using awk

```sh
# ------------------- Create Sample Log File -------------------

ubuntu:~$ touch access.log
ubuntu:~$ cat <<EOF > access.log
192.168.1.10 - - [10/Jan/2026:10:15:20] "GET /index.html HTTP/1.1" 200 1024
192.168.1.11 - - [10/Jan/2026:10:16:45] "POST /login HTTP/1.1" 401 512
192.168.1.12 - - [10/Jan/2026:10:17:10] "GET /dashboard HTTP/1.1" 200 2048
192.168.1.10 - - [10/Jan/2026:10:18:55] "GET /admin HTTP/1.1" 403 256
192.168.1.13 - - [10/Jan/2026:10:19:30] "GET /index.html HTTP/1.1" 200 1024
EOF

# ------------------- Use grep (Filtering) -------------------

# ------------------- Find All Failed Requests (401 & 403) -------------------

ubuntu:~$ grep -E "401|403" access.log
192.168.1.11 - - [10/Jan/2026:10:16:45] "POST /login HTTP/1.1" 401 512
192.168.1.10 - - [10/Jan/2026:10:18:55] "GET /admin HTTP/1.1" 403 256

# ------------------- Find Requests from a Specific IP -------------------

ubuntu:~$ grep "192.168.1.10" access.log
192.168.1.10 - - [10/Jan/2026:10:15:20] "GET /index.html HTTP/1.1" 200 1024
192.168.1.10 - - [10/Jan/2026:10:18:55] "GET /admin HTTP/1.1" 403 256

# ------------------- Use sed (Transformation) -------------------

# ------------------- Mask IP Addresses (Security Practice) -------------------

ubuntu:~$ sed 's/[0-9]\+\.[0-9]\+\.[0-9]\+\.[0-9]\+/XXX.XXX.XXX.XXX/' access.log
XXX.XXX.XXX.XXX - - [10/Jan/2026:10:15:20] "GET /index.html HTTP/1.1" 200 1024
XXX.XXX.XXX.XXX - - [10/Jan/2026:10:16:45] "POST /login HTTP/1.1" 401 512
XXX.XXX.XXX.XXX - - [10/Jan/2026:10:17:10] "GET /dashboard HTTP/1.1" 200 2048
XXX.XXX.XXX.XXX - - [10/Jan/2026:10:18:55] "GET /admin HTTP/1.1" 403 256
XXX.XXX.XXX.XXX - - [10/Jan/2026:10:19:30] "GET /index.html HTTP/1.1" 200 1024

# ------------------- Replace HTTP with HTTPS -------------------

ubuntu:~$ sed 's/HTTP/HTTPS/g' access.log
192.168.1.10 - - [10/Jan/2026:10:15:20] "GET /index.html HTTPS/1.1" 200 1024
192.168.1.11 - - [10/Jan/2026:10:16:45] "POST /login HTTPS/1.1" 401 512
192.168.1.12 - - [10/Jan/2026:10:17:10] "GET /dashboard HTTPS/1.1" 200 2048
192.168.1.10 - - [10/Jan/2026:10:18:55] "GET /admin HTTPS/1.1" 403 256
192.168.1.13 - - [10/Jan/2026:10:19:30] "GET /index.html HTTPS/1.1" 200 1024

# ------------------- Remove Date & Time -------------------

ubuntu:~$ sed 's/\[.*\]//g' access.log
192.168.1.10 - -  "GET /index.html HTTP/1.1" 200 1024
192.168.1.11 - -  "POST /login HTTP/1.1" 401 512
192.168.1.12 - -  "GET /dashboard HTTP/1.1" 200 2048
192.168.1.10 - -  "GET /admin HTTP/1.1" 403 256
192.168.1.13 - -  "GET /index.html HTTP/1.1" 200 1024

# ------------------- Use awk (Analysis) -------------------

# ------------------- Extract IP and Status Code -------------------

ubuntu:~$ awk '{print $1, $9}' access.log
192.168.1.10 1024
192.168.1.11 512
192.168.1.12 2048
192.168.1.10 256
192.168.1.13 1024

# ------------------- Count Requests Per IP -------------------

ubuntu:~$ awk '{print $1}' access.log | sort | uniq -c
      2 192.168.1.10
      1 192.168.1.11
      1 192.168.1.12
      1 192.168.1.13

# ------------------- Show Only Successful Requests (200) -------------------

ubuntu:~$ awk '$9 == 200 {print $1, $7}' access.log
ubuntu:~$ awk '$9 == 1024 {print $1, $7}' access.log
192.168.1.10 HTTP/1.1"
192.168.1.13 HTTP/1.1"

# ------------------- Error Report -------------------

ubuntu:~$ grep -E "401|403" access.log | awk '{print $1, $7, $9}'
192.168.1.11 HTTP/1.1" 512
192.168.1.10 HTTP/1.1" 256

# ------------------- Top IP by Request Count -------------------

ubuntu:~$ awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -1
      2 192.168.1.10


ubuntu:~$ awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -4
      2 192.168.1.10
      1 192.168.1.13
      1 192.168.1.12
      1 192.168.1.11

# ------------------- Clean Log Report (Final Output) -------------------

ubuntu:~$ grep "GET" access.log \
> | sed 's/\[.*\]//g' \
> | awk '{print "IP:", $1, "| URL:", $7, "| Status:", $9}'
IP: 192.168.1.10 | URL: 200 | Status: 
IP: 192.168.1.12 | URL: 200 | Status: 
IP: 192.168.1.10 | URL: 403 | Status: 
IP: 192.168.1.13 | URL: 200 | Status: 
ubuntu:~$ 
```

# Category 10: Error Handling & Debugging

## Q34 — How to handle errors in shell scripts?
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

## Q35 — How to debug shell scripts?
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

## Q36 — How to create log files from scripts?
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

# Category 11: Performance Monitoring

## Q37 — How to monitor system performance?
**Answer**
- CPU: `top`, `htop`, `mpstat`  
- Memory: `free -h`, `vmstat`  
- Disk I/O: `iostat`, `iotop`  
- Network: `iftop`, `nethogs`  
- Overall: `sar`, `dstat`


## Q38 — How to find and resolve high CPU usage?
**Answer**
```bash
ps aux --sort=-%cpu | head -10
top -p <PID>
cat /proc/<PID>/status
pidstat 1 5
perf top   # advanced profiling
```
Investigate runaway processes, optimize code, or limit resources (cgroups).


## Q39 — How to troubleshoot memory issues?
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


# Category 12: Automation & Cron Jobs

## Q40 — How to schedule tasks with cron?
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


## Q41 — How to create systemd services?
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


# Category 13: Security & Hardening

## Q42 — How to secure a Linux server?
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


## Q43 — How to audit file permissions and security?
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

# Category 14: Real-world Scripting Scenarios

## Q44 — How to create a backup script?
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

## Q45 — How to create a log analysis script?
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

## Q46 — How to create a system monitoring script?
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

## Q47 — How to create a user management script?
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

## Q48 — How to create a Docker container management script?
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

## Q49 — How to create an SSL certificate monitoring script?
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

## Q50 — How to create a database maintenance script?
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

