## 1. Disk Monitoring Script

**Purpose:**
  1. check the total disk consumption of the system
  2. setup threshold and alert if disk exceeds the threshold
   
**Use:**
 - `awk` → To find patterns and perform specified actions on matching lines or data fields
 - `sed` → Use for searching, replacing, inserting, and deleting text without opening the file interactively

**Use Cases:**
  - Monitor disk usage on servers to prevent full disks and system issues.
  - Automate alerts for system administrators.
  - Can be scheduled via cron to run periodically and report disk status.

**Tips / Notes:**
  - Always use `df -h` (not `du -sh`) for filesystem usage, because du shows folder sizes, not total disk usage.
  - Ensure numeric comparison uses `[ "$usage" -ge "$threshold" ]` (remove extra than).
  - Can extend the script to email alerts when usage exceeds the threshold.
  - For multiple partitions, loop through `df -h` lines instead of just `NR==2`.
    
```sh

ubuntu:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           191M  976K  190M   1% /run
/dev/vda1        19G  5.2G   14G  29% /
tmpfs           952M   84K  952M   1% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/vda16      881M  117M  703M  15% /boot
/dev/vda15      105M  6.2M   99M   6% /boot/efi

ubuntu:~$ vi monitor-disk-usage.sh
ubuntu:~$ cat monitor-disk-usage.sh

#!/bin/bash

threshold=20

usage=$(df -h /* | awk 'NR==2 {print $5}' | sed 's/%/ /');

if [ "$usage -ge than $threshold" ]; then
  echo "disk usage is higher than $threshold : $usage"
else
  echo "disk usage is normal"
fi

ubuntu:~$ ./monitor-disk-usage.sh 
disk usage is higher than 20 : 29 
```

## 2. Process Monitoring Script 

**Purpose:**
  1. check whether a process is running or not on the system/ server
  2. check is the package of that process is installed or not
  3. Optionally, restart or check the status of the service if needed.

**Use:**
  - `pgrep` → To check whetehr process is running or not, used to look through currently running processes and list their process IDs (PIDs) that match a specified pattern or other criteria
  - `systemctl cmds` → To check status, start, stop, restart services
  - `debian package` → To check whether package is installed or not

**Use Cases:**
  - Ensure critical services like cron, nginx, mysql, nodejs are always running.
  - Automate restart of failed processes.
  - Check package installation before troubleshooting process failures.
  - Can be scheduled in cron for periodic monitoring.

**Tips / Notes:**
  - Always run the script with sudo if restarting system services.
  - Can extend to log failures or send alerts via email/Slack when processes are down.
  - Works with multiple processes at once by passing multiple arguments.

```sh
ubuntu:~$ vi process-monitor.sh
ubuntu:~$ cat process-monitor.sh 

#!/bin/bash

if [ $# -eq 0 ]; then
  echo "No process provided: $0"
  exit 1
fi

for process in "$@"; do
  if ! pgrep "$process"; then
    echo "process is not running... Restarting $process"
    sudo systemctl restart "$process"
  else
    echo "process "$process" is running"
    dpkg -l | grep -i "$process"
    sudo systemctl status "$process"
  fi
done

ubuntu:~$ ./process-monitor.sh cron
787
process cron is running
ii  cron                             3.0pl1-184ubuntu2                       amd64        process scheduling daemon
ii  cron-daemon-common               3.0pl1-184ubuntu2                       all          process scheduling daemons configuration files
● cron.service - Regular background program processing daemon
     Loaded: loaded (/usr/lib/systemd/system/cron.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-01-12 16:27:19 UTC; 9min ago
       Docs: man:cron(8)
   Main PID: 787 (cron)
      Tasks: 8 (limit: 2237)
     Memory: 6.5M (peak: 10.2M)
        CPU: 86ms
     CGroup: /system.slice/cron.service
             ├─ 787 /usr/sbin/cron -f -P
             ├─ 840 "dhcpcd: enp1s0 [ip4] [ip6]"
             ├─ 841 "dhcpcd: [privileged proxy] enp1s0 [ip4] [ip6]"
             ├─ 842 "dhcpcd: [network proxy] enp1s0 [ip4] [ip6]"
             ├─ 843 "dhcpcd: [control proxy] enp1s0 [ip4] [ip6]"
             ├─1027 "dhcpcd: [BPF ARP] enp1s0 172.30.1.2"
             ├─1126 "dhcpcd: [DHCP6 proxy] fe80::f16b:b748:45c1:d58b"
             └─1138 "dhcpcd: [BOOTP proxy] 172.30.1.2"

Jan 12 16:27:25 ubuntu dhcpcd[841]: enp1s0: adding route to 172.30.1.0/24
Jan 12 16:27:25 ubuntu dhcpcd[841]: enp1s0: adding default route via 172.30.1.1
Jan 12 16:27:25 ubuntu dhcpcd[841]: control command: dhcpcd enp1s0
Jan 12 16:27:25 ubuntu dhcpcd[841]: control command: dhcpcd enp1s0
Jan 12 16:27:25 ubuntu CRON[802]: (CRON) info (No MTA installed, discarding output)
Jan 12 16:27:25 ubuntu CRON[802]: pam_unix(cron:session): session closed for user root
Jan 12 16:27:33 ubuntu dhcpcd[841]: enp1s0: no IPv6 Routers available
Jan 12 16:35:01 ubuntu CRON[1639]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Jan 12 16:35:01 ubuntu CRON[1640]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1)
Jan 12 16:35:01 ubuntu CRON[1639]: pam_unix(cron:session): session closed for user root
ubuntu:~$ 
```

