## Deploying a Service on Docker Overlay Network

### Step 1: Initialize a Docker Swarm.
```sh
bash-5.1$  docker swarm init
Swarm initialized: current node (qfn8motckmjs5eea9jvx2l7ln) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0h9lh29mqvkzlv90t4mplc0m3jw5kv5bl5a9zuysqlairlk2dw-equkykkjvvcmt1goxo28wu6iw 172.20.0.5:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
### Step 2: Create an overlay network named my_overlay_network.
```sh
bash-5.1$ docker network create --driver overlay my_overlay_network
T4nwm1ofs5ydha6neeg1t3esk

bash-5.1$ docker network ls
NETWORK ID     NAME                 DRIVER    SCOPE
ceb2bcd86b7a   bridge               bridge    local
5171484d9426   docker_gwbridge      bridge    local
9a6aaa901878   host                 host      local
octa8oe8v9er   ingress              overlay   swarm
t4nwm1ofs5yd   my_overlay_network   overlay   swarm
022e79526049   none                 null      local
```

### Step 3: Deploy a new service called overlay_service using the nginx image on the my_overlay_network.
```sh
bash-5.1$ docker service create --name overlay_service --network my_overlay_network --replicas 1 nginx
9nrv4mfrcjeqoe0hdlsorj54a
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 

bash-5.1$ docker service ls
ID             NAME              MODE         REPLICAS   IMAGE          PORTS
9nrv4mfrcjeq   overlay_service   replicated   1/1        nginx:latest  
```

### Step 4: Scale the service to 3 replicas.
```sh
bash-5.1$ docker service scale overlay_service=3
overlay_service scaled to 3
overall progress: 3 out of 3 tasks 
1/3: running   [==================================================>] 
2/3: running   [==================================================>] 
3/3: running   [==================================================>] 
verify: Service converged 

bash-5.1$ docker service ls
ID             NAME              MODE         REPLICAS   IMAGE          PORTS
9nrv4mfrcjeq   overlay_service   replicated   3/3        nginx:latest   
```

### Step 5: Verify the deployment and scaling by listing the services and tasks.
```sh
bash-5.1$ docker service ps overlay_service

ID             NAME                IMAGE          NODE           DESIRED STATE   CURRENT STATE                ERROR     PORTS
q2sfoumbwc40   overlay_service.1   nginx:latest   de25e7f92cf9   Running         Running about a minute ago             
z21nugj3vkeh   overlay_service.2   nginx:latest   de25e7f92cf9   Running         Running 41 seconds ago                 
i4db8ciimu5c   overlay_service.3   nginx:latest   de25e7f92cf9   Running         Running 41 seconds ago                 
bash-5.1$ 
```
