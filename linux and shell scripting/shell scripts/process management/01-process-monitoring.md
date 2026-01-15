## Process Monitoring Script 

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
