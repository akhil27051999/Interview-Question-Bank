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
