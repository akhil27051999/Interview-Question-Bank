# Docker — Q&A (DevOps Interview Prep)

## Table of Contents (Docker)
- [Category 1: Docker Fundamentals & Architecture](#category-1-docker-fundamentals--architecture)
- [Category 2: Dockerfile & Image Management](#category-2-dockerfile--image-management)
- [Category 3: Storage & Volumes](#category-3-storage--volumes)
- [Category 4: Networking](#category-4-networking)
- [Category 5: Security & Best Practices](#category-5-security--best-practices)
- [Category 6: Troubleshooting & Real-time Scenarios](#category-6-troubleshooting--real-time-scenarios)
- [Category 7: Advanced Concepts](#category-7-advanced-concepts)

---

## Category 1: Docker Fundamentals & Architecture

### 1. What is Docker and how does it differ from traditional virtualization?
**Answer**

> Docker is a platform for developing, shipping, and running applications in containers.

Key Differences:
- **Virtualization:** Runs a complete OS on a hypervisor; higher resource overhead.  
- **Containers:** Share the host OS kernel, run isolated processes; lightweight and fast.  
- **Startup Time:** VMs take minutes; containers start in seconds.  
- **Resource Usage:** VMs often require GBs of memory; containers can work in MBs.

---

### 2. Explain Docker architecture and its main components.
**Answer**

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
# CONTAINER CREATED

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

# CONTAINER RUNNING 
# COMMAND: docker start <cont_id/cont_name>

bash-5.1$ docker start nginx-cont
nginx-cont
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS         PORTS     NAMES
32f3cdffb2a0   nginx     "/docker-entrypoint.…"   21 seconds ago   Up 2 seconds   80/tcp    nginx-cont

# NOTE : Also use command 'docker run -d --name <cont_name> nginx' to start/run new container which is not created before

# CONTAINER PAUSE/UNPAUSE

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

# CONTAINER STOP
# COMMAND: docker stop <cont-name> / docker kill <cont-name>

bash-5.1$ docker stop nginx-cont
nginx-cont
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS                     PORTS     NAMES
32f3cdffb2a0   nginx     "/docker-entrypoint.…"   About a minute ago   Exited (0) 2 seconds ago             nginx-cont

# CONTAINER RESTARTING

bash-5.1$ docker run -d --restart=on-failure --name nginx-cont nginx
docker: Error response from daemon: Conflict. The container name "/nginx-cont" is already in use by container "32f3cdffb2a0a866f91761750792449e5ef4c3bfcec7f095517c5d93fcddd7ad". You have to remove (or rename) that container to be able to reuse that name.
See 'docker run --help'.
bash-5.1$ docker rm nginx-cont
nginx-cont
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES


# > RESTART POLICY: ON FAILURE

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

# > RESTART POLICY: ALWAYS

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

# Step 1: Create a Docker Volume

bash-5.1$ docker volume create mydbdata
mydbdata

bash-5.1$ docker volume ls
DRIVER    VOLUME NAME
local     mydbdata

# Step 2: Run MySQL Container Using the Volume

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

# Step 3: Connect to MySQL and Add Data

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

# Step 4: Stop and Remove the Container

bash-5.1$ docker stop mysql-db
mysql-db
bash-5.1$ docker rm mysql-db
mysql-db

# Step 5: Recreate Container Using the Same Volume

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

# Step 6: Share the same volume between 2 containers

bash-5.1$ docker run -d --name mysql-lab3 -v mydbdata:/var/lib/mysql mysql:8
1a8707e864e9a1648a6d3d1786104dc630111cb590b43b8425e70979ae38ca1f

# Both containers see the same data.

# Step 7: Test backup & restore
bash-5.1$ docker run --rm -v mydbdata:/data busybox tar cvf /backup.tar /data
```
---

#### Bind Mounts 

```sh
# Step 1: Create a Host Directory

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

# Step 2: Run MySQL Using Bind Mount

bash-5.1$ docker run -d --name mysql-db -e MYSQL_ROOT_PASSWORD=mypasswd -e MYSQL_DATABASE=testdb -v ~/mysql-data:/var/lib/mysql mysql:8
6b016784f88acabaae09f27eb9c3473368f1a87cb83b69c487f06938cab9574c
bash-5.1$ docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS          PORTS                 NAMES
6b016784f88a   mysql:8   "docker-entrypoint.s…"   About a minute ago   Up 59 seconds   3306/tcp, 33060/tcp   mysql-db

# Step 3: Add Sample Data

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

# Step 4: Stop and Remove Container

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

### Category 4: Networking

#### 12. Explain Docker network types.
**Answer**

- **bridge:** Default network for standalone containers on a host. Containers on the same bridge can communicate.  
- **host:** Container shares the host network namespace (no network isolation).  
- **overlay:** Connects containers across multiple Docker daemons (used by Swarm).  
- **macvlan:** Assigns a MAC address to containers, making them look like physical devices on the network.  
- **none:** Container has no network interfaces.

---

#### 13. How do containers communicate with each other?
**Answer**

- On the same bridge network: use container name as hostname (Docker DNS).  
- On custom user-defined networks: DNS-based service discovery is available and is recommended.  
- Using published ports: access containers from the host via mapped ports.  
- Legacy: links (deprecated) — avoid using links.

---

#### 14. What is Docker Compose and when to use it?
**Answer**

Docker Compose lets you define and run multi-container applications using a YAML file (`docker-compose.yml`). Use it for:
- Local development and testing.  
- Defining simple multi-service stacks.  
- Orchestrating containers for small deployments (not a replacement for full orchestration platforms like Kubernetes).

---

### Category 5: Security & Best Practices

#### 15. What are Docker security best practices?
**Answer**

- Run containers as a non-root user where possible.  
- Use minimal and trusted base images.  
- Scan images for vulnerabilities (Trivy, Clair, Snyk, Docker Scout).  
- Pin base image versions (avoid `latest` in production).  
- Implement resource limits and runtime constraints.  
- Keep Docker daemon and host OS patched and up to date.  
- Use secret management and avoid baking secrets into images.

---

#### 16. How do you limit container resources?
**Answer**

Example:
```bash
docker run --memory=512m --cpus=1.5 --pids-limit=100 my-app
```

- `--memory` (or `-m`): memory limit.  
- `--cpus`: limit CPU quota.  
- `--pids-limit`: limit number of processes.

---

#### 17. What is container scanning and why is it important?
**Answer**

Container scanning analyzes images to find known vulnerabilities in packages and OS components.

Tools: **Trivy**, **Clair**, **Docker Scout**, **Snyk**.  
Importance: Discover and remediate vulnerabilities before deploying images to production.

---

### Category 6: Troubleshooting & Real-time Scenarios

#### 18. Container is running but application isn't accessible. How to debug?
**Answer**

- Check container logs: `docker logs <container_name>`  
- Verify port mapping: `docker ps` and check `-p`/`--publish` flags.  
- Enter container: `docker exec -it <container_name> sh` (or bash) to test internal connectivity and services.  
- Check application configuration and listen address (e.g., `0.0.0.0` vs `localhost`).  
- Verify firewall and network settings on the host.

---

#### 19. How to debug a container that exits immediately?
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

#### 20. Docker build is failing due to space issues. How to resolve?
**Answer**

- Clean up unused Docker resources: `docker system prune` (be cautious — can remove stopped containers, networks, images).  
- Remove dangling images: `docker image prune` or `docker image prune -a` for all unused.  
- Check disk usage where Docker stores data (Docker root dir).  
- Use smaller base images and multi-stage builds to reduce image sizes.

---

#### 21. How to monitor Docker container performance?
**Answer**

- `docker stats` for real-time metrics per container.  
- `docker top <container>` to see running processes inside a container.  
- Centralized logging and monitoring: ELK/EFK stack, Prometheus + cAdvisor, Datadog, Sysdig.  
- Use resource limits and alerts in monitoring tooling.

---

#### 22. How to backup and restore Docker containers?
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

### Category 7: Advanced Concepts

#### 23. What is Docker Swarm and how does it compare to Kubernetes?
**Answer**

- **Docker Swarm:** Docker's native orchestration solution; simpler to set up and use.  
- **Kubernetes:** More feature-rich and complex; larger ecosystem and extensibility.

Comparison:
- **Setup:** Swarm is easier; Kubernetes is more complex.  
- **Features & Ecosystem:** Kubernetes has more features and wide ecosystem support.  
- **Scaling & Production Use:** Kubernetes is more commonly used for large-scale production environments.

---

#### 24. Explain Docker build cache and how it works.
**Answer**

Docker caches layers created by build instructions. If a build instruction and its context haven't changed, Docker reuses the cached layer. Cache is invalidated when:
- The instruction changes.  
- The base image changes.  
- Files referenced by `COPY`/`ADD` change (checksum differences).

---

#### 25. What are .dockerignore files and why use them?
**Answer**

Similar to `.gitignore` — `.dockerignore` excludes files from the build context sent to the daemon.

Benefits:
- Smaller build context → faster builds.  
- Smaller images (fewer unnecessary files).  
- Avoid leaking sensitive files into builds.

---

#### 26. How do you handle application configuration in Docker?
**Answer**

- Use environment variables for simple configuration.  
- Mount configuration files as volumes for complex configs or local development.  
- Use Docker Configs (in Swarm mode) for distributing config data.  
- Use external configuration services (Consul, etcd, AWS Parameter Store).

---

#### 27. What is the difference between ADD and COPY?
**Answer**

- **COPY:** Simple and predictable — copies files and directories from the build context into the image.  
- **ADD:** Like `COPY` but also supports extracting local tar archives into the image and downloading files from URLs.  

**Best Practice:** Use `COPY` unless you specifically need `ADD`'s extra features.

---

#### 28. How to reduce Docker image build time?
**Answer**

- Leverage build cache (order Dockerfile layers to maximize cache hits).  
- Use BuildKit for faster, parallel builds.  
- Minimize context size with `.dockerignore`.  
- Use smaller base images and multi-stage builds.  
- Combine `RUN` instructions where appropriate.

---

#### 29. What are Docker health checks?
**Answer**

Health checks are configured in the Dockerfile to let Docker determine whether an application in the container is healthy.

Example:
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```

---

#### 30. How do you update running containers without downtime?
**Answer**

- Build a new image version.  
- Use an orchestration platform (Docker Swarm, Kubernetes) to perform rolling updates.  
- Use blue-green or canary deployment strategies with a load balancer and health checks to switch traffic gradually.

