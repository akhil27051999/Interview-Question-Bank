## Disk Full: Log Management

**Purpose**
  1. Checking disk usage
  2. Finding which files are consuming space
  3. Freeing up space safely
  4. Configuring log rotation to prevent the issue in the future
     
> Imagine this is a production server where services stopped because /var/log has consumed all available disk space.

### Step 1: Check Disk Usage

  - Which filesystem is full?
  - What percentage of space is used?
  - Do you see / or /var/log close to 100%?

```sh
ubuntu:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           191M  984K  190M   1% /run
/dev/vda1        19G   16G  3.2G  83% /
tmpfs           952M   84K  952M   1% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/vda16      881M  117M  703M  15% /boot
/dev/vda15      105M  6.2M   99M   6% /boot/efi
```
### Step 2: Identify Large Log Files
```
ubuntu:~$ du -sh /var/log/* | sort -h
0       /var/log/README
0       /var/log/alternatives.log
0       /var/log/apport.log
0       /var/log/btmp
0       /var/log/dpkg.log
4.0K    /var/log/auth.log
4.0K    /var/log/btmp.1
4.0K    /var/log/dist-upgrade
4.0K    /var/log/landscape
4.0K    /var/log/lastlog
4.0K    /var/log/private
4.0K    /var/log/unattended-upgrades
8.0K    /var/log/cloud-init-output.log
12K     /var/log/killercoda
12K     /var/log/wtmp
16K     /var/log/alternatives.log.1
16K     /var/log/auth.log.1
16K     /var/log/dmesg.1.gz
16K     /var/log/dmesg.2.gz
20K     /var/log/sysstat
60K     /var/log/apt
60K     /var/log/dmesg
60K     /var/log/dmesg.0
68K     /var/log/kern.log
92K     /var/log/cloud-init.log
144K    /var/log/kern.log.1
144K    /var/log/syslog
256K    /var/log/dpkg.log.1
392K    /var/log/syslog.1
18M     /var/log/journal
11G     /var/log/biglog.log
```

### Step 3: Free Disk Space

**To free up space by truncating or archiving the big log file.**

1. Option A – truncate (clears content but keeps file):

```sh
 > /var/log/biglog.log
 ```

2. Option B – compress old logs (saves space but keeps history):
```sh
gzip /var/log/biglog.log.1
```

3. Verify space is freed:
```
df -h
```

```sh
ubuntu:~$ gzip /var/log/biglog.log.1 &
[1] 1750

ubuntu:~$ du -sh /var/log/* | sort -h
0       /var/log/README
0       /var/log/alternatives.log
0       /var/log/apport.log
0       /var/log/biglog.log
0       /var/log/btmp
0       /var/log/dpkg.log
4.0K    /var/log/auth.log
4.0K    /var/log/btmp.1
4.0K    /var/log/dist-upgrade
4.0K    /var/log/landscape
4.0K    /var/log/lastlog
4.0K    /var/log/private
4.0K    /var/log/unattended-upgrades
8.0K    /var/log/cloud-init-output.log
12K     /var/log/killercoda
12K     /var/log/wtmp
16K     /var/log/alternatives.log.1
16K     /var/log/auth.log.1
16K     /var/log/dmesg.1.gz
16K     /var/log/dmesg.2.gz
20K     /var/log/sysstat
60K     /var/log/apt
60K     /var/log/dmesg
60K     /var/log/dmesg.0
68K     /var/log/kern.log
92K     /var/log/cloud-init.log
144K    /var/log/kern.log.1
144K    /var/log/syslog
256K    /var/log/dpkg.log.1
392K    /var/log/syslog.1
9.8M    /var/log/biglog.log.1.gz
18M     /var/log/journal

ubuntu:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           191M  984K  190M   1% /run
/dev/vda1        19G  5.2G   14G  29% /
tmpfs           952M   84K  952M   1% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/vda16      881M  117M  703M  15% /boot
/dev/vda15      105M  6.2M   99M   6% /boot/efi
```
### Step 4: Configure Log Rotation

**To prevent logs from filling the disk again, configure logrotate .**

1. Check existing config:
```sh
cat /etc/logrotate.conf
```

2. Now create a new config for biglog.log:
```sh
sudo nano /etc/logrotate.d/biglog
```

3. Add:
```sh
/var/log/biglog.log {
    daily
    rotate 7
    copytruncate
    compress
    missingok
    notifempty
}
```
> This should keep logs small and automatically rotated every week.

```sh
ubuntu:~$ cat /etc/logrotate.conf
# see "man logrotate" for details

# global options do not affect preceding include directives

# rotate log files weekly
weekly

# use the adm group by default, since this is the owning group
# of /var/log/.
su root adm

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
#dateext

# uncomment this if you want your log files compressed
#compress

# packages drop log rotation information into this directory
include /etc/logrotate.d

# system-specific logs may also be configured here.

ubuntu:~$ sudo nano /etc/logrotate.d/biglog
ubuntu:~$ sudo cat /etc/logrotate.d/biglog
/var/log/biglog.log {
    daily
    rotate 7
    copytruncate
    compress
    missingok
    notifempty
}
ubuntu:~$
```
