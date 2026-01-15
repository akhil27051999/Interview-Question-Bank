## Check Open Ports of a Linux server

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
