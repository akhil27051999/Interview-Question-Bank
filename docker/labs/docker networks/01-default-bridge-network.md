## Docker Bridge Network

### Step 1: Run two containers

```sh
bash-5.1$ docker run -d --name web1 nginx
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
2dd1fb169f27643b6a357a807ab19ee77466e1588994c6806c0c6515c82b3ed0

bash-5.1$ docker run -d --name web2 nginx
d70dc3b59c8981bb51fd4a3c0e50fd7d63fb1ced93224b890ff36fe1cd3f3624
```
### Step 2: Inspect default bridge

```sh
bash-5.1$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "118f65fe21ff0bf690ef05b5632e82f58343d4ff9f30f4f202737e223d6c3f74",
        "Created": "2026-01-07T15:00:05.102762156Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "2dd1fb169f27643b6a357a807ab19ee77466e1588994c6806c0c6515c82b3ed0": {
                "Name": "web1",
                "EndpointID": "e6658b3ae687f27b569b0cb4727fcaf494f58eb34af976550264d93692d253ea",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "d70dc3b59c8981bb51fd4a3c0e50fd7d63fb1ced93224b890ff36fe1cd3f3624": {
                "Name": "web2",
                "EndpointID": "5894f1fc28a80610c8c5308bd1cb8afd004c7bfceacd175d04e65188403357ae",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

### Step 3: Test DNS (will fail)
```sh
bash-5.1$ docker exec -it web1 bash -c "apt update && apt install -y iputils-ping"
Get:1 http://deb.debian.org/debian trixie InRelease [140 kB]
Get:2 http://deb.debian.org/debian trixie-updates InRelease [47.3 kB]
Get:3 http://deb.debian.org/debian-security trixie-security InRelease [43.4 kB]
Get:4 http://deb.debian.org/debian trixie/main amd64 Packages [9670 kB]
Get:5 http://deb.debian.org/debian trixie-updates/main amd64 Packages [5412 B]
Get:6 http://deb.debian.org/debian-security trixie-security/main amd64 Packages [94.2 kB]
Fetched 10.0 MB in 1s (14.0 MB/s)                          
All packages are up to date.    
Installing:                     
  iputils-ping

Installing dependencies:
  linux-sysctl-defaults

Summary:
  Upgrading: 0, Installing: 2, Removing: 0, Not Upgrading: 0
  Download size: 56.9 kB
  Space needed: 211 kB / 24.9 GB available

Get:1 http://deb.debian.org/debian trixie/main amd64 iputils-ping amd64 3:20240905-3 [51.2 kB]
Get:2 http://deb.debian.org/debian trixie/main amd64 linux-sysctl-defaults all 4.12 [5624 B]
Fetched 56.9 kB in 0s (3632 kB/s)              
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

bash-5.1$ docker exec web1 ping -c 4 web2
ping: web2: Name or service not known

bash-5.1$ 
```
#### NOTE: 
  - Default bridge has no automatic DNS ❌
  - Containers must use IPs → bad practice
