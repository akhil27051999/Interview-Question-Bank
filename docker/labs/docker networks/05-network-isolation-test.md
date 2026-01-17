## Network Isolation Test

### Step 1: Create Isolated network

```sh
bash-5.1$ docker network create isolated-net
df26996a588f9b21b46a44ca7df3f9f99018cae2edd4fe5a9c822d44a29a94ec

bash-5.1$ docker network ls
NETWORK ID     NAME           DRIVER    SCOPE
3988f947a319   app-net        bridge    local
118f65fe21ff   bridge         bridge    local
e5203f00748b   host           host      local
df26996a588f   isolated-net   bridge    local
82714d432fcc   none           null      local
```

### Step 2: Run and Test the Container with Isolated Network

```sh
bash-5.1$ docker run -d --name hacker --network isolated-net busybox sleep 3600
3c23ad856a11f227a4b422001bec332bf1ece995dee04a1f3d4c0289fe49a3b4

bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS                                   NAMES
3c23ad856a11   busybox   "sleep 3600"             About a minute ago   Up About a minute                                           hacker
2948619f3302   nginx     "/docker-entrypoint.…"   5 minutes ago        Up 5 minutes        0.0.0.0:8080->80/tcp, :::8080->80/tcp   web-pub
d2b896606ab2   mysql:8   "docker-entrypoint.s…"   10 minutes ago       Up 10 minutes       3306/tcp, 33060/tcp                     mysql
e9cef19c5748   busybox   "sleep 3600"             26 minutes ago       Up 26 minutes                                               api
bc359e13bbc3   nginx     "/docker-entrypoint.…"   27 minutes ago       Up 27 minutes       80/tcp                                  web
d70dc3b59c89   nginx     "/docker-entrypoint.…"   36 minutes ago       Up 36 minutes       80/tcp                                  web2
2dd1fb169f27   nginx     "/docker-entrypoint.…"   36 minutes ago       Up 36 minutes       80/tcp                                  web1

bash-5.1$ docker exec hacker ping -c 4 web
ping: bad address 'web'
```

#### ✅ This tells us two things
  1. hacker is on isolated-net
  2. web is on app-net
  3. Docker DNS only works within the same network

#### ❌ Containers on different Docker networks cannot: 
  - Resolve each other’s names
  - Reach each other’s IPs
  - Communicate at all
  - This is intentional network isolation, not an error.