## 3. Check Open Ports of a Linux server

**Purpose:**
  1. Check whether a specific port is open (listening) or not on a Linux system/server.
  2. Identify which process is using that port if it’s open.

**Use:**
  - `netstat` → Lists network connections, listening ports, and associated processes.
  - `ss`→ Is a modern replacement for netstat.
  - `lsof` → List open files. In Linux, network sockets are treated as files, so lsof can list ports.

**Use Cases**
  - Check if services like SSH (22), HTTP (80), HTTPS (443), custom apps are running.
  - Troubleshoot network connectivity issues.
  - Identify which process is using a port that might conflict with your application.

```sh
ubuntu:~$ ss -tuln | grep 127.0.0.1
tcp   LISTEN 0      4096                           127.0.0.1:35865      0.0.0.0:*          

ubuntu:~$ ss -tuln
Netid         State          Recv-Q         Send-Q                                     Local Address:Port                  Peer Address:Port        Process        
udp           UNCONN         0              0                                             127.0.0.54:53                         0.0.0.0:*                          
udp           UNCONN         0              0                                          127.0.0.53%lo:53                         0.0.0.0:*                          
udp           UNCONN         0              0                                             172.30.1.2:68                         0.0.0.0:*                          
udp           UNCONN         0              0                                      172.30.1.2%enp1s0:68                         0.0.0.0:*                          
udp           UNCONN         0              0                      [fe80::4525:8746:b38:35b1]%enp1s0:546                           [::]:*                          
tcp           LISTEN         0              4096                                           127.0.0.1:35865                      0.0.0.0:*                          
tcp           LISTEN         0              4096                                       127.0.0.53%lo:53                         0.0.0.0:*                          
tcp           LISTEN         0              128                                              0.0.0.0:40200                      0.0.0.0:*                          
tcp           LISTEN         0              511                                              0.0.0.0:40205                      0.0.0.0:*                          
tcp           LISTEN         0              4096                                          127.0.0.54:53                         0.0.0.0:*                          
tcp           LISTEN         0              4096                                             0.0.0.0:22                         0.0.0.0:*                          
tcp           LISTEN         0              4096                                                   *:40305                            *:*                          
tcp           LISTEN         0              4096                                                   *:40300                            *:*                          
tcp           LISTEN         0              4096                                                [::]:22                            [::]:*                          


ubuntu:~$ netstat -tulnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:35865         0.0.0.0:*               LISTEN      636/containerd      
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      1218/systemd-resolv 
tcp        0      0 0.0.0.0:40200           0.0.0.0:*               LISTEN      1303/kc-terminal    
tcp        0      0 0.0.0.0:40205           0.0.0.0:*               LISTEN      1254/node           
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      1218/systemd-resolv 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1/init              
tcp6       0      0 :::40305                :::*                    LISTEN      1291/runtime-info-s 
tcp6       0      0 :::40300                :::*                    LISTEN      1244/runtime-scenar 
tcp6       0      0 :::22                   :::*                    LISTEN      1/init              
udp        0      0 127.0.0.54:53           0.0.0.0:*                           1218/systemd-resolv 
udp        0      0 127.0.0.53:53           0.0.0.0:*                           1218/systemd-resolv 
udp        0      0 172.30.1.2:68           0.0.0.0:*                           1165/dhcpcd: [BOOTP 
udp        0      0 172.30.1.2:68           0.0.0.0:*                           459/systemd-network 
udp6       0      0 fe80::4525:8746:b38:546 :::*                                1153/dhcpcd: [DHCP6 

ubuntu:~$ vi check-ports.sh
ubuntu:~$ cat check-ports.sh 
#!/bin/bash

if netstat -tulnp | grep "$1"  2>/dev/null; then
  echo "port '$1' is open"
else
  echo "port '$1' is not open"
fi

ubuntu:~$ ./check-ports.sh 8080
port '8080' is not open

ubuntu:~$ ./check-ports.sh 53  
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      1218/systemd-resolv 
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      1218/systemd-resolv 
udp        0      0 127.0.0.54:53           0.0.0.0:*                           1218/systemd-resolv 
udp        0      0 127.0.0.53:53           0.0.0.0:*                           1218/systemd-resolv 
udp6       0      0 fe80::4525:8746:b38:546 :::*                                1153/dhcpcd: [DHCP6 
port '53' is open

ubuntu:~$ ./check-ports.sh 443
port '443' is not open

ubuntu:~$ ./check-ports.sh 80 
udp6       0      0 fe80::4525:8746:b38:546 :::*                                1153/dhcpcd: [DHCP6 
port '80' is open

ubuntu:~$ vi check-ports.sh
ubuntu:~$ cat check-ports.sh 
#!/bin/bash

if lsof -i :"$1"  2>/dev/null; then
  echo "port '$1' is open"
else
  echo "port '$1' is not open"
fi

ubuntu:~$ ./check-ports.sh 53
COMMAND    PID            USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd-r 1218 systemd-resolve   14u  IPv4   8816      0t0  UDP 127.0.0.53:domain 
systemd-r 1218 systemd-resolve   15u  IPv4   8817      0t0  TCP 127.0.0.53:domain (LISTEN)
systemd-r 1218 systemd-resolve   16u  IPv4   8818      0t0  UDP 127.0.0.54:domain 
systemd-r 1218 systemd-resolve   17u  IPv4   8819      0t0  TCP 127.0.0.54:domain (LISTEN)
port '53' is open

ubuntu:~$ ./check-ports.sh 80
port '80' is not open
ubuntu:~$ ./check-ports.sh 443
port '443' is not open
ubuntu:~$ ./check-ports.sh 8080
port '8080' is not open

ubuntu:~$ ./check-ports.sh 22  
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd    1 root   93u  IPv4   5857      0t0  TCP *:ssh (LISTEN)
systemd    1 root   94u  IPv6   5861      0t0  TCP *:ssh (LISTEN)
sshd    1221 root    3u  IPv4   5857      0t0  TCP *:ssh (LISTEN)
sshd    1221 root    4u  IPv6   5861      0t0  TCP *:ssh (LISTEN)
port '22' is open

ubuntu:~$ lsof -i     
COMMAND    PID            USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd      1            root   93u  IPv4   5857      0t0  TCP *:ssh (LISTEN)
systemd      1            root   94u  IPv6   5861      0t0  TCP *:ssh (LISTEN)
systemd-n  459 systemd-network   21u  IPv4   3772      0t0  UDP 172.30.1.2:bootpc 
container  636            root    9u  IPv4   7573      0t0  TCP localhost:35865 (LISTEN)
dhcpcd    1153          dhcpcd    7u  IPv6   8481      0t0  UDP [fe80::4525:8746:b38:35b1]:dhcpv6-client 
dhcpcd    1165          dhcpcd    7u  IPv4   8542      0t0  UDP 172.30.1.2:bootpc 
systemd-r 1218 systemd-resolve   14u  IPv4   8816      0t0  UDP 127.0.0.53:domain 
systemd-r 1218 systemd-resolve   15u  IPv4   8817      0t0  TCP 127.0.0.53:domain (LISTEN)
systemd-r 1218 systemd-resolve   16u  IPv4   8818      0t0  UDP 127.0.0.54:domain 
systemd-r 1218 systemd-resolve   17u  IPv4   8819      0t0  TCP 127.0.0.54:domain (LISTEN)
sshd      1221            root    3u  IPv4   5857      0t0  TCP *:ssh (LISTEN)
sshd      1221            root    4u  IPv6   5861      0t0  TCP *:ssh (LISTEN)
runtime-s 1244            root    3u  IPv6   9079      0t0  TCP *:40300 (LISTEN)
node      1254            root   18u  IPv4   9622      0t0  TCP *:40205 (LISTEN)
node      1254            root   19u  IPv4  12256      0t0  TCP 172.30.1.2:40205->10.244.5.94:42846 (ESTABLISHED)
runtime-i 1291            root    3u  IPv6   9689      0t0  TCP *:40305 (LISTEN)
kc-termin 1303            root   12u  IPv4   9741      0t0  TCP *:40200 (LISTEN)
kc-termin 1303            root   13u  IPv4  11725      0t0  TCP 172.30.1.2:40200->10.244.4.181:39424 (ESTABLISHED)

ubuntu:~$ lsof -i :8080
ubuntu:~$ lsof -i :443
ubuntu:~$ lsof -i :53  
COMMAND    PID            USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd-r 1218 systemd-resolve   14u  IPv4   8816      0t0  UDP 127.0.0.53:domain 
systemd-r 1218 systemd-resolve   15u  IPv4   8817      0t0  TCP 127.0.0.53:domain (LISTEN)
systemd-r 1218 systemd-resolve   16u  IPv4   8818      0t0  UDP 127.0.0.54:domain 
systemd-r 1218 systemd-resolve   17u  IPv4   8819      0t0  TCP 127.0.0.54:domain (LISTEN)
ubuntu:~$ lsof -i :22  
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd    1 root   93u  IPv4   5857      0t0  TCP *:ssh (LISTEN)
systemd    1 root   94u  IPv6   5861      0t0  TCP *:ssh (LISTEN)
sshd    1221 root    3u  IPv4   5857      0t0  TCP *:ssh (LISTEN)
sshd    1221 root    4u  IPv6   5861      0t0  TCP *:ssh (LISTEN)

```

