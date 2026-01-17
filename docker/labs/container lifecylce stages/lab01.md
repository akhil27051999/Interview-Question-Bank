### CONTAINER CREATED

```sh
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
bash-5.1$ docker create --name nginx-cont nginx
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
32f3cdffb2a0a866f91761750792449e5ef4c3bfcec7f095517c5d93fcddd7ad

# Note: Container is created from a image but not yet started 
# > File system is prepared but no processes is running

bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS    PORTS     NAMES
32f3cdffb2a0   nginx     "/docker-entrypoint.…"   3 seconds ago   Created             nginx-cont
```

### CONTAINER RUNNING 
> COMMAND: docker start <cont_id/cont_name>

```sh
bash-5.1$ docker start nginx-cont
nginx-cont
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS         PORTS     NAMES
32f3cdffb2a0   nginx     "/docker-entrypoint.…"   21 seconds ago   Up 2 seconds   80/tcp    nginx-cont

# NOTE : Also use command 'docker run -d --name <cont_name> nginx' to start/run new container which is not created before
```

### CONTAINER PAUSE/UNPAUSE
```sh
bash-5.1$ docker pause nginx-cont
nginx-cont
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                   PORTS     NAMES
32f3cdffb2a0   nginx     "/docker-entrypoint.…"   49 seconds ago   Up 30 seconds (Paused)   80/tcp    nginx-cont
bash-5.1$ docker unpause nginx-cont
nginx-cont
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
32f3cdffb2a0   nginx     "/docker-entrypoint.…"   57 seconds ago   Up 39 seconds   80/tcp    nginx-cont
```

### CONTAINER STOP
> COMMAND: docker stop <cont-name> / docker kill <cont-name>
```sh
bash-5.1$ docker stop nginx-cont
nginx-cont
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS                     PORTS     NAMES
32f3cdffb2a0   nginx     "/docker-entrypoint.…"   About a minute ago   Exited (0) 2 seconds ago             nginx-cont
```

### CONTAINER RESTARTING
```sh
bash-5.1$ docker run -d --restart=on-failure --name nginx-cont nginx
docker: Error response from daemon: Conflict. The container name "/nginx-cont" is already in use by container "32f3cdffb2a0a866f91761750792449e5ef4c3bfcec7f095517c5d93fcddd7ad". You have to remove (or rename) that container to be able to reuse that name.
See 'docker run --help'.
bash-5.1$ docker rm nginx-cont
nginx-cont
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```
####  RESTART POLICY: ON FAILURE
```sh
bash-5.1$ docker run -d --restart=on-failure --name nginx-cont nginx
0a6daef2f3f68d2713b84aa6d914e269864687b9716c2a2461413eccda05d2b2
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
0a6daef2f3f6   nginx     "/docker-entrypoint.…"   6 seconds ago   Up 5 seconds   80/tcp    nginx-cont
bash-5.1$ docker kill nginx-cont
nginx-cont
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS                       PORTS     NAMES
0a6daef2f3f6   nginx     "/docker-entrypoint.…"   About a minute ago   Exited (137) 4 seconds ago             nginx-cont
```

#### RESTART POLICY: ALWAYS
```sh
bash-5.1$ docker run -d --restart=always --name nginx-cont nginx
dbf03e67021e8b38cabd9b78e78fd469dfbd00ee2ac914769c4e86fc7c9ac4ed
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
dbf03e67021e   nginx     "/docker-entrypoint.…"   5 seconds ago   Up 4 seconds   80/tcp    nginx-cont
bash-5.1$ docker kill nginx-cont
nginx-cont
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                       PORTS     NAMES
dbf03e67021e   nginx     "/docker-entrypoint.…"   18 seconds ago   Exited (137) 2 seconds ago             nginx-cont
bash-5.1$ 

# NOTE: Docker restart policies only apply to unintentional exits 
# — any user-initiated stop (stop or kill) disables restarts.
```
