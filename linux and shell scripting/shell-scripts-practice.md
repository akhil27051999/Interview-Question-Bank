## Disk Monitoring script

**1. check the total disk consumption of the system**

**2. setup threshold and alert if disk exceeds the threshold**
   
**use:**
 - `awk` : to find patterns and perform specified actions on matching lines or data fields
 - `sed` : use for searching, replacing, inserting, and deleting text without opening the file interactively
     
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
ubuntu:~$ chmod +x monitor-disk-usage.sh 
ubuntu:~$ cat monitor-disk-usage.sh 
#!/bin/bash

threshold=20

usage=$(du -sh /* | awk 'NR==2 {print $5}' | sed 's/%/ /');

if [ "$usage -ge than $threshold" ]; then

  echo "disk usage is higher than $threshold : $usage"

else

  echo "disk usage is normal"

fi
ubuntu:~$ ./monitor-disk-usage.sh 
du: cannot access '/proc/2252/task/2252/fd/4': No such file or directory
du: cannot access '/proc/2252/task/2252/fdinfo/4': No such file or directory
du: cannot access '/proc/2252/fd/3': No such file or directory
du: cannot access '/proc/2252/fdinfo/3': No such file or directory
disk usage is higher than 20 : 

ubuntu:~$ du -sh
76K     .

# Mistake : used du -sh command instead of df -h 

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
ubuntu:~$ 
ubuntu:~$ ./monitor-disk-usage.sh 
disk usage is higher than 20 : 29 

ubuntu:~$ 
```

## process monitoring script 

**1. check whether a process is running or not on the system/ server**

**2. check is the package of that process is installed or not**

  **use:**
  - `pgrep` : to check whetehr process is running or not, used to look through currently running processes and list their process IDs (PIDs) that match a specified pattern or other criteria
  - `systemctl cmds` : to check status, start, stop, restart services
  - `debian package` : to check whether package is installed or not 


```sh

ubuntu:~$ vi process-monitor.sh
ubuntu:~$ chmod +x process-monitor.sh 
ubuntu:~$ cat process-monitor.sh 

#!/bin/bash

if [ $# -eq 0 ]; then
  echo "No process provided: $0"
  exit 1
fi

for process in "$@"; do
  if ! pgrep "$process"; then
    echo "process is not running... Restarting $process"
    dpkg -l | grep -i "$process"
    sudo systemctl restart "$process"
  else
    echo "process "$process" is running"
  fi
done

ubuntu:~$ ./process-monitor.sh     
No process provided: ./process-monitor.sh

ubuntu:~$ ./process-monitor.sh cron
787
process cron is running
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

## Check Open Ports of a Linux server

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
ubuntu:~$ lsof -i :53  
COMMAND    PID            USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd-r 1218 systemd-resolve   14u  IPv4   8816      0t0  UDP 127.0.0.53:domain 
systemd-r 1218 systemd-resolve   15u  IPv4   8817      0t0  TCP 127.0.0.53:domain (LISTEN)
systemd-r 1218 systemd-resolve   16u  IPv4   8818      0t0  UDP 127.0.0.54:domain 
systemd-r 1218 systemd-resolve   17u  IPv4   8819      0t0  TCP 127.0.0.54:domain (LISTEN)

ubuntu:~$ lsof -i :443
ubuntu:~$ lsof -i :22  
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd    1 root   93u  IPv4   5857      0t0  TCP *:ssh (LISTEN)
systemd    1 root   94u  IPv6   5861      0t0  TCP *:ssh (LISTEN)
sshd    1221 root    3u  IPv4   5857      0t0  TCP *:ssh (LISTEN)
sshd    1221 root    4u  IPv6   5861      0t0  TCP *:ssh (LISTEN)
ubuntu:~$
```

### Archive and Compress Log Files Older Than 7 Days / Size of 1M (or) 1k

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
ubuntu:/var/log/arc-logs$ cat auth.log.gz
"\ٮ"
    j:N¯يt|
           `c}5=QaF_MԽ4,DTk8+5ؘ'rXd׭uC7M(cL
ʰQ:d,VG;r,֫0VɔiP_
                <3f!Y\YijBڹXENc -Ù{qE`̗5fiцk֒Z"Q bYVhFjI;Ó2,8+iA~ƜBU\t,攈M>csA@Wy\֛
o6&^A)r;?0(σv,-i`[T0ٟN6&xWzY(&^tU"]y8i\dyupӘȋ56kʒ|۔VU Y-1
                                                        Qj*,V۔bx<r[fe%mV\jq&$r!`njR2(""#4AR Ya2ebkm+$ڙ/to|}*y\yyZd>[Ia` j\=,s<!
$-Yҍ.S"lD_ֶ"ݶH9-2NDМ#-0̚iI⿈~p#                                                                                                  ˂0K>gn[h0+F<sS!
                             [(A%\Gd2;EvloJgt%i11wO:=J֣W) $s'n'4^qQFU.[|t0Y/;`"D*?Tzj^PU4eOiڝQ8G?(9]ͥU]wCl~Tw0MDX7j']$oX0Iܽqz
u
 "y8J~g66B*?$]O%x1E-\;/ G).'{7(%vϒy1yTQK.
                                         k_h#JT
z ss2ls3rxR!z~AY~>'?#!d4} Y 1/E(N       qo
                                          8zG2/yOtsQojϊ8%~ˡ>-}cTu*N݂l0tf뢕Erfa4x[,{\ `Kѱ<OHy4qZۃ/S~5@5E
                                                                                                      =$F<5g-'@.vxnw
(屪<`+pXr]Pkp 8b%` cZj
wy@
   2yoh-[fKiJ;Dme86TYd  -YE/ecSoQBoF"6Y`

[-jR>tC]$SėAuaSkַUj9Մ
                    PkZ+?CcUuJc53ϙ2G    \kH|6XDT{g
C
 3l?
    vg?L;pb4ظr]Tco1%    Kk%ah~n|X7FF6ǢrhjuߧMG
                                             9<H#l~餎UEdDXӂ7e0/R_e5-Ҥ \]or1E
                                            
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