## 4. Archive and Compress Log Files Older Than 7 Days / Size of 1M (or) 1k

**Purpose:**
  1. Archive and compress log files on a Linux system to save disk space.
  2. Move the compressed logs to a dedicated archive folder for easier management.

**Use:**
  - `find` → To filter files by size or age.
  - `gzip` → To compress files.
  - `mv` → To move compressed files into a dedicated archive folder.
  - `mkdir -p` → To ensure the archive folder exists before moving files.

**Use Cases:**
  - Reduce disk space used by large log files.
  - Keep a backup/archive of logs for auditing or troubleshooting.
  - Organize logs by moving old/compressed logs to a separate folder.
  - Can be scheduled as a cron job to automate log archiving.

**Tips:**
  - Change -size +1k → -size +1M for larger logs.
  - Change -mtime +7 → -mtime +30 to archive logs older than 30 days.
  - Always check the archive folder to ensure important logs are not accidentally deleted.

```sh
ubuntu:~$ vi archive-logs.sh
ubuntu:~$ chmod +x archive-logs.sh 
ubuntu:~$ cat archive-logs.sh 
#!/bin/bash

file=/var/log/auth.log
arc=/var/log/arc-logs
mkdir -p "$arc"
find "$file" -type f -size +1k  -exec gzip {} \; -exec mv {}.gz "$arc" \;

ubuntu:~$ cd /var/log/
ubuntu:/var/log$ ls -ltrah
total 1.3M
drwxr-xr-x   2 root      root            4.0K Sep  5  2024 dist-upgrade
lrwxrwxrwx   1 root      root              39 Jan 22  2025 README -> ../../usr/share/doc/systemd/README.logs
drwxr-xr-x   2 landscape landscape       4.0K Jan 22  2025 landscape
drwxr-sr-x+  3 root      systemd-journal 4.0K Feb 10  2025 journal
drwx------   2 root      root            4.0K Feb 10  2025 private
-rw-r-----   1 root      adm              15K Feb 10  2025 dmesg.2.gz
-rw-rw----   1 root      utmp             384 Feb 10  2025 btmp.1
-rw-rw-r--   1 root      utmp             292 Feb 10  2025 lastlog
-rw-r-----   1 root      adm              15K Dec 18 17:13 dmesg.1.gz
-rw-r-----   1 root      adm              59K Dec 18 17:14 dmesg.0
drwxr-xr-x  12 root      root            4.0K Dec 18 17:15 ..
-rw-r--r--   1 root      root             15K Dec 18 17:16 alternatives.log.1
-rw-r-----   1 syslog    adm             141K Dec 18 17:16 kern.log.1
-rw-r--r--   1 root      root            250K Dec 18 17:16 dpkg.log.1
-rw-r-----   1 syslog    adm             389K Dec 18 17:17 syslog.1
-rw-r-----   1 syslog    adm              15K Dec 18 17:17 auth.log.1
drwxr-xr-x   2 root      root            4.0K Jan 14 04:33 sysstat
-rw-r--r--   1 root      root               0 Jan 14 04:33 alternatives.log
-rw-r--r--   1 root      root               0 Jan 14 04:33 dpkg.log
-rw-rw----   1 root      utmp               0 Jan 14 04:33 btmp
drwxr-xr-x   2 root      root            4.0K Jan 14 04:33 apt
-rw-rw-r--   1 root      utmp            9.4K Jan 14 04:33 wtmp
-rw-r-----   1 root      adm              56K Jan 14 04:33 dmesg
-rw-r-----   1 syslog    adm              65K Jan 14 04:33 kern.log
drwxr-xr-x   2 root      root            4.0K Jan 14 04:49 killercoda
drwxr-x---   2 root      adm             4.0K Jan 14 05:04 unattended-upgrades
drwxr-xr-x   2 root      root            4.0K Jan 14 05:04 archive
-rw-r-----   1 syslog    adm             145K Jan 14 05:22 syslog
drwxr-xr-x   2 root      root            4.0K Jan 14 05:22 arc-logs
drwxrwxr-x  12 root      syslog          4.0K Jan 14 05:22 .

ubuntu:/var/log$ cd arc-logs/
ubuntu:/var/log/arc-logs$ ls
auth.log.gz

ubuntu:~$ vi archive-logs.sh 
ubuntu:~$ cat archive-logs.sh 
#!/bin/bash

file=/var/log/kern.log
arc=/var/log/arc-logs1
mkdir -p "$arc"
find "$file" -type f -mtime 0 -exec gzip {} \; -exec mv {}.gz "$arc" \;

ubuntu:~$ ./archive-logs.sh 

ubuntu:~$ cd /var/log/
ubuntu:/var/log$ ls 
README              apt        archive     btmp.1        dmesg.0     dpkg.log    kern.log.1  lastlog  syslog.1             wtmp
alternatives.log    arc-logs   auth.log.1  dist-upgrade  dmesg.1.gz  dpkg.log.1  killercoda  private  sysstat
alternatives.log.1  arc-logs1  btmp        dmesg         dmesg.2.gz  journal     landscape   syslog   unattended-upgrades

ubuntu:/var/log$ ls -lh
total 1.2M
lrwxrwxrwx  1 root      root              39 Jan 22  2025 README -> ../../usr/share/doc/systemd/README.logs
-rw-r--r--  1 root      root               0 Jan 14 04:33 alternatives.log
-rw-r--r--  1 root      root             15K Dec 18 17:16 alternatives.log.1
drwxr-xr-x  2 root      root            4.0K Jan 14 04:33 apt
drwxr-xr-x  2 root      root            4.0K Jan 14 05:22 arc-logs
drwxr-xr-x  2 root      root            4.0K Jan 14 05:33 arc-logs1
drwxr-xr-x  2 root      root            4.0K Jan 14 05:04 archive
-rw-r-----  1 syslog    adm              15K Dec 18 17:17 auth.log.1
-rw-rw----  1 root      utmp               0 Jan 14 04:33 btmp
-rw-rw----  1 root      utmp             384 Feb 10  2025 btmp.1
drwxr-xr-x  2 root      root            4.0K Sep  5  2024 dist-upgrade
-rw-r-----  1 root      adm              56K Jan 14 04:33 dmesg
-rw-r-----  1 root      adm              59K Dec 18 17:14 dmesg.0
-rw-r-----  1 root      adm              15K Dec 18 17:13 dmesg.1.gz
-rw-r-----  1 root      adm              15K Feb 10  2025 dmesg.2.gz
-rw-r--r--  1 root      root               0 Jan 14 04:33 dpkg.log
-rw-r--r--  1 root      root            250K Dec 18 17:16 dpkg.log.1
drwxr-sr-x+ 3 root      systemd-journal 4.0K Feb 10  2025 journal
-rw-r-----  1 syslog    adm             141K Dec 18 17:16 kern.log.1
drwxr-xr-x  2 root      root            4.0K Jan 14 04:49 killercoda
drwxr-xr-x  2 landscape landscape       4.0K Jan 22  2025 landscape
-rw-rw-r--  1 root      utmp             292 Feb 10  2025 lastlog
drwx------  2 root      root            4.0K Feb 10  2025 private
-rw-r-----  1 syslog    adm             145K Jan 14 05:30 syslog
-rw-r-----  1 syslog    adm             389K Dec 18 17:17 syslog.1
drwxr-xr-x  2 root      root            4.0K Jan 14 04:33 sysstat
drwxr-x---  2 root      adm             4.0K Jan 14 05:04 unattended-upgrades
-rw-rw-r--  1 root      utmp            9.4K Jan 14 04:33 wtmp

ubuntu:/var/log$ cd arc-logs1/
ubuntu:/var/log/arc-logs1$ ls
kern.log.gz
ubuntu:/var/log/arc-logs1$ 
ubuntu:/var/log/arc-logs1$ cat kern.log.gz 
gikern.log\ksȒ<+2f?
                   npc1mD!@BMͬ/%<ɬ*
˪i-Mlixp {/                       ͨW4lU-v[
Y_֌        ~jjVǞ
[:u5(=X)<*sۮU(IzZEYugajI
                        nwD(#00NMϦ^Czp2Mg$p@@7[_UI
WެkW7\.Yvv2:zՒ{aff:VŬ;z̜Jr4f5`Ak}fx׽M#$Yq,iXϋ)2칸y:YAxW97F-,޼^ObJ

                                                               P1s޿W8\yNOX'|udz
