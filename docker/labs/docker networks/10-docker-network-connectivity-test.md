## Docker Network Connectivity Test

### Step 1: Create Docker network
```sh
bash-5.1$ docker network create my_network

c7f2219d0f0818f3a885b7cc0c5c6cfc5d9fd108fe35550f77aade46797f5a5e
```
### Step 2: Create Docker Containers
```sh
bash-5.1$ docker run -d --name container1 --network my_network nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
02d7611c4eae: Pull complete 
dcea87ab9c4a: Pull complete 
35df28ad1026: Pull complete 
99ae2d6d05ef: Pull complete 
a2b008488679: Pull complete 
d03ca78f31fe: Pull complete 
d6799cf0ce70: Pull complete 
Digest: sha256:ca871a86d45a3ec6864dc45f014b11fe626145569ef0e74deaffc95a3b15b430
Status: Downloaded newer image for nginx:latest
faec006537d8e52788b913e2a3958bc52635a1b7168e8aafe60f3a0b11699fdb

bash-5.1$ docker run -d --name container2 --network my_network nginx:1.23.4

Unable to find image 'nginx:1.23.4' locally
1.23.4: Pulling from library/nginx
f03b40093957: Pull complete 
0972072e0e8a: Pull complete 
a85095acb896: Pull complete 
d24b987aa74e: Pull complete 
6c1a86118ade: Pull complete 
9989f7b33228: Pull complete 
Digest: sha256:f5747a42e3adcb3168049d63278d7251d91185bb5111d2563d58729a5c9179b0
Status: Downloaded newer image for nginx:1.23.4
ecfebcf314fc99b3986ee09df1115615d084fad31684757ba703822ddc40ad06

bash-5.1$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
ecfebcf314fc   nginx:1.23.4   "/docker-entrypoint.…"   9 seconds ago    Up 8 seconds    80/tcp    container2
faec006537d8   nginx          "/docker-entrypoint.…"   46 seconds ago   Up 45 seconds   80/tcp    container1
```

### Step 3: Test the network connectivity between two Docker containers by pinging from one container to another.
```sh
bash-5.1$ docker exec -it container1 bash -c "apt update && apt install -y iputils-ping"
Get:1 http://deb.debian.org/debian trixie InRelease [140 kB]
Get:2 http://deb.debian.org/debian trixie-updates InRelease [47.3 kB]
Get:3 http://deb.debian.org/debian-security trixie-security InRelease [43.4 kB]
Get:4 http://deb.debian.org/debian trixie/main amd64 Packages [9670 kB]
Get:5 http://deb.debian.org/debian trixie-updates/main amd64 Packages [5412 B]
Get:6 http://deb.debian.org/debian-security trixie-security/main amd64 Packages [94.0 kB]
Fetched 10.0 MB in 1s (14.7 MB/s)               
All packages are up to date.    
Installing:                     
  iputils-ping

Installing dependencies:
  linux-sysctl-defaults

Summary:
  Upgrading: 0, Installing: 2, Removing: 0, Not Upgrading: 0
  Download size: 56.9 kB
  Space needed: 211 kB / 26.8 GB available

Get:1 http://deb.debian.org/debian trixie/main amd64 iputils-ping amd64 3:20240905-3 [51.2 kB]
Get:2 http://deb.debian.org/debian trixie/main amd64 linux-sysctl-defaults all 4.12 [5624 B]
Fetched 56.9 kB in 0s (3723 kB/s)
debconf: unable to initialize frontend: Dialog
debconf: (No usable dialog-like program is installed, so the dialog based frontend cannot be used. at /usr/share/perl5/Debconf/FrontEnd/Dialog.pm line 79, <STDIN> line 2.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (Cant locate Term/ReadLine.pm in @INC (you may need to install the Term::ReadLine module) (@INC entries checked: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.40.1 /usr/local/share/perl/5.40.1 /usr/lib/x86_64-linux-gnu/perl5/5.40 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl-base /usr/lib/x86_64-linux-gnu/perl/5.40 /usr/share/perl/5.40 /usr/local/lib/site_perl) at /usr/share/perl5/Debconf/FrontEnd/Readline.pm line 8, <STDIN> line 2.)
debconf: falling back to frontend: Teletype
Selecting previously unselected package iputils-ping.
(Reading database ... 6699 files and directories currently installed.)
Preparing to unpack .../iputils-ping_3%3a20240905-3_amd64.deb ...
Unpacking iputils-ping (3:20240905-3) ...
Selecting previously unselected package linux-sysctl-defaults.
Preparing to unpack .../linux-sysctl-defaults_4.12_all.deb ...
Unpacking linux-sysctl-defaults (4.12) ...
Setting up linux-sysctl-defaults (4.12) ...
Setting up iputils-ping (3:20240905-3) ...

bash-5.1$ docker exec container1 ping -c 3 container2

PING container2 (172.18.0.3) 56(84) bytes of data.
64 bytes from container2.my_network (172.18.0.3): icmp_seq=1 ttl=64 time=0.076 ms
64 bytes from container2.my_network (172.18.0.3): icmp_seq=2 ttl=64 time=0.065 ms
64 bytes from container2.my_network (172.18.0.3): icmp_seq=3 ttl=64 time=0.066 ms

--- container2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2033ms
rtt min/avg/max/mdev = 0.065/0.069/0.076/0.005 ms
bash-5.1$ 
```
