## Real App Communication (Web + DB)

### Step 1: Run MySQL
```sh
bash-5.1$ docker run -d \
  --name mysql \
  --network app-net \
  -e MYSQL_ROOT_PASSWORD=pass \
  -e MYSQL_DATABASE=appdb \
  mysql:8

Unable to find image 'mysql:8' locally
8: Pulling from library/mysql
ad9d782f3f87: Pull complete 
3709f9999ba9: Pull complete 
88358ea2a37f: Pull complete 
98f63f165ac1: Pull complete 
100b56c3fd28: Pull complete 
23eb2baa39f3: Pull complete 
08d96bdd8a50: Pull complete 
c68ab04cc1e9: Pull complete 
bec4df3fa85f: Pull complete 
8c32caf90444: Pull complete 
Digest: sha256:90544b3775490579867a30988d48f0215fc3b88d78d8d62b2c0d96ee9226a2b7
Status: Downloaded newer image for mysql:8
d2b896606ab2df87908d6813239e9dc2b137f73ef2beca8c8cc07ee31c772b10

bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                 NAMES
d2b896606ab2   mysql:8   "docker-entrypoint.s…"   8 seconds ago    Up 7 seconds    3306/tcp, 33060/tcp   mysql
e9cef19c5748   busybox   "sleep 3600"             16 minutes ago   Up 16 minutes                         api
bc359e13bbc3   nginx     "/docker-entrypoint.…"   17 minutes ago   Up 17 minutes   80/tcp                web
d70dc3b59c89   nginx     "/docker-entrypoint.…"   26 minutes ago   Up 26 minutes   80/tcp                web2
2dd1fb169f27   nginx     "/docker-entrypoint.…"   26 minutes ago   Up 26 minutes   80/tcp                web1
```

### Step 2: Access DB from another container
```sh
bash-5.1$ docker exec -it api sh

/ # ping -c 6 mysql
PING mysql (172.18.0.4): 56 data bytes
64 bytes from 172.18.0.4: seq=0 ttl=64 time=0.147 ms
64 bytes from 172.18.0.4: seq=1 ttl=64 time=0.092 ms
64 bytes from 172.18.0.4: seq=2 ttl=64 time=0.095 ms
64 bytes from 172.18.0.4: seq=3 ttl=64 time=0.100 ms
64 bytes from 172.18.0.4: seq=4 ttl=64 time=0.091 ms
64 bytes from 172.18.0.4: seq=5 ttl=64 time=0.103 ms
--- mysql ping statistics ---
6 packets transmitted, 6 packets received, 0% packet loss
round-trip min/avg/max = 0.091/0.104/0.147 ms

/ # exit
```

#### NOTE: 
  - Containers talk via service names ✅
  - No IP hardcoding ✅