(DkS4bxŝs}-y<x4F~,vw<7RC<z\     쟄;;b<3zRй@_r;p=T
                                                 Snh/}/hc40-l5HOq1c4f~6},1TV+)g~fhFQ:!Q~-jYւ
sulSI6?[jTbۺ7ם[iY^kjnZU                                                                     LYn:D!Lj/΄^]7)̺V
4Q򩁪5NM
      WIm-TZl9
              n
               vϭCmRؖ2\탱X^!*>KiF"l`>67SKWpd4L@;=@
;bG;&R8\}_H*>fg΄xz,fVi|mtP1) gMs
                                -2vtoZ^_
<?i

```

## 5. Count servers by role using sort and uniq

**Purpose:**
  1. Count the number of servers grouped by their role/application (like nginx, nodejs, mysql).
  2. Identify which roles are most common in your environment.

**Use:**
  - `awk` → To extract the column with the server role/application.
  - `grep -v '^$'` → Remove empty lines for accurate counting.
  - `sort` → Alphabetically sort before using uniq.
  - `uniq -c` → Count occurrences of each unique role.
  - `sort -nr` → Sort by frequency, most common first.

**Use Cases:**
  - Quickly see how many servers run each application/service.
  - Generate inventory or reporting for Ops/DevOps teams.
  - Identify over-provisioned or under-utilized services.
  - Can be extended to group by environment (prod/dev) or region by changing the awk column.

**Tips:**
  - Always sort alphabetically before uniq -c so counts are correct.
  - Use sort -nr to see most frequent roles first.
  - Can combine with awk '{print $4, $5}' to count by environment + role.
    
```sh
ubuntu:~$ cat servers.txt   
web-01     192.168.10.11   ubuntu   prod   nginx
web-02     192.168.10.12   ubuntu   prod   nginx
app-01     192.168.20.21   ubuntu   prod   nodejs
app-02     192.168.20.22   ubuntu   prod   nodejs
db-01      192.168.30.31   ubuntu   prod   mysql
cache-01   192.168.40.41   ubuntu   prod   redis

