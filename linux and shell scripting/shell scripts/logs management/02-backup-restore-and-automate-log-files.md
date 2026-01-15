## Backup, Restore & Automate /var/log

**Purpose:**
  1. Create a compressed backup of the system log directory (/var/log).
  2. Store the backup safely in a separate location (/backup).
  3. Restore the backup to a temporary directory for verification or recovery.
  4. Automate periodic backups using cron to ensure logs are preserved.

**Use:**
  - `tar` → For archiving and compressing log files efficiently.
  - `date` → Enables daily versioned backups.
  - `mkdir -p` → Makes the script idempotent (safe to rerun).
  - `/tmp/restore` → Used to verify backup integrity before real recovery.

**Use Cases:**
  - System administration log backups before upgrades or maintenance.
  - Incident response and forensic analysis of historical logs.
  - Disk cleanup workflows (backup → delete old logs → restore if needed).
  - Automation via cron jobs for daily or weekly backups.
  - Migration of logs to another system for auditing or compliance.

**Tips:**
  - Never restore directly to / unless absolutely required; always validate in /tmp first.
  - Exclude unnecessary files (like old backups inside /var/log) to avoid recursive growth:
    ```sh
    tar --exclude='*.gz' --exclude='backup-logs*' -czf "$backup_file" "$source_dir"
    ```
  - Add logging for automation:
    ```sh
    echo "$(date): Backup completed" >> /var/log/log-backup.log
    ```
  - Use tar -tzf to list contents without extracting:
    ```sh
    tar -tzf backup-logs-2026-01-14.tar.gz
    ```
  - For production systems, consider:
    - Rotating backups
    - Offloading to S3 / NFS / remote server
    - Encrypting archives (gpg or openssl)
  - Cron Automation:
    ```sh
    */5 * * * * /root/backup.sh >> /backup/backup.log 2>&1
    ```
    - Runs the backup script every 5 minutes (adjust timing as needed).
    - Redirects standard output and errors to a log file (backup.log) for monitoring.
  - Log Rotation:
    - Monitor /backup size and remove old backups periodically to avoid filling up disk space.
  - Permissions:
    - Ensure the backup destination is writable by the user running the cron job (usually root).
      
