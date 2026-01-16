## System Information Script

**Purpose:**
  - Generate a comprehensive system information report on a Linux machine.
  - Collect hardware, OS, memory, disk, network, and process details in a readable format.
  - Useful for system audits, troubleshooting, and monitoring.

**Use Cases:**
  - `System auditing`: Quick overview of hardware, software, and network.
  - `Troubleshooting`: Identify high memory/cpu usage processes.
  - `Server monitoring`: Check uptime, logged-in users, and running services.
  - `Documentation`: Generate automated reports for multiple servers.
    
```sh
ubuntu:~$ vi system-info.sh
ubuntu:~$ cat system-info.sh

#!/bin/bash

echo "=============================="
echo "      SYSTEM INFORMATION"
echo "=============================="

echo ""
echo "Hostname        : $(hostname)"
echo "OS              : $(grep '^PRETTY_NAME' /etc/os-release | cut -d= -f2 | tr -d '"')"
echo "Kernel Version  : $(uname -r)"
echo "Architecture    : $(uname -m)"

echo ""
echo "Uptime:"
uptime -p

echo ""
echo "CPU Information:"
lscpu | grep -E 'Model name|CPU\(s\)|Architecture'

echo ""
echo "Memory Usage:"
free -h

echo ""
echo "Disk Usage:"
df -h --total | grep -E 'Filesystem|total'  # only gives the above two lines from df -h command

echo ""
echo "Top 5 Memory Consuming Processes:"
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head -6

echo ""
echo "Network Information:"
ip -br addr show                           # -br : breif output

echo ""
echo "DNS Servers:"
grep nameserver /etc/resolv.conf

echo ""
echo "Logged-in Users:"
who

echo ""
echo "=============================="
echo "      END OF REPORT"
echo "=============================="

ubuntu:~$ chmod +x system-info.sh 
ubuntu:~$ ./system-info.sh

==============================
      SYSTEM INFORMATION
==============================

Hostname        : ubuntu
OS              : Ubuntu 24.04.3 LTS
Kernel Version  : 6.8.0-90-generic
Architecture    : x86_64

Uptime:
up 11 minutes

CPU Information:
Architecture:                         x86_64
CPU(s):                               1
On-line CPU(s) list:                  0
Model name:                           Intel Xeon E312xx (Sandy Bridge, IBRS update)
BIOS Model name:                      RHEL-9.6.0 PC (Q35 + ICH9, 2009)  CPU @ 2.0GHz
NUMA node0 CPU(s):                    0

Memory Usage:
               total        used        free      shared  buff/cache   available
Mem:           1.9Gi       461Mi       562Mi       1.0Mi       1.0Gi       1.4Gi
Swap:          1.0Gi          0B       1.0Gi

Disk Usage:
Filesystem      Size  Used Avail Use% Mounted on
total            21G  5.3G   16G  26% -

Top 5 Memory Consuming Processes:
    PID    PPID CMD                         %MEM %CPU
    946       1 /usr/bin/dockerd -H fd:// -  3.6  0.0
   1319       1 /opt/theia/node /opt/theia/  3.5  0.1
   1719    1319 /opt/theia/node /opt/theia/  2.7  0.8
    655       1 /usr/bin/containerd          2.4  0.0
    347       1 /sbin/multipathd -d -s       1.3  0.0

Network Information:
lo               UNKNOWN        127.0.0.1/8 ::1/128 
enp1s0           UP             172.30.1.2/24 fe80::b2e6:2d6d:2c58:33d2/64 
docker0          DOWN           172.17.0.1/16 

DNS Servers:
nameserver 8.8.8.8
nameserver 1.1.1.1

Logged-in Users:

==============================
      END OF REPORT
==============================
ubuntu:~$
```