web-dev-01 10.0.10.11      ubuntu   dev    nginx
app-dev-01 10.0.20.21      ubuntu   dev    nodejs
db-dev-01  10.0.30.31      ubuntu   dev    mysql

bastion-01 192.168.1.10    ubuntu   prod   ssh
monitor-01 192.168.50.50   ubuntu   prod   prometheus

ubuntu:~$ vi count-servers.sh
ubuntu:~$ chmod +x count-servers.sh   
ubuntu:~$ cat count-servers.sh 
#!/bin/bash

awk '{print $5}' servers.txt | grep -v '^$' | sort | uniq -c

ubuntu:~$ ./count-servers.sh 
      2 mysql
      3 nginx
      3 nodejs
      1 prometheus
      1 redis
      1 ssh

ubuntu:~$ 
# Always sort alphabetically first before uniq -c
# Use sort -nr after counting to sort by frequency
# Don’t use -n on strings

ubuntu:~$ vi count-servers.sh 
ubuntu:~$ cat count-servers.sh 
#!/bin/bash

awk '{print $5}' servers.txt | grep -v '^$' | sort | uniq -c | sort -nr

ubuntu:~$ ./count-servers.sh 
      3 nodejs
      3 nginx
      2 mysql
      1 ssh
      1 redis
      1 prometheus

