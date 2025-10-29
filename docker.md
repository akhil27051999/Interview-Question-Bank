# DevOps Interview Preparation Guide

A comprehensive collection of real-time and technical interview questions for DevOps roles.

Technologies Covered
- Kubernetes
- Docker
- Terraform
- AWS
- CI/CD
- Monitoring
- Linux & Shell Scripting
- Networking

---

## Table of Contents
- [Docker](#docker)
  - [Category 1: Docker Fundamentals & Architecture](#category-1-docker-fundamentals--architecture)
  - [Category 2: Dockerfile & Image Management](#category-2-dockerfile--image-management)
  - [Category 3: Storage & Volumes](#category-3-storage--volumes)
  - [Category 4: Networking](#category-4-networking)
  - [Category 5: Security & Best Practices](#category-5-security--best-practices)
  - [Category 6: Troubleshooting & Real-time Scenarios](#category-6-troubleshooting--real-time-scenarios)
  - [Category 7: Advanced Concepts](#category-7-advanced-concepts)

---

## Docker

### Category 1: Docker Fundamentals & Architecture

1. **What is Docker and how does it differ from traditional virtualization?**  
Answer:  
Docker is a platform for developing, shipping, and running applications in containers.

Key Differences:
- **Virtualization**: Runs a complete OS on a hypervisor; higher resource overhead.  
- **Containers**: Share the host OS kernel, run isolated processes; lightweight and fast.  
- **Startup Time**: VMs take minutes; containers start in seconds.  
- **Resource Usage**: VMs often require GBs of memory; containers can work in MBs.

2. **Explain Docker architecture and its main components.**  
Answer:
- **Docker Daemon**: Background service that manages containers, images, networks, volumes.  
- **Docker Client**: CLI (docker) used to interact with the daemon (e.g., docker run, docker build).  
- **Docker Images**: Read-only templates used to create containers.  
- **Docker Containers**: Runnable instances of images with a writable layer.  
- **Docker Registry**: Storage and distribution for images (Docker Hub, private registries).  
- **Dockerfile**: Script with build instructions for creating images.

3. **What is the difference between a Docker image and container?**  
Answer:
- **Image**: Read-only template containing application code and dependencies; built from a Dockerfile.  
- **Container**: Runnable instance of an image; has a writable layer on top.

4. **Explain Docker container lifecycle states.**  
Answer:
- **Created**: Container object created but not started.  
- **Running**: Container is actively executing.  
- **Paused**: Process execution suspended (using cgroups freezer).  
- **Stopped**: Process terminated but metadata remains.  
- **Removed**: Container deleted from the host.

---

### Category 2: Dockerfile & Image Management

5. **What is a Dockerfile and explain common instructions?**  
Answer:  
A Dockerfile is a text file with instructions to build a Docker image.

Common Instructions:
- **FROM**: Base image.  
- **RUN**: Execute commands during image build.  
- **COPY / ADD**: Copy files from host into the image (ADD has extra features).  
- **WORKDIR**: Set working directory.  
- **EXPOSE**: Document the port the container listens on.  
- **CMD / ENTRYPOINT**: Default command to run when the container starts.

6. **What is the difference between CMD and ENTRYPOINT?**  
Answer:
- **CMD**: Default command/arguments that can be overridden at runtime (`docker run image command`).  
- **ENTRYPOINT**: The main command that always runs; CMD acts as default arguments to ENTRYPOINT.  
Best Practice: Use ENTRYPOINT for the main application and CMD for default arguments.

7. **How do you optimize Docker image size?**  
Answer:
- Use minimal base images (e.g., Alpine).  
- Combine related RUN commands to reduce the number of layers.  
- Use a .dockerignore to exclude unnecessary files from the build context.  
- Clean package manager caches in the same RUN step where packages are installed.  
- Use multi-stage builds to omit build-time dependencies from the final image.

8. **What are multi-stage builds and why are they useful?**  
Answer:  
Multi-stage builds use multiple FROM statements so you can build artifacts in one stage and copy only the final artifacts into a smaller runtime image. They reduce final image size and keep build dependencies out of the runtime image.

Example:
```dockerfile
# Build stage
FROM golang:1.19 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Final stage
FROM alpine:latest
COPY --from=builder /app/myapp /
CMD ["./myapp"]
```

---

### Category 3: Storage & Volumes

9. **What are Docker volumes and when to use them?**  
Answer:  
Volumes provide persistent storage for containers independent of the container lifecycle.

Common Use Cases:
- Database data persistence.  
- Configuration or state that must survive container recreation.  
- Sharing data between multiple containers.  
- Backups and restores.

10. **Difference between bind mounts and named volumes?**  
Answer:
- **Bind Mounts**: Map a host directory or file into the container. Dependent on the host filesystem and paths. Great for development (live code).  
- **Named Volumes**: Managed by Docker, stored in Docker's storage area, more portable across hosts (when used with volume drivers). Preferred for production.

11. **How do you manage sensitive data in Docker?**  
Answer:
- Use **Docker Secrets** in Swarm mode for sensitive information.  
- Avoid storing secrets in environment variables when possible.  
- Mount files with restricted permissions (read-only) into the container.  
- Use external secret managers (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault).  
- Ensure secrets are not baked into images or committed to source control.

---

### Category 4: Networking

12. **Explain Docker network types.**  
Answer:
- **bridge**: Default network for standalone containers on a host. Containers on the same bridge can communicate.  
- **host**: Container shares the host network namespace (no network isolation).  
- **overlay**: Connects containers across multiple Docker daemons (used by Swarm).  
- **macvlan**: Assigns a MAC address to containers, making them look like physical devices on the network.  
- **none**: Container has no network interfaces.

13. **How do containers communicate with each other?**  
Answer:
- On the same bridge network: use container name as hostname (Docker DNS).  
- On custom user-defined networks: DNS-based service discovery is available and is recommended.  
- Using published ports: access containers from the host via mapped ports.  
- Legacy: links (deprecated) — avoid using links.

14. **What is Docker Compose and when to use it?**  
Answer:  
Docker Compose lets you define and run multi-container applications using a YAML file (docker-compose.yml). Use it for:
- Local development and testing.  
- Defining simple multi-service stacks.  
- Orchestrating containers for small deployments (not a replacement for full orchestration platforms like Kubernetes).

---

### Category 5: Security & Best Practices

15. **What are Docker security best practices?**  
Answer:
- Run containers as a non-root user where possible.  
- Use minimal and trusted base images.  
- Scan images for vulnerabilities (Trivy, Clair, Snyk, Docker Scout).  
- Pin base image versions (avoid latest in production).  
- Implement resource limits and runtime constraints.  
- Keep Docker daemon and host OS patched and up to date.  
- Use secret management and avoid baking secrets into images.

16. **How do you limit container resources?**  
Answer:
Example:
```bash
docker run --memory=512m --cpus=1.5 --pids-limit=100 my-app
```
- --memory (or -m): memory limit.  
- --cpus: limit CPU quota.  
- --pids-limit: limit number of processes.

17. **What is container scanning and why is it important?**  
Answer:  
Container scanning analyzes images to find known vulnerabilities in packages and OS components.

Tools: Trivy, Clair, Docker Scout, Snyk.  
Importance: Discover and remediate vulnerabilities before deploying images to production.

---

### Category 6: Troubleshooting & Real-time Scenarios

18. **Container is running but application isn't accessible. How to debug?**  
Answer:
- Check container logs: `docker logs <container_name>`  
- Verify port mapping: `docker ps` and check -p/--publish flags.  
- Enter container: `docker exec -it <container_name> sh` (or bash) to test internal connectivity and services.  
- Check application configuration and listen address (e.g., 0.0.0.0 vs localhost).  
- Verify firewall and network settings on the host.

19. **How to debug a container that exits immediately?**  
Answer:
- Run interactively to observe behavior: `docker run -it --entrypoint sh image_name`  
- Check exit code: `docker ps -a` and inspect the STATUS/EXIT CODE.  
- Inspect logs: `docker logs <container_id>`  
- Confirm entrypoint/CMD syntax and file existence.  
- Check resource constraints and health checks causing restarts.

20. **Docker build is failing due to space issues. How to resolve?**  
Answer:
- Clean up unused Docker resources: `docker system prune` (be cautious — can remove stopped containers, networks, images).  
- Remove dangling images: `docker image prune` or `docker image prune -a` for all unused.  
- Check disk usage where Docker stores data (Docker root dir).  
- Use smaller base images and multi-stage builds to reduce image sizes.

21. **How to monitor Docker container performance?**  
Answer:
- `docker stats` for real-time metrics per container.  
- `docker top <container>` to see running processes inside a container.  
- Centralized logging and monitoring: ELK/EFK stack, Prometheus + cAdvisor, Datadog, Sysdig.  
- Use resource limits and alerts in monitoring tooling.

22. **How to backup and restore Docker containers?**  
Answer:
Backup:
```bash
docker commit container_name backup_image
docker save backup_image > backup.tar
```
Restore:
```bash
docker load < backup.tar
docker run backup_image
```
For persistent data, backup volumes (use docker run --volumes-from or use volume plugin snapshots).

---

### Category 7: Advanced Concepts

23. **What is Docker Swarm and how does it compare to Kubernetes?**  
Answer:
- **Docker Swarm**: Docker's native orchestration solution; simpler to set up and use.  
- **Kubernetes**: More feature-rich and complex; larger ecosystem and extensibility.

Comparison:
- **Setup**: Swarm is easier; Kubernetes is more complex.  
- **Features & Ecosystem**: Kubernetes has more features and wide ecosystem support.  
- **Scaling & Production Use**: Kubernetes is more commonly used for large-scale production environments.

24. **Explain Docker build cache and how it works.**  
Answer:
Docker caches layers created by build instructions. If a build instruction and its context haven't changed, Docker reuses the cached layer. Cache is invalidated when:
- The instruction changes.  
- The base image changes.  
- Files referenced by COPY/ADD change (checksum differences).

25. **What are .dockerignore files and why use them?**  
Answer:
Similar to .gitignore — .dockerignore excludes files from the build context sent to the daemon.

Benefits:
- Smaller build context → faster builds.  
- Smaller images (fewer unnecessary files).  
- Avoid leaking sensitive files into builds.

26. **How do you handle application configuration in Docker?**  
Answer:
- Use environment variables for simple configuration.  
- Mount configuration files as volumes for complex configs or local development.  
- Use Docker Configs (in Swarm mode) for distributing config data.  
- Use external configuration services (Consul, etcd, AWS Parameter Store).

27. **What is the difference between ADD and COPY?**  
Answer:
- **COPY**: Simple and predictable — copies files and directories from the build context into the image.  
- **ADD**: Like COPY but also supports extracting local tar archives into the image and downloading files from URLs.  
Best Practice: Use COPY unless you specifically need ADD's extra features.

28. **How to reduce Docker image build time?**  
Answer:
- Leverage build cache (order Dockerfile layers to maximize cache hits).  
- Use BuildKit for faster, parallel builds.  
- Minimize context size with .dockerignore.  
- Use smaller base images and multi-stage builds.  
- Combine RUN instructions where appropriate.

29. **What are Docker health checks?**  
Answer:
Health checks are configured in the Dockerfile to let Docker determine whether an application in the container is healthy.

Example:
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```

30. **How do you update running containers without downtime?**  
Answer:
- Build a new image version.  
- Use an orchestration platform (Docker Swarm, Kubernetes) to perform rolling updates.  
- Use blue-green or canary deployment strategies with a load balancer and health checks to switch traffic gradually.

---

If you'd like, I can:
- Add this Docker section into a combined README with the Kubernetes section already present.
- Split each technology into separate Markdown files.
- Generate printable flashcards, a condensed cheat-sheet, or a PDF export.

Which would you prefer next?
