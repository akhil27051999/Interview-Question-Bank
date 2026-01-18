## Network Service Troubleshooting

**You will troubleshoot a simulated web service failure by:**
  1. Identifying which process is running on a given port.
  2. Checking the network state using ss .
  3. Testing basic connectivity using nc (Netcat).

### Step 1: Start a Dummy Service
**We will simulate a simple web service that has been configured to run on port 8081. This command will start a Python HTTP server in the background (& ) for us to troubleshoot.**
```sh
python3 -m http.server 8081 &
```

```sh
ubuntu:~$ python3 -m http.server 8081 &
[1] 1787

ubuntu:~$ Serving HTTP on 0.0.0.0 port 8081 (http://0.0.0.0:8081/) ...
^C
```
> The output [1] followed by a process ID (PID) confirms the command has been sent to the background. The server is now running, but we need to confirm it is actually listening on the network.

### Step 2: Check whether the process is running or not

```sh
ubuntu:~$ ps aux | grep python
root         707  0.0  1.1 110016 22912 ?        Ssl  15:14   0:00 /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal
root        1787  0.0  1.0 103304 19968 pts/0    S    15:20   0:00 python3 -m http.server 8081
root        1848  0.0  0.0   3528  1792 pts/0    S+   15:23   0:00 grep --color=auto python
ubuntu:~$ ps aux | grep 8081  
root        1787  0.0  1.0 103304 19968 pts/0    S    15:20   0:00 python3 -m http.server 8081
root        1850  0.0  0.0   3528  1792 pts/0    S+   15:23   0:00 grep --color=auto 8081
```

### Step 3: Identify Listening Ports with ss

**We will use the following flags:**
  - `-t` : Show TCP sockets.
  - `-l` : Show listening sockets.
  - `-n` : Show numerical addresses (don't try to resolve names).
  - `-p` : Show the process using the socket.
    
**Run the command in another tab and use grep to quickly find our target port, 8081.**
```sh
ss -tlpn | grep 8081
```
> You should see an entry showing the python3 process listening on port 8081. This confirms the application is running and has successfully claimed the port.

```sh
ubuntu:~$ ss -tulnp
Netid   State    Recv-Q    Send-Q                            Local Address:Port        Peer Address:Port   Process                                                  
udp     UNCONN   0         0                                    127.0.0.54:53               0.0.0.0:*       users:(("systemd-resolve",pid=1201,fd=16))              
udp     UNCONN   0         0                                 127.0.0.53%lo:53               0.0.0.0:*       users:(("systemd-resolve",pid=1201,fd=14))              
udp     UNCONN   0         0                                    172.30.1.2:68               0.0.0.0:*       users:(("dhcpcd",pid=1148,fd=7))                        
udp     UNCONN   0         0                             172.30.1.2%enp1s0:68               0.0.0.0:*       users:(("systemd-network",pid=452,fd=21))               
udp     UNCONN   0         0            [fe80::14ab:b95b:b5b7:e3e9]%enp1s0:546                 [::]:*       users:(("dhcpcd",pid=875,fd=7))                         
tcp     LISTEN   0         4096                              127.0.0.53%lo:53               0.0.0.0:*       users:(("systemd-resolve",pid=1201,fd=15))              
tcp     LISTEN   0         4096                                    0.0.0.0:22               0.0.0.0:*       users:(("sshd",pid=687,fd=3),("systemd",pid=1,fd=93))   
tcp     LISTEN   0         4096                                 127.0.0.54:53               0.0.0.0:*       users:(("systemd-resolve",pid=1201,fd=17))              
tcp     LISTEN   0         128                                     0.0.0.0:40200            0.0.0.0:*       users:(("kc-terminal",pid=1281,fd=12))                  
tcp     LISTEN   0         511                                     0.0.0.0:40205            0.0.0.0:*       users:(("node",pid=1232,fd=18))                         
tcp     LISTEN   0         5                                       0.0.0.0:8081             0.0.0.0:*       users:(("python3",pid=1787,fd=3))                       
tcp     LISTEN   0         4096                                  127.0.0.1:35497            0.0.0.0:*       users:(("containerd",pid=644,fd=9))                     
tcp     LISTEN   0         4096                                       [::]:22                  [::]:*       users:(("sshd",pid=687,fd=4),("systemd",pid=1,fd=94))   
tcp     LISTEN   0         4096                                          *:40300                  *:*       users:(("runtime-scenari",pid=1222,fd=3))               
tcp     LISTEN   0         4096                                          *:40305                  *:*       users:(("runtime-info-se",pid=1269,fd=3))

ubuntu:~$ netstat -tulnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      1201/systemd-resolv 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1/init              
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      1201/systemd-resolv 
tcp        0      0 0.0.0.0:40200           0.0.0.0:*               LISTEN      1281/kc-terminal    
tcp        0      0 0.0.0.0:40205           0.0.0.0:*               LISTEN      1232/node           
tcp        0      0 0.0.0.0:8081            0.0.0.0:*               LISTEN      1787/python3        
tcp        0      0 127.0.0.1:35497         0.0.0.0:*               LISTEN      644/containerd      
tcp6       0      0 :::22                   :::*                    LISTEN      1/init              
tcp6       0      0 :::40300                :::*                    LISTEN      1222/runtime-scenar 
tcp6       0      0 :::40305                :::*                    LISTEN      1269/runtime-info-s 
udp        0      0 127.0.0.54:53           0.0.0.0:*                           1201/systemd-resolv 
udp        0      0 127.0.0.53:53           0.0.0.0:*                           1201/systemd-resolv 
udp        0      0 172.30.1.2:68           0.0.0.0:*                           1148/dhcpcd: [BOOTP 
udp        0      0 172.30.1.2:68           0.0.0.0:*                           452/systemd-network 
udp6       0      0 fe80::14ab:b95b:b5b:546 :::*                                875/dhcpcd: [DHCP6  
```

### Step 4: Verify Process ID (PID) with lsof

**While ss is great for an overview, lsof (list open files) is a powerful tool to explicitly find which process has opened a specific network file (a socket).**

**Get the exact PID for the process on port 8081.**

```sh
ubuntu:~$ lsof -i :8081
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
python3 1787 root    3u  IPv4  12139      0t0  TCP *:tproxy (LISTEN)
```
> Remember the PID you see. You would use this PID if you needed to stop the process using kill <PID>.

### Step 5: Test TCP Connectivity with nc
**Now that we know the service is listening, we need to test if it's reachable. nc (Netcat) is often called the "Swiss Army Knife" of networking and can be used to quickly verify that a connection can be established.**

**Verify that the local host can successfully connect to port 8081.**
**We use the following flags for a connection test:**
  - `-v` : Verbose output.
  - `-z` : Zero-I/O mode (just scan for a listener, don't send data).
```sh
nc -vz 127.0.0.1 8081
```

```sh
ubuntu:~$ nc -zv 127.0.0.1 8081
Connection to 127.0.0.1 8081 port [tcp/tproxy] succeeded!
ubuntu:~$
```

> If the output confirms a successful connection (e.g., "Connection to 127.0.0.1 8081 port [tcp/*] succeeded!"), you have successfully confirmed the service's network availability.