# grep -v '^$'→ remove empty spaces
# sort → alphabetically
# uniq -c → count occurrences
# sort -nr → sort numerically descending by count
```

## 6. Ping List of Webservers from a file

**Purpose:**
  1. Check network connectivity to a list of servers/webservers stored in a file.
  2. Verify whether the servers are reachable (pingable).

**Use:**
  - `awk` → To extract the IP address column from the file.
  - `ping` → To check network connectivity to each IP.
  - `while read` → To iterate over each IP individually, since ping cannot read multiple IPs from stdin.

**Use Cases:**
  - Verify if servers are reachable from your monitoring machine.
  - Troubleshoot network issues or identify down hosts.
  - Can be automated via cron to run periodic connectivity checks.

**Tips:**
  - Always use the user-supplied filename variable ($file) instead of hardcoding a filename.
  - Do not pipe IPs directly to ping; it only accepts one IP at a time.
  - The mistakes in the first script version:
    1. {} was incorrectly used (works only with find -exec).
    2. Ignored the filename variable and hardcoded servers.txt.
  - You can also redirect results to a log file for later review:

```sh
ubuntu:~$ touch servers.txt
ubuntu:~$ vi servers.txt 
ubuntu:~$ cat servers.txt 
web-01     192.168.10.11   ubuntu   prod   nginx
web-02     192.168.10.12   ubuntu   prod   nginx
app-01     192.168.20.21   ubuntu   prod   nodejs
app-02     192.168.20.22   ubuntu   prod   nodejs
db-01      192.168.30.31   ubuntu   prod   mysql
cache-01   192.168.40.41   ubuntu   prod   redis

