## Ping List of Webservers from a file

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
