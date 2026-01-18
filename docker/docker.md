# Docker — Q&A (DevOps Interview Prep)

## Table of Contents (Docker)
- [Category 1: Docker Fundamentals & Architecture](#category-1-docker-fundamentals--architecture)
- [Category 2: Dockerfile & Image Management](#category-2-dockerfile--image-management)
- [Category 3: Storage & Volumes](#category-3-storage--volumes)
- [Category 4: Networking](#category-4-networking)
- [Category 5: Security & Best Practices](#category-5-security--best-practices)
- [Category 6: Troubleshooting & Real-time Scenarios](#category-6-troubleshooting--real-time-scenarios)
- [Category 7: Advanced Concepts](#category-7-advanced-concepts)

## Category 1: Docker Fundamentals & Architecture

### 1. What is Docker and how does it differ from traditional virtualization?
**Answer**

<img width="1440" height="820" alt="image" src="https://github.com/user-attachments/assets/5f46af2a-f442-4629-8b9d-12ed372508fd" />

Docker is a platform for developing, shipping, and running applications in containers.

Key Differences:
- **Virtualization:** Runs a complete OS on a hypervisor; higher resource overhead.  
- **Containers:** Share the host OS kernel, run isolated processes; lightweight and fast.  
- **Startup Time:** VMs take minutes; containers start in seconds.  
- **Resource Usage:** VMs often require GBs of memory; containers can work in MBs.

---

### 2. Explain Docker architecture and its main components.
**Answer**
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e142f46d-1389-4964-bd5c-f5d010293b8e" />

- **Docker Daemon:** Background service that manages containers, images, networks, volumes.  
- **Docker Client:** CLI (`docker`) used to interact with the daemon (e.g., `docker run`, `docker build`).  
- **Docker Images:** Read-only templates used to create containers.  
- **Docker Containers:** Runnable instances of images with a writable layer.  
- **Docker Registry:** Storage and distribution for images (Docker Hub, private registries).  
- **Dockerfile:** Script with build instructions for creating images.

---

### 3. What is the difference between a Docker image and container?
**Answer**

- **Image:** Read-only template containing application code and dependencies; built from a Dockerfile.  
- **Container:** Runnable instance of an image; has a writable layer on top.

---

### 4. Explain Docker container lifecycle states.
**Answer**

- **Created:** Container object created but not started.  
- **Running:** Container is actively executing.  
- **Paused:** Process execution suspended (using cgroups freezer).  
- **Stopped:** Process terminated but metadata remains.  
- **Removed:** Container deleted from the host.

#### DOCKER LIFE-CYCLE STAGES EXPLAINATION

```bash
# --------------- CONTAINER CREATED ---------------

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

# --------------- CONTAINER RUNNING ---------------

# COMMAND: docker start <cont_id/cont_name>

bash-5.1$ docker start nginx-cont
nginx-cont
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS         PORTS     NAMES
32f3cdffb2a0   nginx     "/docker-entrypoint.…"   21 seconds ago   Up 2 seconds   80/tcp    nginx-cont

# NOTE : Also use command 'docker run -d --name <cont_name> nginx' to start/run new container which is not created before

# --------------- CONTAINER PAUSE/UNPAUSE ---------------

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

# --------------- CONTAINER STOP ---------------

# COMMAND: docker stop <cont-name> / docker kill <cont-name>

bash-5.1$ docker stop nginx-cont
nginx-cont
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS                     PORTS     NAMES
32f3cdffb2a0   nginx     "/docker-entrypoint.…"   About a minute ago   Exited (0) 2 seconds ago             nginx-cont

# --------------- CONTAINER RESTARTING ---------------

bash-5.1$ docker run -d --restart=on-failure --name nginx-cont nginx
docker: Error response from daemon: Conflict. The container name "/nginx-cont" is already in use by container "32f3cdffb2a0a866f91761750792449e5ef4c3bfcec7f095517c5d93fcddd7ad". You have to remove (or rename) that container to be able to reuse that name.
See 'docker run --help'.
bash-5.1$ docker rm nginx-cont
nginx-cont
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES


# --------------- RESTART POLICY: ON FAILURE ---------------

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

# --------------- RESTART POLICY: ALWAYS ---------------

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

---

## Category 2: Dockerfile & Image Management

### 5. What is a Dockerfile and explain common instructions?
**Answer**

A Dockerfile is a text file with instructions to build a Docker image.

Common Instructions:
| Instruction    | Purpose                                 | When it Runs    | Key Notes / Best Practices                                  |
| -------------- | --------------------------------------- | --------------- | ----------------------------------------------------------- |
| **FROM**       | Specifies the base image                | Build time      | Must be first (except `ARG`); supports multi-stage builds   |
| **RUN**        | Executes commands to modify the image   | Build time      | Each RUN creates a layer; combine commands to reduce layers |
| **COPY**       | Copies files from host to image         | Build time      | Preferred over `ADD` (predictable behavior)                 |
| **ADD**        | Copies files with extra features        | Build time      | Can extract tar files & fetch URLs (use sparingly)          |
| **WORKDIR**    | Sets working directory inside container | Build time      | Avoids repeated `cd`; auto-creates directory                |
| **CMD**        | Default command when container starts   | Runtime         | Can be overridden by `docker run`                           |
| **ENTRYPOINT** | Main executable of the container        | Runtime         | Harder to override; often paired with CMD                   |
| **EXPOSE**     | Documents container listening ports     | Build time      | Does NOT publish ports                                      |
| **ENV**        | Sets environment variables              | Build & Runtime | Available inside container                                  |
| **ARG**        | Defines build-time variables            | Build time only | Not available at runtime unless copied to ENV               |
| **VOLUME**     | Creates mount point for persistent data | Runtime         | Data survives container restarts                            |
| **USER**       | Runs container as a specific user       | Runtime         | Security best practice (avoid root)                         |
| **LABEL**      | Adds metadata to the image              | Build time      | Useful for documentation & automation                       |

#### Basic Dockerfile Implementation for project

```dockerfile
# app/Dockerfile
FROM python:3.10-slim

# Set working directory
WORKDIR /app

# Copy requirements and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Expose port
EXPOSE 5000

# Set environment variables
ENV FLASK_APP=wsgi.py
ENV FLASK_ENV=production

# Run the application with Gunicorn
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "wsgi:app"]
```
---

### 6. What is the difference between CMD and ENTRYPOINT?
**Answer**

- **CMD:** Default command/arguments that can be overridden at runtime (`docker run image command`).  
- **ENTRYPOINT:** The main command that always runs; `CMD` acts as default arguments to `ENTRYPOINT`.  

**Best Practice:** Use `ENTRYPOINT` for the main application and `CMD` for default arguments.

---

### 7. How do you optimize Docker image size?
**Answer**

**1. Layer Caching**
  - Copy and install dependencies before copying the application code to make use of Docker build cache.
    
    - `Good:`
	```dockerfile
	COPY requirements.txt .
	RUN pip install -r requirements.txt
	COPY . .
	```

	- `Bad:`
	```dockerfile
	COPY . .
	RUN pip install -r requirements.txt
	```

**2. Use Minimal Base Images**
  - Alpine or slim variants reduce image size. Be aware of compatibility (some wheels require build tools).
**3. Install Only Required Packages**
  - Avoid installing unnecessary OS packages in the runtime image.

**4. Use --no-cache-dir with pip**
  - Prevents caching wheels in the image: pip install --no-cache-dir -r requirements.txt

**5. Multi-stage Builds**
  - Build dependencies in a builder stage and copy only the installed packages to the runtime image to keep final image small.
**6. .dockerignore**
  - Exclude local files, tests, and secrets from the build context to speed up builds and avoid leaking secrets.

---

### 8. What are multi-stage builds and why are they useful?
**Answer**

Multi-stage builds use multiple `FROM` statements so we can build artifacts in one stage and copy only the final artifacts into a smaller runtime image. They reduce final image size and keep build dependencies out of the runtime image.

**Why use them?**

  - Produce significantly smaller images by excluding build-time dependencies (compilers, headers).
  - Improve security by reducing attack surface.
  - Separate build artifacts from runtime files.
    
  - `Project multi-stage example`: 

    build stage installs dependencies into /root/.local, runtime stage copies them over and runs as non-root user.

	```dockerfile
	# Stage 1: Build Python dependencies
	FROM python:3.10-alpine AS build
	WORKDIR /api/app

	# Install build dependencies for psycopg2
	RUN apk add --no-cache gcc musl-dev postgresql-dev

	# Copy requirements and install dependencies
	COPY requirements.txt ./
	RUN pip install --user --no-cache-dir -r requirements.txt \
	    && find /root/.local -name '*.pyc' -delete \
		&& find /root/.local -name '__pycache__' -delete

	# Stage 2: Main application image
	FROM python:3.10-alpine AS main
	WORKDIR /api
	
	# Install PostgreSQL client for pg_isready
	RUN apk add --no-cache postgresql-client

	# Copy installed Python packages from build stage
	COPY --from=build /root/.local /root/.local

	# Copy the entire app folder
	COPY . ./app

	# Add pip binaries to PATH
	ENV PATH=/root/.local/bin:$PATH
	ENV FLASK_APP=app/wsgi.py
	
	# Expose port
	EXPOSE 5000

	# Start Gunicorn server
	CMD ["gunicorn", "app.wsgi:app"]
	```

---

## Category 3: Storage & Volumes

### 9. What are Docker volumes and when to use them?
**Answer**

Volumes provide persistent storage for containers independent of the container lifecycle.

Common Use Cases:
- Database data persistence.  
- Configuration or state that must survive container recreation.  
- Sharing data between multiple containers.  
- Backups and restores.

---

### 10. Difference between bind mounts and named volumes?
**Answer**

- **Bind Mounts:** Map a host directory or file into the container. Dependent on the host filesystem and paths. Great for development (live code).  
- **Named Volumes:** Managed by Docker, stored in Docker's storage area, more portable across hosts (when used with volume drivers). Preferred for production.

---

### 11. How do you manage sensitive data in Docker?
**Answer**

- Use **Docker Secrets** in Swarm mode for sensitive information.  
- Avoid storing secrets in environment variables when possible.  
- Mount files with restricted permissions (read-only) into the container.  
- Use external secret managers (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault).  
- Ensure secrets are not baked into images or committed to source control.

---

### Lab Scenario: Using Docker Volumes with MySQL (Named Volumes vs Bind Mounts)

**Goal:** Run a MySQL container with persistent data using a Docker volume. Practice creating, inspecting, sharing, and removing volumes.

#### Named Volumes

```sh
bash-5.1$ docker volume ls
DRIVER    VOLUME NAME

# --------------- Step 1: Create a Docker Volume ---------------

bash-5.1$ docker volume create mydbdata
mydbdata

bash-5.1$ docker volume ls
DRIVER    VOLUME NAME
local     mydbdata

# --------------- Step 2: Run MySQL Container Using the Volume ---------------

bash-5.1$ docker run -d \
> --name mysql-db \
> -e MYSQL_ROOT_PASSWORD=mypasswd \
> -e MYSQL_DATABASE=testdb \
> -v mydbdata:/var/lib/mysql \
> mysql:8
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
f6a82162c6f45938638a139cfe80a43fcd202f656253ff5082b0d8d28c082c23

bash-5.1$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
mysql        8         54c6e074ef93   14 hours ago   785MB

bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                 NAMES
f6a82162c6f4   mysql:8   "docker-entrypoint.s…"   9 seconds ago   Up 9 seconds   3306/tcp, 33060/tcp   mysql-db


bash-5.1$ docker inspect -f '{{.Path}} {{.Args}}' mysql-db
docker-entrypoint.sh [mysqld]

# --------------- Step 3: Connect to MySQL and Add Data ---------------

bash-5.1$ docker exec -it mysql-db /bin/sh

sh-5.1# mysql -u root -p mypasswd
Enter password: 
ERROR 1049 (42000): Unknown database 'mypasswd'
sh-5.1# mysql -u root -pmypasswd
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.4.7 MySQL Community Server - GPL

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> USE testdb;
Database changed

mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| testdb             |
+--------------------+
5 rows in set (0.00 sec)

mysql> CREATE TABLE users (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(50));
Query OK, 0 rows affected (0.03 sec)

mysql> INSERT INTO users (name) VALUES ('Akhil'), ('Ravi');
Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM users;
+----+-------+
| id | name  |
+----+-------+
|  1 | Akhil |
|  2 | Ravi  |
+----+-------+
2 rows in set (0.00 sec)

mysql> exit
Bye

sh-5.1 # ctrl+x

# --------------- Step 4: Stop and Remove the Container ---------------

bash-5.1$ docker stop mysql-db
mysql-db
bash-5.1$ docker rm mysql-db
mysql-db

# --------------- Step 5: Recreate Container Using the Same Volume ---------------

bash-5.1$ docker run -d \
> --name mysql-db2 \
> -e MYSQL_ROOT_PASSWORD=mypasswd \
> -e MYSQL_DATABASE=testdb \
> -v mydbdata:/var/lib/mysql \
> mysql:8
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
1a8707e864e9a1648a6d3d1786104dc630111cb590b43b8425e70979ae38ca1f

bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                 NAMES
1a8707e864e9   mysql:8   "docker-entrypoint.s…"   5 seconds ago   Up 5 seconds   3306/tcp, 33060/tcp   mysql-db2

bash-5.1$ docker exec -it mysql-db2 mysql -u root -pmypasswd
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.4.7 MySQL Community Server - GPL

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> USE testdb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

mysql> SELECT * FROM users;
+----+-------+
| id | name  |
+----+-------+
|  1 | Akhil |
|  2 | Ravi  |
+----+-------+
2 rows in set (0.00 sec)

# We'll see the previous records. Data persisted across container removal.

mysql>

# --------------- Step 6: Share the same volume between 2 containers ---------------

bash-5.1$ docker run -d --name mysql-lab3 -v mydbdata:/var/lib/mysql mysql:8
1a8707e864e9a1648a6d3d1786104dc630111cb590b43b8425e70979ae38ca1f

# Both containers see the same data.

# --------------- Step 7: Test backup & restore ---------------

bash-5.1$ docker run --rm -v mydbdata:/data busybox tar cvf /backup.tar /data
```
---

#### Bind Mounts 

```sh
# --------------- Step 1: Create a Host Directory ---------------

bash-5.1$ mkdir -p ~/mysql-data
bash-5.1$ ls
mysql-data
bash-5.1$ docker run -d \
> --name mysql-db \
> -e MYSQL_ROOT_PASSWORD=mypasswd \
> -e MYSQL_DATABASE=testdb \
> -v ~/mysql-data:var/lib/mysql \
> mysql:8
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
docker: Error response from daemon: invalid volume specification: '/home/user/mysql-data:var/lib/mysql': invalid mount config for type "bind": invalid mount path: 'var/lib/mysql' mount path must be absolute.
See 'docker run --help'.

# --------------- Step 2: Run MySQL Using Bind Mount ---------------

bash-5.1$ docker run -d --name mysql-db -e MYSQL_ROOT_PASSWORD=mypasswd -e MYSQL_DATABASE=testdb -v ~/mysql-data:/var/lib/mysql mysql:8
6b016784f88acabaae09f27eb9c3473368f1a87cb83b69c487f06938cab9574c
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS          PORTS                 NAMES
6b016784f88a   mysql:8   "docker-entrypoint.s…"   About a minute ago   Up 59 seconds   3306/tcp, 33060/tcp   mysql-db

# --------------- Step 3: Add Sample Data ---------------

bash-5.1$ docker exec -it mysql-db /bin/sh

sh-5.1# mysql -u root -pmypasswd
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.4.7 MySQL Community Server - GPL

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> USE testdb;
Database changed

mysql> CREATE TABLE test_table (
    ->   id INT AUTO_INCREMENT PRIMARY KEY,
    ->   name VARCHAR(50)
    -> );
Query OK, 0 rows affected (0.03 sec)

mysql> SHOW TABLES;
+------------------+
| Tables_in_testdb |
+------------------+
| test_table       |
+------------------+
1 row in set (0.00 sec)

mysql> INSERT INTO test_table (name)
    -> VALUES ('BindMountUser');
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM test_table;
+----+---------------+
| id | name          |
+----+---------------+
|  1 | BindMountUser |
+----+---------------+
1 row in set (0.00 sec)

mysql> 

mysql> exit
Bye
sh-5.1# crtl+d
exit

bash-5.1$ ls
mysql-data
bash-5.1$ ls ~/mysql-data
#ib_16384_0.dblwr      auto.cnf               ca-key.pem             ib_buffer_pool         mysql.ibd              private_key.pem        sys
#ib_16384_1.dblwr      binlog.000001          ca.pem                 ibdata1                mysql.sock             public_key.pem         testdb
#innodb_redo           binlog.000002          client-cert.pem        ibtmp1                 mysql_upgrade_history  server-cert.pem        undo_001
#innodb_temp           binlog.index           client-key.pem         mysql                  performance_schema     server-key.pem         undo_002

# --------------- Step 4: Stop and Remove Container ---------------

bash-5.1$ docker stop mysql-db
mysql-db
bash-5.1$ docker rm mysql-db
mysql-db
bash-5.1$ ls mysql-data/
#ib_16384_0.dblwr      #innodb_temp           binlog.000002          ca.pem                 ib_buffer_pool         mysql.ibd              performance_schema     server-cert.pem        testdb
#ib_16384_1.dblwr      auto.cnf               binlog.index           client-cert.pem        ibdata1                mysql.sock             private_key.pem        server-key.pem         undo_001
#innodb_redo           binlog.000001          ca-key.pem             client-key.pem         mysql                  mysql_upgrade_history  public_key.pem         sys                    undo_002
bash-5.1$ 
bash-5.1$

```
#### Interview Tip:

> Use named volumes for production data (Docker manages it), and bind mounts for development (you need direct access to host files).

#### Key Differences: Named vs Bind Mounts

| Feature           | Named Volume                                | Bind Mount                                 |
| ----------------- | ------------------------------------------- | ------------------------------------------ |
| Managed by Docker | ✅                                          | ❌                                        |
| Location          | Docker decides (`/var/lib/docker/volumes/`) | Host directory you choose                  |
| Ease of use       | Easy to backup, share, reuse                | More flexible, direct access to host files |
| Portability       | High                                        | Medium (depends on host path)              |
| Use Case          | Databases, production data                  | Dev environment, config sharing, logs      |

---

## Category 4: Networking

### 12. Explain Docker network types.
**Answer**

- **bridge:** Default network for standalone containers on a host. Containers on the same bridge can communicate.  
- **host:** Container shares the host network namespace (no network isolation).  
- **overlay:** Connects containers across multiple Docker daemons (used by Swarm).  
- **macvlan:** Assigns a MAC address to containers, making them look like physical devices on the network.  
- **none:** Container has no network interfaces.

---

### 13. How do containers communicate with each other?
**Answer**

- On the same bridge network: use container name as hostname (Docker DNS).  
- On custom user-defined networks: DNS-based service discovery is available and is recommended.  
- Using published ports: access containers from the host via mapped ports.  
- Legacy: links (deprecated) — avoid using links.

---

### Docker Networking Lab Scenario: Build, isolate, connect, and scale containers using all Docker network 

**Goal**
- Bridge (default & user-defined)
- Container DNS
- Port mapping
- Host network
- None network
- Overlay network (Swarm)
- Network inspection & troubleshooting
---

#### Default Bridge Network

```sh

# --------------- Step 1: Run two containers ---------------

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

# --------------- Step 2: Inspect default bridge ---------------

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

# --------------- Step 3: Test DNS (will fail) ---------------

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

# NOTE: 
# - Default bridge has no automatic DNS ❌
# - Containers must use IPs → bad practice
```

---

#### User-Defined Bridge (Best Practice)

```sh

# --------------- Step 3: Create custom bridge ---------------

bash-5.1$ docker network create app-net
3988f947a319bc6830e461fe4485570d2eb95e97914db73c2ae65d93e1a09c55

bash-5.1$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
3988f947a319   app-net   bridge    local
118f65fe21ff   bridge    bridge    local
e5203f00748b   host      host      local
82714d432fcc   none      null      local

# --------------- Step 4: Run containers in it ---------------

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

# --------------- Step 5: Test DNS ---------------

bash-5.1$ docker exec api ping -c 5 web
PING web (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.119 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.086 ms
64 bytes from 172.18.0.2: seq=2 ttl=64 time=0.143 ms
64 bytes from 172.18.0.2: seq=3 ttl=64 time=0.085 ms
64 bytes from 172.18.0.2: seq=4 ttl=64 time=0.086 ms
bash-5.1$ 

# Note: 
# User-defined bridge provides:
# - DNS ✅
# - Isolation ✅
# - Clean service discovery ✅
```
---

#### Real App Communication (Web + DB)

```sh

# --------------- Step 6: Run MySQL ---------------

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

# --------------- Step 7: Access DB from another container ---------------

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

# NOTE: 
# - Containers talk via service names ✅
# - No IP hardcoding ✅
```

---

#### Port Mapping (Host ↔ Container)

```sh
# --------------- Step 8: Expose nginx ---------------

bash-5.1$ docker run -d \
  --name web-pub \
  --network app-net \
  -p 8080:80 \
  nginx
2948619f33021eabe485fb6dde9a6a673a4bce1d72c9da5f738c3ef04542425f

bash-5.1$ docker ps | grep web-pub
2948619f3302   nginx     "/docker-entrypoint.…"   15 seconds ago   Up 14 seconds   0.0.0.0:8080->80/tcp, :::8080->80/tcp   web-pub

# --------------- Step 9: Test from host ---------------

bash-5.1$ curl http://localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

# Note:
# -p is host → container
# Containers do NOT need ports mapped to talk internally

```
---

#### Network Isolation Test

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


# ✅ This tells us two things
# 1. hacker is on isolated-net
# 2. web is on app-net
# 3. Docker DNS only works within the same network

# ❌ Containers on different Docker networks cannot: 
# - Resolve each other’s names 
# - Reach each other’s IPs
# - Communicate at all
# - This is intentional network isolation, not an error.

```
---

#### None Network (Total Isolation)

```sh

# --------------- Step 13: Run none network ---------------

bash-5.1$ docker run -it --network none busybox

# --------------- Step 14: Test networking ---------------

/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever

/ # ping google.com
ping: bad address 'google.com'

# Used for batch jobs, security workloads
```
---

#### Host Network (No Isolation)

```sh

# --------------- Step 15: Run nginx on host network ---------------

bash-5.1$ docker run -d --network host nginx
98708b650334c6e792fbf755b9ef52724003fe5433691358a0d706dd02461d74

bash-5.1$ docker network ls
NETWORK ID     NAME           DRIVER    SCOPE
3988f947a319   app-net        bridge    local
118f65fe21ff   bridge         bridge    local
e5203f00748b   host           host      local
df26996a588f   isolated-net   bridge    local
82714d432fcc   none           null      local

# --------------- Step 16: Access directly ---------------

bash-5.1$ curl http://localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
bash-5.1$ 

# NOTE:
# - Container shares host network
# - No port mapping
# - Linux only
# - Risky but fast
```
---

#### Configuring Docker Bridge Network

```sh

# --------------- Step 1: Create a custom Docker bridge network named my_bridge_network. ---------------

bash-5.1$ docker network create --driver bridge my_bridge_network
dc75b836fc70f758979ce580c02f5574d442594bbf863f85dcfd9fc0a0b9d422

# --------------- Step 2: Run a new container called web_container using the nginx image and connect it to the my_bridge_network. ---------------

bash-5.1$ docker run --name web_container --network my_bridge_network -d nginx
e87c9e07f8e6eae4feaf2ae9812939edd248b99173aa11704bdcdc6c94579a4f

# --------------- Verify that the container is connected to the bridge network by inspecting the network and container. ---------------

bash-5.1$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
e87c9e07f8e6   nginx     "/docker-entrypoint.…"   5 seconds ago   Up 4 seconds   80/tcp    web_container

bash-5.1$ docker network inspect my_bridge_network
[
    {
        "Name": "my_bridge_network",
        "Id": "dc75b836fc70f758979ce580c02f5574d442594bbf863f85dcfd9fc0a0b9d422",
        "Created": "2026-01-05T16:36:12.898053584Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
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
            "e87c9e07f8e6eae4feaf2ae9812939edd248b99173aa11704bdcdc6c94579a4f": {
                "Name": "web_container",
                "EndpointID": "dc9f27e91d73a2f84c53fde7e30a8852ce587d0dd00c9abbf34b710b592b8243",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

bash-5.1$ docker inspect web_container
[
    {
        "Id": "e87c9e07f8e6eae4feaf2ae9812939edd248b99173aa11704bdcdc6c94579a4f",
        "Created": "2026-01-05T16:36:52.873873187Z",
        "Path": "/docker-entrypoint.sh",
        "Args": [
            "nginx",
            "-g",
            "daemon off;"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 982,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2026-01-05T16:36:53.33867287Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:058f4935d1cbc026f046e4c7f6ef3b1d778170ac61f293709a2fc89b1cff7009",
        "ResolvConfPath": "/var/lib/docker/containers/e87c9e07f8e6eae4feaf2ae9812939edd248b99173aa11704bdcdc6c94579a4f/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/e87c9e07f8e6eae4feaf2ae9812939edd248b99173aa11704bdcdc6c94579a4f/hostname",
        "HostsPath": "/var/lib/docker/containers/e87c9e07f8e6eae4feaf2ae9812939edd248b99173aa11704bdcdc6c94579a4f/hosts",
        "LogPath": "/var/lib/docker/containers/e87c9e07f8e6eae4feaf2ae9812939edd248b99173aa11704bdcdc6c94579a4f/e87c9e07f8e6eae4feaf2ae9812939edd248b99173aa11704bdcdc6c94579a4f-json.log",
        "Name": "/web_container",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "my_bridge_network",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "private",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": null,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/b29ad46ba89795f0e99496c30c5c3f03b96449a7f2178c0c6146c9209e6f39b4-init/diff:/var/lib/docker/overlay2/02159621de3c6c8a3fcd77ff42a48e56a87209cae476c57e1e9ecfc3300146ae/diff:/var/lib/docker/overlay2/ecb6f730fa8f8106242397eb6487a6dea1f7a5948ed01f9ccc5f61b8698c2982/diff:/var/lib/docker/overlay2/c6608b3e3b25aa909e3305ca9b0877276aa5295a173d0e1fbab0a351097f283a/diff:/var/lib/docker/overlay2/a2743dd3ebf090fe411a9578385a8a070aefa809be8b0aa50a3a6e83ea28fd5f/diff:/var/lib/docker/overlay2/f3bfd8cb288922c3c02045e77ff7034136fa1eb20c8c71e054b35932b4413677/diff:/var/lib/docker/overlay2/c5b3404a57412fa3108389e91591af61f8e13aeea5c9ffe4a2a5e744db7832b8/diff:/var/lib/docker/overlay2/ec6ea0cffbdd68140e6b702d09012a92b00e816b7f1742e971187e32a66556fd/diff",
                "MergedDir": "/var/lib/docker/overlay2/b29ad46ba89795f0e99496c30c5c3f03b96449a7f2178c0c6146c9209e6f39b4/merged",
                "UpperDir": "/var/lib/docker/overlay2/b29ad46ba89795f0e99496c30c5c3f03b96449a7f2178c0c6146c9209e6f39b4/diff",
                "WorkDir": "/var/lib/docker/overlay2/b29ad46ba89795f0e99496c30c5c3f03b96449a7f2178c0c6146c9209e6f39b4/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "e87c9e07f8e6",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.29.4",
                "NJS_VERSION=0.9.4",
                "NJS_RELEASE=1~trixie",
                "PKG_RELEASE=1~trixie",
                "DYNPKG_RELEASE=1~trixie"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "Image": "nginx",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGQUIT"
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "13177d4fbeb0904255b0e25e30ea5d35f5a262838cef561202259f51f29710d3",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "80/tcp": null
            },
            "SandboxKey": "/var/run/docker/netns/13177d4fbeb0",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "",
            "Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "",
            "IPPrefixLen": 0,
            "IPv6Gateway": "",
            "MacAddress": "",
            "Networks": {
                "my_bridge_network": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": [
                        "e87c9e07f8e6"
                    ],
                    "NetworkID": "dc75b836fc70f758979ce580c02f5574d442594bbf863f85dcfd9fc0a0b9d422",
                    "EndpointID": "dc9f27e91d73a2f84c53fde7e30a8852ce587d0dd00c9abbf34b710b592b8243",
                    "Gateway": "172.18.0.1",
                    "IPAddress": "172.18.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:12:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
bash-5.1$ 

```
---
#### Deploying a Service on Docker Overlay Network

```sh

# --------------- Step 1: Initialize a Docker Swarm. ---------------

bash-5.1$  docker swarm init
Swarm initialized: current node (qfn8motckmjs5eea9jvx2l7ln) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0h9lh29mqvkzlv90t4mplc0m3jw5kv5bl5a9zuysqlairlk2dw-equkykkjvvcmt1goxo28wu6iw 172.20.0.5:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

# --------------- Step 2: Create an overlay network named my_overlay_network. ---------------

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

# --------------- Step 3: Deploy a new service called overlay_service using the nginx image on the my_overlay_network. ---------------

bash-5.1$ docker service create --name overlay_service --network my_overlay_network --replicas 1 nginx
9nrv4mfrcjeqoe0hdlsorj54a
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 

bash-5.1$ docker service ls
ID             NAME              MODE         REPLICAS   IMAGE          PORTS
9nrv4mfrcjeq   overlay_service   replicated   1/1        nginx:latest  
 
# --------------- Step 4: Scale the service to 3 replicas. ---------------

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

# --------------- Step 5: Verify the deployment and scaling by listing the services and tasks. ---------------

bash-5.1$ docker service ps overlay_service

ID             NAME                IMAGE          NODE           DESIRED STATE   CURRENT STATE                ERROR     PORTS
q2sfoumbwc40   overlay_service.1   nginx:latest   de25e7f92cf9   Running         Running about a minute ago             
z21nugj3vkeh   overlay_service.2   nginx:latest   de25e7f92cf9   Running         Running 41 seconds ago                 
i4db8ciimu5c   overlay_service.3   nginx:latest   de25e7f92cf9   Running         Running 41 seconds ago                 
bash-5.1$ 

```
---
#### Docker Network Connectivity

- Test the network connectivity between two Docker containers by pinging from one container to another.
  - Create a network named my_network
  - Container Names: container1 and container2
  - Image: nginx and nginx:1.23.4


```sh

bash-5.1$ docker network create my_network

c7f2219d0f0818f3a885b7cc0c5c6cfc5d9fd108fe35550f77aade46797f5a5e

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

# --------------- Test the network connectivity between two Docker containers by pinging from one container to another ---------------

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
---

### 14. What is Docker Compose and when to use it?
**Answer**

Docker Compose lets you define and run multi-container applications using a YAML file (`docker-compose.yml`). Use it for:
- Local development and testing.  
- Defining simple multi-service stacks.  
- Orchestrating containers for small deployments (not a replacement for full orchestration platforms like Kubernetes).

---

### Project Docker Compose Stack (docker-compose.yml)

#### Purpose & Role:
- Multi-container application definition
- Service orchestration for Flask app, PostgreSQL, and Nginx

#### Service Architecture:

##### 1. Flask Application Service
- Builds from ./app directory context
- Uses environment variables from external file
- Health-check dependency on PostgreSQL
- Volume mounts for development (live code reload)

##### 2. PostgreSQL Database Service
- Official Postgres 15 image
- Persistent volume for data storage
- Health checks using pg_isready
- Port 5432 exposed for external connections

##### 3. Nginx Load Balancer Service
- Alpine-based lightweight image
- Custom configuration mounted as volume
- Port 80 exposed for web traffic
- Dependency on Flask app service

#### Docker Compose Architecture

```text
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Nginx (80)    │    │  Flask API(5000)│    │ PostgreSQL(5432)│
│   Load Balancer │────│   REST API      │────│   Database      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
        ↑                       ↑                       ↑
        │                       │                       │
┌─────────────────────────────────────────────────────────────────┐
│                    Docker Compose Network                       │
└─────────────────────────────────────────────────────────────────┘

```

#### Service Relationships:

```text
Nginx (Port 80) → Flask App (Port 5000) → PostgreSQL (Port 5432)
```

#### Key Features:
- Environment variable injection via external file
- Container health monitoring
- Automatic restart policies
- Persistent data volume for database
  
### Project Docker Compose File 
```yaml
services:
  flask-app:
    build: 
      context: ./app
      dockerfile: Dockerfile
    image: flask-app:1.0.0
    container_name: flask-app-container
    restart: always
    env_file:
      - ${ENV_FILE}
    depends_on:
      postgres:
        condition: service_healthy
    volumes:
      - ./:/app

  postgres:
    image: postgres:15
    container_name: postgres-container
    restart: always
    env_file:
      - ${ENV_FILE}
    ports:
      - "5432:5432"   
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-postgres} -d ${POSTGRES_DB:-postgres}"]
      interval: 10s
      timeout: 20s
      retries: 5

  nginx:
    image: nginx:alpine
    container_name: nginx-container
    restart: always
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - flask-app

volumes:
  pgdata:
    driver: local
```
---

### System Architecture Summary

#### Deployment Flow:
- Vagrant → Provisions AWS EC2 instance
- Provisioning Script → Installs Docker and dependencies
- Docker Compose → Orchestrates multi-container deployment
- Nginx → Routes traffic to Flask application
- Flask App → Processes requests with PostgreSQL backend

#### Network Exposure:
- External: Port 80 (Nginx) on AWS instance
- Internal: Container network between services
- Database: Port 5432 exposed for management access

#### Current Limitations:
- Single Flask instance (no load balancing between multiple app instances)
- Basic Nginx configuration without advanced routing rules
- Development-focused volume mounts in production setup
---

## Category 5: Security & Best Practices

### 15. What are Docker security best practices?
**Answer**

- Run containers as a non-root user where possible.  
- Use minimal and trusted base images.  
- Scan images for vulnerabilities (Trivy, Clair, Snyk, Docker Scout).  
- Pin base image versions (avoid `latest` in production).  
- Implement resource limits and runtime constraints.  
- Keep Docker daemon and host OS patched and up to date.  
- Use secret management and avoid baking secrets into images.

---

### 16. How do you limit container resources?
**Answer**

Example:
```bash
docker run --memory=512m --cpus=1.5 --pids-limit=100 my-app
```

- `--memory` (or `-m`): memory limit.  
- `--cpus`: limit CPU quota.  
- `--pids-limit`: limit number of processes.

---

### 17. What is container scanning and why is it important?
**Answer**

Container scanning analyzes images to find known vulnerabilities in packages and OS components.

Tools: **Trivy**, **Clair**, **Docker Scout**, **Snyk**.  
Importance: Discover and remediate vulnerabilities before deploying images to production.

---

## Category 6: Troubleshooting & Real-time Scenarios

### 18. Container is running but application isn't accessible. How to debug?
**Answer**

- Check container logs: `docker logs <container_name>`  
- Verify port mapping: `docker ps` and check `-p`/`--publish` flags.  
- Enter container: `docker exec -it <container_name> sh` (or bash) to test internal connectivity and services.  
- Check application configuration and listen address (e.g., `0.0.0.0` vs `localhost`).  
- Verify firewall and network settings on the host.

### Lab Scenario : Container is RUNNING but Application is NOT Accessible

`Interview Gold Answer`:

  - The container was running and the port was mapped correctly, but the application was bound to localhost.
  - After changing the bind address to 0.0.0.0, external access worked.**

```sh

# --------------- Problem : Container is RUNNING but Application is NOT Accessible ---------------

bash-5.1$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

# --------------- Run the container with a hidden mistake ---------------

bash-5.1$ docker run -d \
  --name web-app \
  -p 8080:3000 \
  hashicorp/http-echo \
  -listen=127.0.0.1:3000 \
  -text="Hello from Docker"
Unable to find image 'hashicorp/http-echo:latest' locally
latest: Pulling from hashicorp/http-echo
3d17c666b2a2: Pull complete 
e5dbef90bae3: Pull complete 
2047f8b74670: Pull complete 
eebb06941f3e: Pull complete 
02cd68c0cbf6: Pull complete 
d3c894b5b2b0: Pull complete 
b40161cd83fc: Pull complete 
65efb1cabba4: Pull complete 
13547472c521: Pull complete 
5d4746900976: Pull complete 
Digest: sha256:fcb75f691c8b0414d670ae570240cbf95502cc18a9ba57e982ecac589760a186
Status: Downloaded newer image for hashicorp/http-echo:latest
765fb25776f3bf2b2eb5fe59a798c2ccdaccca52566eb6309a3264156b09b674

bash-5.1$ docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS         PORTS                                                 NAMES
765fb25776f3   hashicorp/http-echo   "/http-echo -listen=…"   4 seconds ago   Up 3 seconds   5678/tcp, 0.0.0.0:8080->3000/tcp, :::8080->3000/tcp   web-app

# --------------- Curl the app and check the reachability ---------------

bash-5.1$ curl http://localhost:8080
curl: (56) Recv failure: Connection reset by peer

# TROUBLESHOOTING STEPS

# --------------- Step 1: Check container logs ---------------

bash-5.1$ docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED              STATUS              PORTS                                                 NAMES
765fb25776f3   hashicorp/http-echo   "/http-echo -listen=…"   About a minute ago   Up About a minute   5678/tcp, 0.0.0.0:8080->3000/tcp, :::8080->3000/tcp   web-app

# --------------- Step 2: Check container logs ---------------

bash-5.1$ docker logs web-app
2026/01/08 08:40:43 [INFO] server is listening on 127.0.0.1:3000

# --------------- Step 3: Verify port mapping ---------------

bash-5.1$ docker port web-app
3000/tcp -> 0.0.0.0:8080
3000/tcp -> :::8080

# --------------- Step 4: Enter the container and test internally ---------------

bash-5.1$ docker exec -it web-app sh
OCI runtime exec failed: exec failed: container_linux.go:380: starting container process caused: exec: "sh": executable file not found in $PATH: unknown

bash-5.1$ docker exec -it web-app /bin/sh
OCI runtime exec failed: exec failed: container_linux.go:380: starting container process caused: exec: "/bin/sh": stat /bin/sh: no such file or directory: unknown

bash-5.1$ curl http://localhost:8080
curl: (56) Recv failure: Connection reset by peer

# Problem
# - The application is bound to localhost (127.0.0.1)
# - Docker can’t forward traffic from outside the container to localhost.

# FIX THE ISSUE

# --------------- Step 1: Stop and remove the broken container ---------------

bash-5.1$ docker rm -f web-app
web-app
bash-5.1$ docker run -d \
  --name web-app \
  -p 8080:3000 \
  hashicorp/http-echo \
  -listen=0.0.0.0:3000 \
  -text="Hello from Docker"
ed9c7b0db37b771117cf4b147633b42838769ef743432ee4878951d69b348ab6

# --------------- Step 2: Verify Fix ---------------

bash-5.1$ curl http://localhost:8080
Hello from Docker
bash-5.1$ 
```
---

### 19. How to debug a container that exits immediately?
**Answer**

- Run interactively to observe behavior:  
  ```bash
  docker run -it --entrypoint sh image_name
  ```
- Check exit code: `docker ps -a` and inspect the STATUS/EXIT CODE.  
- Inspect logs: `docker logs <container_id>`  
- Confirm entrypoint/CMD syntax and file existence.  
- Check resource constraints and health checks causing restarts.

---

### Lab Scenario: You joined a DevOps team. A CI pipeline builds three microservice containers:
  - auth-service
  - report-service
  - analytics-service

	*All three containers exit immediately after deployment. Your task is to identify the root cause for each.*

#### SERVICE 1 — auth-service (❌ Failure: Missing ENTRYPOINT File)

`Interview Gold Answer`:

  - he container failed in the Created state because the ENTRYPOINT script lacked execute permissions.
  - Docker found the file but couldn’t execute it.
  - I fixed it by adding a shebang and executable permissions, and enforced it in the Dockerfile using chmod.
  
```sh
bash-5.1$ ls

# --------------- Create a sample authentication file ---------------

bash-5.1$ touch start.sh
bash-5.1$ vi start.sh
bash-5.1$ cat start.sh 
#!/bin/sh
echo "Auth service started"
sleep infinity

# --------------- Create a Dockerfile for authentication service ---------------

bash-5.1$ sudo vi Dockerfile.auth
bash-5.1$ ls
Dockerfile.auth  start.sh
bash-5.1$ cat Dockerfile.auth 
FROM alpine
WORKDIR /app
COPY start.sh /app/start.sh
ENTRYPOINT ["/app/start.sh"]

# --------------- Build a Docker image using the Dockerfile.auth ---------------

bash-5.1$ docker build -t auth-service -f Dockerfile.auth .
Sending build context to Docker daemon  3.584kB
Step 1/4 : FROM alpine
 ---> e7b39c54cdec
Step 2/4 : WORKDIR /app
 ---> Using cache
 ---> 1360888528f4
Step 3/4 : COPY start.sh /app/start.sh
 ---> f2a1225d0f0d
Step 4/4 : ENTRYPOINT ["/app/start.sh"]
 ---> Running in fdbd5e1acd2a
Removing intermediate container fdbd5e1acd2a
 ---> 4213f3e7f96e
Successfully built 4213f3e7f96e
Successfully tagged auth-service:latest

bash-5.1$ docker images
REPOSITORY     TAG       IMAGE ID       CREATED         SIZE
auth-service   latest    4213f3e7f96e   6 seconds ago   8.44MB
alpine         latest    e7b39c54cdec   3 weeks ago     8.44MB

# --------------- Run the container using the auth-service image ---------------

bash-5.1$ docker run --name auth auth-service
docker: Error response from daemon: failed to create shim: OCI runtime create failed: container_linux.go:380: starting container process caused: exec: "/app/start.sh": permission denied: unknown.
ERRO[0000] error waiting for container: context canceled 

bash-5.1$ docker run --name auth auth-service
docker: Error response from daemon: Conflict. The container name "/auth" is already in use by container "e5f6dd7e9d5923ba43bc2cbc4e28f858458760b5e1957487fee016b850b3c56c". You have to remove (or rename) that container to be able to reuse that name.
See 'docker run --help'.

bash-5.1$ docker ps -a
CONTAINER ID   IMAGE          COMMAND           CREATED         STATUS    PORTS     NAMES
e5f6dd7e9d59   auth-service   "/app/start.sh"   2 minutes ago   Created             auth

bash-5.1$ docker stop auth
auth
bash-5.1$ docker rm auth
auth
bash-5.1$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

bash-5.1$ ls
Dockerfile.auth  start.sh

bash-5.1$ docker run --name auth auth-service
docker: Error response from daemon: failed to create shim: OCI runtime create failed: container_linux.go:380: starting container process caused: exec: "/app/start.sh": permission denied: unknown.
ERRO[0000] error waiting for container: context canceled 

# --------------- Issue: File exists but its not executable ---------------

# Make the file executable

bash-5.1$ chmod +x start.sh
bash-5.1$ ls -l start.sh
-rwxr-xr-x    1 user     user            54 Jan  8 09:51 start.sh

# --------------- Ensures permissions are correct inside the image, even if someone forgets chmod locally. ---------------

bash-5.1$ vi Dockerfile.auth 
bash-5.1$ sudo vi Dockerfile.auth 
bash-5.1$ cat Dockerfile.auth 
FROM alpine
WORKDIR /app
COPY start.sh /app/start.sh
RUN chmod +x /app/start.sh
ENTRYPOINT ["/app/start.sh"]

# --------------- Rebuild the authentication image ---------------

bash-5.1$ docker build -t auth-service -f Dockerfile.auth .
Sending build context to Docker daemon  4.096kB
Step 1/5 : FROM alpine
 ---> e7b39c54cdec
Step 2/5 : WORKDIR /app
 ---> Using cache
 ---> 1360888528f4
Step 3/5 : COPY start.sh /app/start.sh
 ---> c132f02ce1da
Step 4/5 : RUN chmod +x /app/start.sh
 ---> Running in 37e0e4e6a958
Removing intermediate container 37e0e4e6a958
 ---> c27923af4440
Step 5/5 : ENTRYPOINT ["/app/start.sh"]
 ---> Running in 8f2c82a3d792
Removing intermediate container 8f2c82a3d792
 ---> 6398ddf88009
Successfully built 6398ddf88009
Successfully tagged auth-service:latest

# --------------- Start a container and Verify ---------------

bash-5.1$ docker run -d --name auth auth-service
b72bfd487d66a4b17490247b0bec7d31006ad739dae2ed62a18be3798fd4fbbc

bash-5.1$ docker ps
CONTAINER ID   IMAGE          COMMAND           CREATED         STATUS         PORTS     NAMES
b72bfd487d66   auth-service   "/app/start.sh"   9 seconds ago   Up 8 seconds             auth

# --------------- Check the logs whether the app is correctly working or not ---------------

bash-5.1$ docker logs auth
Auth service started
bash-5.1$ 
```

#### SERVICE 2 — report-service (❌ Failure: CMD Completes Normally (Exit 0))

`Interview Gold Answer`:

  - “The container for report-service exited immediately with exit code 0, which indicates a normal, successful termination. 
  - In Docker, a container’s lifecycle is tied to its main process. Here, the CMD executes echo 'Report service started', which runs successfully and then exits.
  - Since there is no long-running foreground process, the container stops immediately.
  - To keep a container running for services, we should replace short-lived commands with a long-running foreground process, such as the actual application process or a placeholder like sleep infinity.

```sh
# --------------- Create a Dockerfile named report ---------------

bash-5.1$ sudo vi Dockerfile.report
bash-5.1$ cat Dockerfile.report 
FROM alpine
CMD ["echo", "Report service started"]

# --------------- Build the image using that Dockerfile ---------------

bash-5.1$ docker build -t report-service -f Dockerfile.report .
Sending build context to Docker daemon   5.12kB
Step 1/2 : FROM alpine
 ---> e7b39c54cdec
Step 2/2 : CMD ["echo", "Report service started"]
 ---> Running in fa4f95f57d58
Removing intermediate container fa4f95f57d58
 ---> 04476b1f81cf
Successfully built 04476b1f81cf
Successfully tagged report-service:latest

# --------------- Run a container for report service ---------------

bash-5.1$ docker run --name report report-service
Report service started

# --------------- Verify container status ---------------

bash-5.1$ docker ps -a
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS                      PORTS     NAMES
88c991ff55c3   report-service   "echo 'Report servic…"   13 seconds ago   Exited (0) 12 seconds ago             report
b72bfd487d66   auth-service     "/app/start.sh"          24 minutes ago   Up 24 minutes                         auth

# --------------- Container exits with Exit status (0) ---------------

# --------------- Check the container logs ---------------

bash-5.1$ docker logs report
Report service started

# --------------- Inspect the container for the CMD/ENTRYPOINT instructions ---------------

bash-5.1$ docker inspect report | grep Cmd
            "Cmd": [

# Root Cause: the container exits because of 
# > Main process (echo) finished
# > No foreground process → container exited


# --------------- Fix: Update the Dockerfile with the Real-process that runs without exits ---------------

bash-5.1$ sudo vi Dockerfile.report 
bash-5.1$ cat Dockerfile.report 
FROM alpine
CMD ["sleep", "infinity"]

# --------------- Rebuild the image for the report service ---------------

bash-5.1$ docker build -t report-service -f Dockerfile.report .
Sending build context to Docker daemon   5.12kB
Step 1/2 : FROM alpine
 ---> e7b39c54cdec
Step 2/2 : CMD ["sleep", "infinity"]
 ---> Running in 2c966edd54a9
Removing intermediate container 2c966edd54a9
 ---> 6c1d4d2f67e5
Successfully built 6c1d4d2f67e5
Successfully tagged report-service:latest

bash-5.1$ docker rm report
report

# --------------- Start a new container for the report service ---------------

bash-5.1$ docker run -d --name report report-service
2ab4729d0692e76f55c39371098c79ef2e091a6f03733e4c1cad9a8c880a23b6

# --------------- Verify the container status ---------------

bash-5.1$ docker ps 
CONTAINER ID   IMAGE            COMMAND            CREATED         STATUS         PORTS     NAMES
2ab4729d0692   report-service   "sleep infinity"   4 seconds ago   Up 3 seconds             report

bash-5.1$ docker logs report
bash-5.1$ 

```
#### SERVICE 3 — analytics-service (❌ Failure: OOM Kill (Exit 137))

`Interview Gold Answer`:

  - “The container exited with code 137. This is a SIGKILL, usually caused by exceeding memory limits.
  - I verified the limit using docker inspect and can reproduce the failure interactively.
  - To fix it, either increase memory for the container or optimize the app’s memory usage.”

```sh

# --------------- Run a analytics container with memory=20m ---------------

bash-5.1$ docker run \
  --name analytics \
  --memory=20m \
  python:3.10 \
  python -c "a = ' ' * 200_000_000"
Unable to find image 'python:3.10' locally
3.10: Pulling from library/python
281b80c799de: Pull complete 
15f14138abe4: Pull complete 
378c64c44580: Pull complete 
02e37abc533a: Pull complete 
26be60fca407: Pull complete 
e380570fb967: Pull complete 
a080cbb336ed: Pull complete 
Digest: sha256:bcc149dce8e0b1bf879ad69c253ef39351f1a4b8ced55a5ce594cec159aad65a
Status: Downloaded newer image for python:3.10

# --------------- Check the container status ---------------

bash-5.1$ docker ps -a
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS                       PORTS     NAMES
d3ea75efef83   python:3.10      "python -c 'a = ' ' …"   8 seconds ago    Exited (137) 6 seconds ago             analytics
2ab4729d0692   report-service   "sleep infinity"         14 minutes ago   Up 14 minutes                          report

# observation: the container exits with code 137 which means the container is ran out of memory
# For detailed observation, inspect the container for both OOMKilled status and the Memory

bash-5.1$ docker inspect analytics | grep -i oom
            "OOMKilled": true,
            "OomScoreAdj": 0,
            "OomKillDisable": null,

bash-5.1$ docker inspect analytics | grep -i Memory
            "Memory": 20971520,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 41943040,
            "MemorySwappiness": null,

# --------------- Run interactively to observe ---------------

bash-5.1$ docker run -it --memory=20m python:3.10 sh
# python -c "a = ' ' * 200_000_000"
Killed
# exit

# --------------- Remove the exited/failed container ---------------

bash-5.1$ docker run --name analytics --memory=200m python:3.10 python -c "a = ' ' * 200_000_000"
docker: Error response from daemon: Conflict. The container name "/analytics" is already in use by container "d3ea75efef83456d8072d2309d7dbfffd7b4910dfda015581e2054e574db8357". You have to remove (or rename) that container to be able to reuse that name.
See 'docker run --help'.
bash-5.1$ docker rm analytics
analytics

# --------------- Fix / Avoid OOM by increasing memory for a new analytics container ---------------

bash-5.1$ docker run --name analytics --memory=200m python:3.10 python -c "a = ' ' * 200_000_000"
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS                        PORTS     NAMES
890fcad6c65a   python:3.10      "python -c 'a = ' ' …"   5 seconds ago    Exited (0) 4 seconds ago                analytics
2ab4729d0692   report-service   "sleep infinity"         16 minutes ago   Up 16 minutes                           report
bash-5.1$ 

# --------------- Now the container is not having OOMKilled issue of Memory ---------------

```
---

### 20. Docker build is failing due to space issues. How to resolve?
**Answer**

- Clean up unused Docker resources: `docker system prune` (be cautious — can remove stopped containers, networks, images).  
- Remove dangling images: `docker image prune` or `docker image prune -a` for all unused.  
- Check disk usage where Docker stores data (Docker root dir).  
- Use smaller base images and multi-stage builds to reduce image sizes.

```sh
bash-5.1$ mkdir docker-space-lab
bash-5.1$ cd docker-space-lab

# --------------- Create Dockerfiles that generate large images ---------------

bash-5.1$ sudo vi Dockerfile.large1
bash-5.1$ sudo vi Dockerfile.large2
bash-5.1$ ls

Dockerfile.large1  Dockerfile.large2
bash-5.1$ cat Dockerfile.large1
FROM alpine
RUN dd if=/dev/zero of=/largefile1 bs=1M count=50
CMD ["sleep", "infinity"]

bash-5.1$ cat Dockerfile.large2
FROM alpine
RUN dd if=/dev/zero of=/largefile2 bs=1M count=50
CMD ["sleep", "infinity"]

# --------------- Build multiple images ---------------

bash-5.1$ docker build -t large-image1 -f Dockerfile.large1 .
docker build -t large-image2 -f Dockerfile.large2 .

Sending build context to Docker daemon  3.072kB
Step 1/3 : FROM alpine
latest: Pulling from library/alpine
1074353eec0d: Pull complete 
Digest: sha256:865b95f46d98cf867a156fe4a135ad3fe50d2056aa3f25ed31662dff6da4eb62
Status: Downloaded newer image for alpine:latest
 ---> e7b39c54cdec
Step 2/3 : RUN dd if=/dev/zero of=/largefile1 bs=1M count=50
 ---> Running in 207fa4c1b0e3
50+0 records in
50+0 records out
52428800 bytes (50.0MB) copied, 0.051057 seconds, 979.3MB/s
Removing intermediate container 207fa4c1b0e3
 ---> e884c72bfc85
Step 3/3 : CMD ["sleep", "infinity"]
 ---> Running in bd500131afc0
Removing intermediate container bd500131afc0
 ---> 75ad6b8fc252
Successfully built 75ad6b8fc252
Successfully tagged large-image1:latest

Sending build context to Docker daemon  3.072kB
Step 1/3 : FROM alpine
 ---> e7b39c54cdec
Step 2/3 : RUN dd if=/dev/zero of=/largefile2 bs=1M count=50
 ---> Running in 5d703781acea
50+0 records in
50+0 records out
52428800 bytes (50.0MB) copied, 0.051295 seconds, 974.8MB/s
Removing intermediate container 5d703781acea
 ---> 954ea813abd9
Step 3/3 : CMD ["sleep", "infinity"]
 ---> Running in f94983e7da7f
Removing intermediate container f94983e7da7f
 ---> 8e3e4332973b
Successfully built 8e3e4332973b
Successfully tagged large-image2:latest

# --------------- Run containers ---------------

bash-5.1$ docker run -d --name container1 large-image1
docker run -d --name container2 large-image2
28401a841850a63f62e3d46fb6b44ef563719bf0805ececc59b706ebe34cf64b
684760a84a58d6956b1573d7d240f6a318359a833210555e985ee10e06f028fa

# --------------- Check Docker disk usage ---------------

bash-5.1$ docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          3         2         113.3MB   8.437MB (7%)
Containers      2         2         0B        0B
Local Volumes   0         0         0B        0B
Build Cache     0         0         0B        0B

bash-5.1$ docker build -t another-image -f Dockerfile.large1 .
Sending build context to Docker daemon  3.072kB
Step 1/3 : FROM alpine
 ---> e7b39c54cdec
Step 2/3 : RUN dd if=/dev/zero of=/largefile1 bs=1M count=50
 ---> Using cache
 ---> e884c72bfc85
Step 3/3 : CMD ["sleep", "infinity"]
 ---> Using cache
 ---> 75ad6b8fc252
Successfully built 75ad6b8fc252
Successfully tagged another-image:latest`

# --------------- Simulate disk full error ---------------

# --------------- Cleanup Docker resources ---------------

bash-5.1$ docker image prune
WARNING! This will remove all dangling images.
Are you sure you want to continue? [y/N] y
Total reclaimed space: 0B

bash-5.1$ docker image prune -a
WARNING! This will remove all images without at least one container associated to them.
Are you sure you want to continue? [y/N] y
Deleted Images:
untagged: alpine:latest
untagged: alpine@sha256:865b95f46d98cf867a156fe4a135ad3fe50d2056aa3f25ed31662dff6da4eb62
untagged: another-image:latest

Total reclaimed space: 0B

bash-5.1$ docker system prune -a
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all images without at least one container associated to them
  - all build cache

Are you sure you want to continue? [y/N] y
Total reclaimed space: 0B

# --------------- Verify space freed ---------------

bash-5.1$ docker system df
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          2         2         113.3MB   8.437MB (7%)
Containers      2         2         0B        0B
Local Volumes   0         0         0B        0B
Build Cache     0         0         0B        0B
bash-5.1$ 
```

---

### 21. How to monitor Docker container performance?
**Answer**

- `docker stats` for real-time metrics per container.  
- `docker top <container>` to see running processes inside a container.  
- Centralized logging and monitoring: ELK/EFK stack, Prometheus + cAdvisor, Datadog, Sysdig.  
- Use resource limits and alerts in monitoring tooling.

```sh

bash-5.1$ docker stats
CONTAINER ID   NAME         CPU %     MEM USAGE / LIMIT     MEM %     NET I/O       BLOCK I/O     PIDS
84c57098d95b   nginx-cont   0.00%     3.324MiB / 3.697GiB   0.09%     1.18kB / 0B   0B / 12.3kB   3
^Z
[1]+  Stopped                 docker stats

bash-5.1$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                   NAMES
84c57098d95b   nginx     "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp   nginx-cont

bash-5.1$ docker top nginx-cont
PID                 USER                TIME                COMMAND
700                 root                0:00                nginx: master process nginx -g daemon off;
819                 101                 0:00                nginx: worker process
820                 101                 0:00                nginx: worker process
```

---

### 22. How to backup and restore Docker containers?
**Answer**

**Backup**
```bash
docker commit container_name backup_image
docker save backup_image > backup.tar
```

**Restore**
```bash
docker load < backup.tar
docker run backup_image
```

For persistent data, backup volumes (use `docker run --volumes-from` or use volume plugin snapshots).

---

## Category 7: Advanced Concepts

### 23. What is Docker Swarm and how does it compare to Kubernetes?
**Answer**

- **Docker Swarm:** Docker's native orchestration solution; simpler to set up and use.  
- **Kubernetes:** More feature-rich and complex; larger ecosystem and extensibility.

Comparison:
- **Setup:** Swarm is easier; Kubernetes is more complex.  
- **Features & Ecosystem:** Kubernetes has more features and wide ecosystem support.  
- **Scaling & Production Use:** Kubernetes is more commonly used for large-scale production environments.

```sh
bash-5.1$ docker swarm init
Swarm initialized: current node (u8p4v12na6nt6axn3nkfy2dvt) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-2pc8yndyb03bcdkjsz0md1e0g7zqzepnmn72mxeowy02ekczez-1bycft9rn3sgcttrogw6cstqa 172.20.0.2:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

bash-5.1$ docker service ls
ID        NAME      MODE      REPLICAS   IMAGE     PORTS

# --------------- Create a docker service with replicas '1' ---------------

bash-5.1$ docker service create --name=nginx-cont --replicas 1 nginx
htelkqempdxr7yaztpo9gktwl
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 

bash-5.1$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                   NAMES
2a3b5b5a4d07   nginx:latest   "/docker-entrypoint.…"   27 seconds ago   Up 26 seconds   80/tcp                                  nginx-cont.1.98p5ptndwrwbn1b8b80ofhgmg
84c57098d95b   nginx          "/docker-entrypoint.…"   7 minutes ago    Up 7 minutes    0.0.0.0:8080->80/tcp, :::8080->80/tcp   nginx-cont

# --------------- Scaling of docker serive of nginx-cont from '1' replica to '3' replicas ---------------

bash-5.1$ docker service scale nginx-cont=3
nginx-cont scaled to 3
overall progress: 3 out of 3 tasks 
1/3: running   [==================================================>] 
2/3: running   [==================================================>] 
3/3: running   [==================================================>] 
verify: Service converged 

bash-5.1$ docker service ls
ID             NAME         MODE         REPLICAS   IMAGE          PORTS
htelkqempdxr   nginx-cont   replicated   3/3        nginx:latest   
bash-5.1$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS              PORTS                                   NAMES
d7ba24065dfa   nginx:latest   "/docker-entrypoint.…"   19 seconds ago       Up 17 seconds       80/tcp                                  nginx-cont.3.cwszsns3gmojd50wa90p0y6ks
3029246ca05b   nginx:latest   "/docker-entrypoint.…"   19 seconds ago       Up 17 seconds       80/tcp                                  nginx-cont.2.ej45jof3wl76vjak50l68eyj5
2a3b5b5a4d07   nginx:latest   "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp                                  nginx-cont.1.98p5ptndwrwbn1b8b80ofhgmg
84c57098d95b   nginx          "/docker-entrypoint.…"   7 minutes ago        Up 7 minutes        0.0.0.0:8080->80/tcp, :::8080->80/tcp   nginx-cont
bash-5.1$ 
```
---

### 24. Explain Docker build cache and how it works.
**Answer**

Docker caches layers created by build instructions. If a build instruction and its context haven't changed, Docker reuses the cached layer. Cache is invalidated when:
- The instruction changes.  
- The base image changes.  
- Files referenced by `COPY`/`ADD` change (checksum differences).

#### Project dockerfile 

```dockerfile
# Stage 1: Build Python dependencies
FROM python:3.10-alpine AS build
WORKDIR /api/app

# Install build dependencies for psycopg2
RUN apk add --no-cache gcc musl-dev postgresql-dev

# Copy requirements and install dependencies
COPY requirements.txt ./
RUN pip install --user --no-cache-dir -r requirements.txt \
    && find /root/.local -name '*.pyc' -delete \
    && find /root/.local -name '__pycache__' -delete

# Stage 2: Main application image
FROM python:3.10-alpine AS main
WORKDIR /api

# Install PostgreSQL client for pg_isready
RUN apk add --no-cache postgresql-client

# Copy installed Python packages from build stage
COPY --from=build /root/.local /root/.local

# Copy the entire app folder
COPY . ./app

# Add pip binaries to PATH
ENV PATH=/root/.local/bin:$PATH
ENV FLASK_APP=app/wsgi.py

# Expose port
EXPOSE 5000

# Start Gunicorn server
CMD ["gunicorn", "app.wsgi:app"]
```
---

### 25. What are .dockerignore files and why use them?
**Answer**

Similar to `.gitignore` — `.dockerignore` excludes files from the build context sent to the daemon.

Benefits:
- Smaller build context → faster builds.  
- Smaller images (fewer unnecessary files).  
- Avoid leaking sensitive files into builds.

#### Project .dockerignore file
```sh
# ignore virtual env files into git
/venv/*

# ignore environment variable files
.env.dev
.env.prod

# ignore testing environment variable files
.env.test

# ignore Python bytecode files
app/__pycache__/*
__pycache__/
.pytest_cache
*.pyc

# ignore log files
*.log
```
---

### 26. How do you handle application configuration in Docker?
**Answer**

- Use environment variables for simple configuration.  
- Mount configuration files as volumes for complex configs or local development.  
- Use Docker Configs (in Swarm mode) for distributing config data.  
- Use external configuration services (Consul, etcd, AWS Parameter Store).

---

### 27. What is the difference between ADD and COPY?
**Answer**

- **COPY:** Simple and predictable — copies files and directories from the build context into the image.  
- **ADD:** Like `COPY` but also supports extracting local tar archives into the image and downloading files from URLs.  

**Best Practice:** Use `COPY` unless you specifically need `ADD`'s extra features.

---

### 28. How to reduce Docker image build time?
**Answer**

- Leverage build cache (order Dockerfile layers to maximize cache hits).  
- Use BuildKit for faster, parallel builds.  
- Minimize context size with `.dockerignore`.  
- Use smaller base images and multi-stage builds.  
- Combine `RUN` instructions where appropriate.

---

### 29. What are Docker health checks?
**Answer**

Health checks are configured in the Dockerfile to let Docker determine whether an application in the container is healthy.

Example:
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```

---

### 30. How do you update running containers without downtime?
**Answer**

- Build a new image version.  
- Use an orchestration platform (Docker Swarm, Kubernetes) to perform rolling updates.  
- Use blue-green or canary deployment strategies with a load balancer and health checks to switch traffic gradually.