web-dev-01 10.0.10.11      ubuntu   dev    nginx
app-dev-01 10.0.20.21      ubuntu   dev    nodejs
db-dev-01  10.0.30.31      ubuntu   dev    mysql

bastion-01 192.168.1.10    ubuntu   prod   ssh
monitor-01 192.168.50.50   ubuntu   prod   prometheus

ubuntu:~$ pwd              
/root

ubuntu:~$ ls -lh
total 4.0K
lrwxrwxrwx 1 root root   1 Dec 18 17:16 filesystem -> /
-rw-r--r-- 1 root root 548 Jan 14 05:48 servers.txt

ubuntu:~$ realpath servers.txt 
/root/servers.txt

ubuntu:~$ vi ping-server.sh 
ubuntu:~$ cat ping-server.sh

#!/bin/bash

read -p "please enter the file name:" file

if [ ! -f "$file" ]; then
  echo "file not exists"
  exit 1
fi

awk '{print $2}' servers.txt | while read -r ip; do
  echo "pinging $ip..."
  ping -c 4 $ip
  echo
done
 
ubuntu:~$ ./ping-server.sh 
please enter the file name:servers.txt
pinging 192.168.10.11...
PING 192.168.10.11 (192.168.10.11) 56(84) bytes of data.

--- 192.168.10.11 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3105ms


pinging 192.168.10.12...
PING 192.168.10.12 (192.168.10.12) 56(84) bytes of data.

--- 192.168.10.12 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3054ms


pinging 192.168.20.21...
PING 192.168.20.21 (192.168.20.21) 56(84) bytes of data.

--- 192.168.20.21 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3054ms


pinging 192.168.20.22...
PING 192.168.20.22 (192.168.20.22) 56(84) bytes of data.

--- 192.168.20.22 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3054ms


pinging 192.168.30.31...
PING 192.168.30.31 (192.168.30.31) 56(84) bytes of data.

--- 192.168.30.31 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3054ms


pinging 192.168.40.41...
PING 192.168.40.41 (192.168.40.41) 56(84) bytes of data.

--- 192.168.40.41 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3053ms


pinging ...
ping: usage error: Destination address required

pinging 10.0.10.11...
PING 10.0.10.11 (10.0.10.11) 56(84) bytes of data.

--- 10.0.10.11 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3052ms


pinging 10.0.20.21...
PING 10.0.20.21 (10.0.20.21) 56(84) bytes of data.

--- 10.0.20.21 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3054ms


pinging 10.0.30.31...
PING 10.0.30.31 (10.0.30.31) 56(84) bytes of data.

--- 10.0.30.31 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3054ms


pinging ...
ping: usage error: Destination address required

pinging 192.168.1.10...
PING 192.168.1.10 (192.168.1.10) 56(84) bytes of data.

--- 192.168.1.10 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3051ms


pinging 192.168.50.50...
PING 192.168.50.50 (192.168.50.50) 56(84) bytes of data.

--- 192.168.50.50 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3054ms


pinging ...
ping: usage error: Destination address required

ubuntu:~$ 
```

## 7. Backup, Restore & Automate /var/log

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

## 8. Validate USER input 

**Purpose:**
  1. Accept user input at runtime using the terminal.
  2. Validate whether the entered value is positive, negative, or zero.
  3. Demonstrate conditional logic (if / elif / else) in shell scripting.

**use:**
  - `read -p` → Prompts the user and stores input in a variable (num).
  - `-gt` → Checks if the number is greater than zero.
  - `-lt` → Checks if the number is less than zero.
  - `if / elif / else` → Executes logic based on the condition outcome.
  - `echo` → Displays the result to the user.

**Use Cases:**
  - Validating user-provided values in shell scripts.
  - Simple input checks before running critical operations.
  - Used in menu-driven scripts or automation tools.
  - Basic validation for DevOps utility scripts (ports, thresholds, IDs).
    
```sh
ubuntu:~$ vi user-script.sh
ubuntu:~$ chmod +x user-script.sh 
ubuntu:~$ cat user-script.sh 

#!/bin/bash

read -p "enter the value:" num

if [ "$num" -gt 0 ]; then 
  echo "Its a positive number"
elif [ "$num" -lt 0 ]; then
  echo "Its a negitive number"
else
  echo "Its a zero"
fi