```sh
ubuntu:~$ cd /var/log && ls -lh
total 1.4M
lrwxrwxrwx  1 root      root              39 Jan 22  2025 README -> ../../usr/share/doc/systemd/README.logs
-rw-r--r--  1 root      root               0 Jan 14 09:46 alternatives.log
-rw-r--r--  1 root      root             15K Dec 18 17:16 alternatives.log.1
-rw-r-----  1 root      adm                0 Feb 10  2025 apport.log
drwxr-xr-x  2 root      root            4.0K Jan 14 09:46 apt
-rw-r-----  1 syslog    adm             2.6K Jan 14 10:09 auth.log
-rw-r-----  1 syslog    adm              15K Dec 18 17:17 auth.log.1
-rw-rw----  1 root      utmp               0 Jan 14 09:46 btmp
-rw-rw----  1 root      utmp             384 Feb 10  2025 btmp.1
-rw-r-----  1 root      adm             5.5K Feb 10  2025 cloud-init-output.log
-rw-r-----  1 syslog    adm              92K Feb 10  2025 cloud-init.log
drwxr-xr-x  2 root      root            4.0K Sep  5  2024 dist-upgrade
-rw-r-----  1 root      adm              59K Jan 14 09:46 dmesg
-rw-r-----  1 root      adm              59K Dec 18 17:14 dmesg.0
-rw-r-----  1 root      adm              15K Dec 18 17:13 dmesg.1.gz
-rw-r-----  1 root      adm              15K Feb 10  2025 dmesg.2.gz
-rw-r--r--  1 root      root               0 Jan 14 09:46 dpkg.log
-rw-r--r--  1 root      root            250K Dec 18 17:16 dpkg.log.1
drwxr-sr-x+ 3 root      systemd-journal 4.0K Feb 10  2025 journal
-rw-r-----  1 syslog    adm              65K Jan 14 09:46 kern.log
-rw-r-----  1 syslog    adm             141K Dec 18 17:16 kern.log.1
drwxr-xr-x  2 root      root            4.0K Jan 14 09:52 killercoda
drwxr-xr-x  2 landscape landscape       4.0K Jan 22  2025 landscape
-rw-rw-r--  1 root      utmp             292 Feb 10  2025 lastlog
drwx------  2 root      root            4.0K Feb 10  2025 private
-rw-r-----  1 syslog    adm             142K Jan 14 10:09 syslog
-rw-r-----  1 syslog    adm             389K Dec 18 17:17 syslog.1
drwxr-xr-x  2 root      root            4.0K Jan 14 09:46 sysstat
drwxr-x---  2 root      adm             4.0K Feb 10  2025 unattended-upgrades
-rw-rw-r--  1 root      utmp            9.4K Jan 14 09:46 wtmp


ubuntu:/var/log$ cd
ubuntu:~$ vi backup.sh 
ubuntu:~$ cat backup.sh 

#!/bin/bash

source_dir=/var/log
dest_dir=/backup

mkdir -p "$dest_dir"
date=$(date +%F)

backup_file="$dest_dir/backup-logs-$date.tar.gz"

tar -czf "$backup_file" "$source_dir"

echo "Backup successful for $source_dir on $date"

ubuntu:~$ ls
backup.sh  filesystem

ubuntu:~$ ./backup.sh 
tar: Removing leading `/` from member names
Backup successful for /var/log on 2026-01-14

ubuntu:~$ cd /backup/
ubuntu:/backup$ ls
backup-logs-2026-01-14.tar.gz
ubuntu:/backup$ cd

ubuntu:~$ mkdir -p /tmp/restore
ubuntu:~$ ls
backup.sh  filesystem

ubuntu:~$ tar -xzf /backup/backup-logs-2026-01-14.tar.gz -C /tmp/restore
ubuntu:~$ cd /tmp/restore/
ubuntu:/tmp/restore$ ls
var
ubuntu:/tmp/restore$ cd var/log/

ubuntu:/tmp/restore/var/log$ ls -lh
total 2.4M
lrwxrwxrwx 1 root      root              39 Jan 22  2025 README -> ../../usr/share/doc/systemd/README.logs
-rw-r--r-- 1 root      root               0 Jan 14 09:46 alternatives.log
-rw-r--r-- 1 root      root             15K Dec 18 17:16 alternatives.log.1
-rw-r----- 1 root      adm                0 Feb 10  2025 apport.log
drwxr-xr-x 2 root      root            4.0K Jan 14 09:46 apt
-rw-r----- 1 syslog    adm             3.0K Jan 14 10:17 auth.log
-rw-r----- 1 syslog    adm              15K Dec 18 17:17 auth.log.1
-rw-r--r-- 1 root      root            1.1M Jan 14 10:15 backup-logs-2026-01-14.tar.gz
-rw-rw---- 1 root      utmp               0 Jan 14 09:46 btmp
-rw-rw---- 1 root      utmp             384 Feb 10  2025 btmp.1
-rw-r----- 1 root      adm             5.5K Feb 10  2025 cloud-init-output.log
-rw-r----- 1 syslog    adm              92K Feb 10  2025 cloud-init.log
drwxr-xr-x 2 root      root            4.0K Sep  5  2024 dist-upgrade
-rw-r----- 1 root      adm              59K Jan 14 09:46 dmesg
-rw-r----- 1 root      adm              59K Dec 18 17:14 dmesg.0
-rw-r----- 1 root      adm              15K Dec 18 17:13 dmesg.1.gz
-rw-r----- 1 root      adm              15K Feb 10  2025 dmesg.2.gz
-rw-r--r-- 1 root      root               0 Jan 14 09:46 dpkg.log
-rw-r--r-- 1 root      root            250K Dec 18 17:16 dpkg.log.1
drwxr-sr-x 3 root      systemd-journal 4.0K Feb 10  2025 journal
-rw-r----- 1 syslog    adm              65K Jan 14 09:46 kern.log
-rw-r----- 1 syslog    adm             141K Dec 18 17:16 kern.log.1
drwxr-xr-x 2 root      root            4.0K Jan 14 09:52 killercoda
drwxr-xr-x 2 landscape landscape       4.0K Jan 22  2025 landscape
-rw-rw-r-- 1 root      utmp             292 Feb 10  2025 lastlog
drwx------ 2 root      root            4.0K Feb 10  2025 private
-rw-r----- 1 syslog    adm             142K Jan 14 10:17 syslog
-rw-r----- 1 syslog    adm             389K Dec 18 17:17 syslog.1
drwxr-xr-x 2 root      root            4.0K Jan 14 09:46 sysstat
drwxr-x--- 2 root      adm             4.0K Feb 10  2025 unattended-upgrades
-rw-rw-r-- 1 root      utmp            9.4K Jan 14 09:46 wtmp
ubuntu:/tmp/restore/var/log$ 
ubuntu:/tmp/restore/var/log$ cd

ubuntu:~$ cd /backup/
ubuntu:/backup$ ls
backup-logs-2026-01-14.tar.gz

ubuntu:~$ crontab -e
no crontab for root - using an empty one

Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny
  4. /bin/ed

Choose 1-4 [1]: 1
crontab: installing new crontab
ubuntu:~$ crontab -l
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
# 
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
# 
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
# 
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command
*/5 * * * * /root/backup.sh >> /backup/backup.log 2>&1
      
ubuntu:~$ cd /backup/
ubuntu:/backup$ ls
backup-logs-2026-01-14.tar.gz

ubuntu:/backup$ ls -ltrah
total 1.1M
drwxr-xr-x 23 root root 4.0K Jan 14 10:54 ..
drwxr-xr-x  2 root root 4.0K Jan 14 11:00 .
-rw-r--r--  1 root root 1.1M Jan 14 11:00 backup-logs-2026-01-14.tar.gz
-rw-r--r--  1 root root   89 Jan 14 11:00 backup.log

ubuntu:/backup$ cat backup.log 
tar: Removing leading `/` from member names
Backup successful for /var/log on 2026-01-14

ubuntu:/backup$ cat backup.log 
tar: Removing leading `/` from member names
Backup successful for /var/log on 2026-01-14
tar: Removing leading `/` from member names
Backup successful for /var/log on 2026-01-14
```
