## User Defined Bridge Network

### Step 1: Create custom bridge

```sh
bash-5.1$ docker network create app-net
3988f947a319bc6830e461fe4485570d2eb95e97914db73c2ae65d93e1a09c55

bash-5.1$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
3988f947a319   app-net   bridge    local
118f65fe21ff   bridge    bridge    local
e5203f00748b   host      host      local
82714d432fcc   none      null      local
```

### Step 2: Run containers in it
```sh
bash-5.1$ docker run -d --name web --network app-net nginx
bc359e13bbc3acbfee3eac422d425099fde131374fdea1fd97c16823b533fc4b

bash-5.1$ docker run -d --name api --network app-net busybox sleep 3600
Unable to find image 'busybox:latest' locally
latest: Pulling from library/busybox
e59838ecfec5: Pull complete 
Digest: sha256:2383baad1860bbe9d8a7a843775048fd07d8afe292b94bd876df64a69aae7cb1
Status: Downloaded newer image for busybox:latest
e9cef19c57482b38c4f17d6728e7217980eaee02e0641a5023a9719748ede1dd

bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS     NAMES
e9cef19c5748   busybox   "sleep 3600"             21 seconds ago       Up 20 seconds                 api
bc359e13bbc3   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp    web
d70dc3b59c89   nginx     "/docker-entrypoint.…"   10 minutes ago       Up 10 minutes       80/tcp    web2
2dd1fb169f27   nginx     "/docker-entrypoint.…"   10 minutes ago       Up 10 minutes       80/tcp    web1
```

### Step 3: Test DNS
```sh
bash-5.1$ docker exec api ping -c 5 web
PING web (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.119 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.086 ms
64 bytes from 172.18.0.2: seq=2 ttl=64 time=0.143 ms
64 bytes from 172.18.0.2: seq=3 ttl=64 time=0.085 ms
64 bytes from 172.18.0.2: seq=4 ttl=64 time=0.086 ms
bash-5.1$ 
```

#### Note: 
  - User-defined bridge provides:
    
    - DNS ✅
    - Isolation ✅
    - Clean service discovery ✅