ubuntu:~$ ./user-script.sh 
enter the value:20
Its a positive number
ubuntu:~$ ./user-script.sh 
enter the value:-19
Its a negitive number
ubuntu:~$ ./user-script.sh 
enter the value:0
Its a zero
```

## 9. Extract & Group IPs by Role

**Purpose:**
  - Extract IP addresses from a structured inventory file (servers.txt)
  - Group and display IPs by role (nginx, nodejs, mysql, etc.)
  - Provide a clear, readable inventory view for operations and automation

**Use:**
  - `$1` → Accepts the server inventory file as input
  - `awk '{print $5}'` → Extracts roles
  - `sort -u` → Removes duplicate roles

**Use Cases:**
  - Service-wise server inventory
  - Preparing inputs for:
    - Ansible
    - SSH automation
    - Monitoring configuration
  - Quick infra audits
  - DevOps interview scripting tasks
  - Filtering production vs dev servers (can be extended)

**Tips:**
  - Blank lines are safely ignored because $5 won’t match
  - Sorting roles improves readability
  - Can be extended to:
    - Filter by environment (prod/dev)
    - Export to CSV / JSON
    - Accept role as a command-line argument

```sh

ubuntu:~$ vi servers.txt
ubuntu:~$ cat servers.txt   
web-01     192.168.10.11   ubuntu   prod   nginx
web-02     192.168.10.12   ubuntu   prod   nginx
app-01     192.168.20.21   ubuntu   prod   nodejs
app-02     192.168.20.22   ubuntu   prod   nodejs
db-01      192.168.30.31   ubuntu   prod   mysql
cache-01   192.168.40.41   ubuntu   prod   redis

web-dev-01 10.0.10.11      ubuntu   dev    nginx
app-dev-01 10.0.20.21      ubuntu   dev    nodejs
db-dev-01  10.0.30.31      ubuntu   dev    mysql

bastion-01 192.168.1.10    ubuntu   prod   ssh
monitor-01 192.168.50.50   ubuntu   prod   prometheus

ubuntu:~$ vi extracts-ips.sh 
ubuntu:~$ cat extracts-ips.sh

#!/bin/bash

if [ ! -f "$1" ]; then
  echo "file is not found"
  exit 1
fi

echo "Extracting IP from $1"
echo "------------------------------"

awk '{print $2}' "$1"

ubuntu:~$ ./extracts-ips.sh server.txt
file is not found
ubuntu:~$ ./extracts-ips.sh servers.txt

Extracting IP from servers.txt
------------------------------
cat
192.168.10.11
192.168.10.12
192.168.20.21
192.168.20.22
192.168.30.31
192.168.40.41

10.0.10.11
10.0.20.21
10.0.30.31

192.168.1.10
192.168.50.50

ubuntu:~$ vi extracts-ips.sh
ubuntu:~$ cat extracts-ips.sh 

#!/bin/bash

if [ ! -f "$1" ]; then
  echo "file is not found"
  exit 1
fi

echo "Extracting IP from $1"
echo "------------------------------"

roles=$(awk '{print $5}' "$1" | sort -u)

for role in $roles; do
  echo "Role: $role"
  awk -v r="$role" '$5 == r {print "  " $2}' "$1"
done

ubuntu:~$ ./extracts-ips.sh servers.txt
Extracting IP from servers.txt
------------------------------
Role: mysql
  192.168.30.31
  10.0.30.31
Role: nginx
  192.168.10.11
  192.168.10.12
  10.0.10.11
Role: nodejs
  192.168.20.21
  192.168.20.22
  10.0.20.21
Role: prometheus
  192.168.50.50
Role: redis
  192.168.40.41
Role: ssh
  192.168.1.10

```
## Top Five Memory consuming processes

```sh
ubuntu:~$ vi process-monitor.sh
ubuntu:~$ cat process-monitor.sh

#!/bin/bash

echo "Top 5 memory consuming processes"
echo "--------------------------------"

ps -aux --sort=-%mem | head -n 6
ubuntu:~$ chmod +x process-monitor.sh 
ubuntu:~$ ./process-monitor.sh

Top 5 memory consuming processes
--------------------------------
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         928  0.0  3.6 1824352 70388 ?       Ssl  16:47   0:00 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
root        1316  0.1  3.5 848960 68516 ?        SNl  16:47   0:01 /opt/theia/node /opt/theia/browser-app/src-gen/backend/main.js /root --hostname=0.0.0.0 --port 40205
root        1727  0.8  2.6 829224 52404 ?        SNl  16:54   0:01 /opt/theia/node /opt/theia/node_modules/@theia/core/lib/node/messaging/ipc-bootstrap --nsfwOptions={}
root         630  0.0  2.4 1654564 47612 ?       Ssl  16:47   0:00 /usr/bin/containerd
root         330  0.0  1.3 288952 27136 ?        SLsl 16:47   0:00 /sbin/multipathd -d -s
```
