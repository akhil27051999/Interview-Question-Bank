# 🐳 Docker Commands Cheat Sheet with Explanations

## 🔧 Docker Installation & Setup

| Command | Description |
|--------|-------------|
| `docker --version` | Check the installed Docker version. |
| `docker info` | Display detailed info about Docker environment (images, containers, drivers). |
| `docker login` | Authenticate to a Docker registry (e.g., Docker Hub). |


## 📦 Working with Docker Images

| Command | Description |
|--------|-------------|
| `docker images` | List all locally available Docker images. |
| `docker pull <image>` | Download an image from Docker Hub or a registry. |
| `docker build -t <name>:<tag> .` | Build an image from a Dockerfile in the current directory. |
| `docker tag <image> <repo>/<image>:<tag>` | Add a custom name/tag to an image. |
| `docker push <repo>/<image>:<tag>` | Upload an image to Docker Hub or a private registry. |
| `docker rmi <image>` | Remove a Docker image from your system. |


## 🚀 Working with Containers

| Command | Description |
|--------|-------------|
| `docker ps` | List running containers. |
| `docker ps -a` | List all containers (running + stopped). |
| `docker run <image>` | Run a container from an image. |
| `docker run -it <image> /bin/bash` | Start a container with interactive terminal. |
| `docker run -d <image>` | Run a container in detached mode (in background). |
| `docker run --name <name> <image>` | Assign a custom name to the container. |
| `docker exec -it <container> /bin/bash` | Access a running container shell. |
| `docker start <container>` | Start a stopped container. |
| `docker stop <container>` | Stop a running container. |
| `docker restart <container>` | Restart a container. |
| `docker rm <container>` | Remove a container. |
| `docker rm -f <container>` | Force remove a running container. |


## 🗂️ Volumes & Data Management

| Command | Description |
|--------|-------------|
| `docker volume create <volume>` | Create a Docker volume for persistent data. |
| `docker volume ls` | List all volumes. |
| `docker volume inspect <volume>` | Show detailed info about a volume. |
| `docker run -v <volume>:/path/in/container <image>` | Mount a volume inside a container. |
| `docker volume rm <volume>` | Remove a volume. |


## 🌐 Docker Networking

| Command | Description |
|--------|-------------|
| `docker network ls` | List all Docker networks. |
| `docker network create <network>` | Create a custom network. |
| `docker network inspect <network>` | Inspect details of a network. |
| `docker network connect <network> <container>` | Connect a container to a network. |
| `docker run --network=<network> <image>` | Start container attached to a specific network. |


## 🛠️ Dockerfile & Image Customization

| Dockerfile Command | Purpose |
|--------------------|---------|
| `FROM` | Base image to use (e.g., `ubuntu`, `alpine`). |
| `RUN` | Execute commands while building the image. |
| `COPY` / `ADD` | Copy files from host into image. |
| `WORKDIR` | Set working directory in container. |
| `CMD` | Default command to run when container starts. |
| `ENTRYPOINT` | Configure container startup behavior. |
| `EXPOSE` | Define ports the container listens on. |
| `ENV` | Set environment variables in the image. |

## 🛠 Build Example:

```bash
docker build -t myapp:latest .
```

## 📂 Docker Compose Commands

Docker Compose allows you to define and run multi-container Docker applications using a single `docker-compose.yml` file.

| Command | Description |
|---------|-------------|
| `docker-compose up` | Start all services defined in the `docker-compose.yml`. Builds images if not present. |
| `docker-compose up -d` | Run all services in detached (background) mode. |
| `docker-compose down` | Stop and remove all running containers, networks, and volumes created by Compose. |
| `docker-compose build` | Build or rebuild services defined in `docker-compose.yml`. |
| `docker-compose ps` | List all containers managed by the current Compose setup. |


## 📊 Docker Logs & Monitoring

Monitor your running containers and services in real-time.

| Command | Description |
|---------|-------------|
| `docker logs <container>` | Display the logs of a specific container. Useful for debugging. |
| `docker stats` | Show real-time CPU, memory, network, and I/O usage for all containers. |
| `docker inspect <container or image>` | Return low-level detailed JSON info about a container or image (e.g., IP, mount points, labels). |
| `docker events` | Stream real-time events from the Docker daemon (e.g., container start/stop). |


## 🧼 Docker Cleanup Commands

Reclaim disk space by cleaning unused, dangling, or stopped Docker resources.

| Command | Description |
|---------|-------------|
| `docker system prune` | Remove all unused containers, networks, dangling images, and build cache. Interactive prompt by default. |
| `docker image prune` | Remove dangling (unreferenced) images. |
| `docker container prune` | Remove all stopped containers. |
| `docker volume prune` | Delete all unused volumes. |
| `docker builder prune` | Clean up the build cache used by Docker image builds. |

