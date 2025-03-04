# Docker Guide: From Basics to Advanced

## Table of Contents
- [What is Docker?](#what-is-docker)
- [Docker vs Virtual Machines](#docker-vs-virtual-machines)
- [Why Use Docker?](#why-use-docker)
- [Installing Docker on Fedora](#installing-docker-on-fedora)
- [Installing Docker on RHEL](#installing-docker-on-rhel)
- [Docker Architecture](#docker-architecture)
- [Docker Commands](#docker-commands)
- [Docker Compose](#docker-compose)
  - [Installing Docker Compose](#installing-docker-compose)
  - [Using Docker Compose](#using-docker-compose)
- [Advanced Docker Concepts](#advanced-docker-concepts)

## What is Docker?

Docker is an open-source platform that automates the deployment, scaling, and management of applications using containerization technology. Containers are lightweight, portable, and self-sufficient units that can run applications and their dependencies isolated from the host system.

Docker enables developers to package an application with all its dependencies into a standardized unit (a container) for software development and deployment. This ensures that the application works seamlessly regardless of the environment it's running in.

Key Docker components:
- **Docker Engine**: The core runtime that builds and runs containers
- **Docker Images**: Read-only templates used to create containers
- **Docker Containers**: Running instances of Docker images
- **Docker Registry**: A repository for Docker images (like Docker Hub)

## Docker vs Virtual Machines

| Feature | Docker Containers | Virtual Machines |
|---------|------------------|------------------|
| Isolation | OS-level virtualization | Hardware-level virtualization |
| Size | Lightweight (MBs) | Heavy (GBs) |
| Boot time | Seconds | Minutes |
| Performance | Near-native | Overhead due to hypervisor |
| Resource usage | Efficient, shared kernel | Resource-intensive |
| OS | Shares host OS kernel | Requires full OS |

Docker containers run on the host OS kernel directly, while VMs run a complete OS with its own kernel. This makes Docker containers more lightweight, faster to start, and more resource-efficient compared to traditional VMs.

## Why Use Docker?

1. **Consistency across environments**: "It works on my machine" problem solved
2. **Isolation**: Applications and their dependencies run in isolated environments
3. **Portability**: Build once, run anywhere (development, testing, production)
4. **Microservices architecture**: Easily deploy and scale individual services
5. **Resource efficiency**: Less overhead compared to VMs
6. **Version control**: Track changes to your container images
7. **Rapid deployment**: Quick start-up and shutdown of containers
8. **Simplified updates**: Easy to update and roll back containers
9. **Better CI/CD integration**: Streamlined testing and deployment pipelines
10. **Large ecosystem**: Vast repository of pre-built images on Docker Hub

## Installing Docker on Fedora

### Method 1: Using DNF (Recommended)

```bash
# Remove old versions if they exist
sudo dnf remove docker docker-client docker-client-latest docker-common \
    docker-latest docker-latest-logrotate docker-logrotate docker-selinux \
    docker-engine-selinux docker-engine

# Add Docker repository
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo

# Install Docker Engine
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Verify installation
sudo docker run hello-world
```

### Method 2: Using the Convenience Script

```bash
# Download the script
curl -fsSL https://get.docker.com -o get-docker.sh

# Run the script
sudo sh get-docker.sh

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker
```

### Post-installation steps (optional)

```bash
# Add your user to the docker group to run Docker without sudo
sudo usermod -aG docker $USER

# Log out and log back in for the changes to take effect
# Or run the following command to apply the changes in the current session
newgrp docker

# Verify you can run Docker without sudo
docker run hello-world
```

## Installing Docker on RHEL

### Method 1: Using YUM/DNF with Docker CE Repository

```bash
# Remove old versions if they exist
sudo yum remove docker docker-client docker-client-latest docker-common \
    docker-latest docker-latest-logrotate docker-logrotate docker-engine

# Install required packages
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# Add Docker repository
sudo yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo

# Install Docker Engine
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Verify installation
sudo docker run hello-world
```

### Method 2: Using RHEL Subscription (RHEL 8 and later)

```bash
# Register your system if not already done
sudo subscription-manager register

# Enable the repositories that include Docker
sudo subscription-manager repos --enable rhel-8-for-x86_64-appstream-rpms
sudo subscription-manager repos --enable rhel-8-for-x86_64-baseos-rpms

# Install Podman, Buildah, and Skopeo (RHEL's container tools)
sudo yum install podman buildah skopeo

# If you need Docker specifically, install it
sudo yum module install container-tools

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker
```

## Docker Architecture

Docker uses a client-server architecture:

- **Docker Client**: The primary way to interact with Docker using CLI commands
- **Docker Daemon (dockerd)**: Server component that listens for API requests and manages Docker objects
- **Docker Registry**: Stores Docker images (Docker Hub is a public registry)
- **Docker Objects**: Images, containers, networks, volumes, etc.

![Docker Architecture](https://docs.docker.com/engine/images/architecture.svg)

## Docker Commands

### Basic Commands

```bash
# Get Docker version
docker --version

# Get detailed Docker info
docker info

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Run a container
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

# Example: Run an Ubuntu container with interactive shell
docker run -it ubuntu /bin/bash

# Stop a container
docker stop CONTAINER_ID_OR_NAME

# Start a stopped container
docker start CONTAINER_ID_OR_NAME

# Restart a container
docker restart CONTAINER_ID_OR_NAME

# Remove a container
docker rm CONTAINER_ID_OR_NAME

# Force remove a running container
docker rm -f CONTAINER_ID_OR_NAME
```

### Image Management

```bash
# List images
docker images

# Pull an image from Docker Hub
docker pull IMAGE_NAME[:TAG]

# Build an image from a Dockerfile
docker build -t IMAGE_NAME[:TAG] PATH_TO_DOCKERFILE

# Remove an image
docker rmi IMAGE_ID_OR_NAME

# Remove dangling images
docker image prune

# Remove all unused images
docker image prune -a
```

### Container Management

```bash
# Run a container with port mapping
docker run -p HOST_PORT:CONTAINER_PORT IMAGE

# Run a container with volume mounting
docker run -v HOST_PATH:CONTAINER_PATH IMAGE

# Run a container in the background (detached mode)
docker run -d IMAGE

# Execute a command in a running container
docker exec -it CONTAINER_ID_OR_NAME COMMAND

# View container logs
docker logs CONTAINER_ID_OR_NAME

# Follow container logs
docker logs -f CONTAINER_ID_OR_NAME

# View container resource usage
docker stats [CONTAINER_ID_OR_NAME]

# Copy files between container and host
docker cp CONTAINER_ID:SOURCE_PATH DESTINATION_PATH
docker cp SOURCE_PATH CONTAINER_ID:DESTINATION_PATH
```

### Network Commands

```bash
# List networks
docker network ls

# Create a network
docker network create NETWORK_NAME

# Connect a container to a network
docker network connect NETWORK_NAME CONTAINER_ID_OR_NAME

# Disconnect a container from a network
docker network disconnect NETWORK_NAME CONTAINER_ID_OR_NAME

# Inspect a network
docker network inspect NETWORK_NAME

# Remove a network
docker network rm NETWORK_NAME
```

### Volume Commands

```bash
# List volumes
docker volume ls

# Create a volume
docker volume create VOLUME_NAME

# Inspect a volume
docker volume inspect VOLUME_NAME

# Remove a volume
docker volume rm VOLUME_NAME

# Remove all unused volumes
docker volume prune
```

### System Commands

```bash
# Display Docker disk usage
docker system df

# Remove unused data (containers, networks, images, volumes)
docker system prune

# Remove all unused resources (including volumes)
docker system prune -a --volumes

# View events in real-time
docker events
```

## Docker Compose

Docker Compose is a tool for defining and running multi-container Docker applications. It uses a YAML file to configure application services, networks, and volumes.

### Installing Docker Compose

#### On Fedora/RHEL with package manager

```bash
# If you installed Docker using the repository method
# Docker Compose plugin should already be installed with docker-ce
# You can check with:
docker compose version

# If you need to install it separately:
sudo dnf install docker-compose-plugin
```

#### Using pip (Python package manager)

```bash
# Install pip if needed
sudo dnf install python3-pip

# Install Docker Compose
pip3 install docker-compose
```

#### Manual installation

```bash
# Download the current stable release
sudo curl -L "https://github.com/docker/compose/releases/download/v2.23.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Apply executable permissions
sudo chmod +x /usr/local/bin/docker-compose

# Create a symbolic link
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

# Verify installation
docker-compose --version
```

### Using Docker Compose

#### Sample docker-compose.yml

```yaml
version: '3'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./website:/usr/share/nginx/html
    restart: always

  db:
    image: postgres:13
    environment:
      POSTGRES_PASSWORD: example
      POSTGRES_USER: user
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: always

volumes:
  postgres_data:
```

#### Docker Compose commands

```bash
# Start services
docker-compose up

# Start services in detached mode
docker-compose up -d

# Stop services
docker-compose down

# Stop services and remove volumes
docker-compose down -v

# View logs
docker-compose logs

# Follow logs
docker-compose logs -f

# Rebuild services
docker-compose up --build

# List running services
docker-compose ps

# Execute a command in a service
docker-compose exec SERVICE_NAME COMMAND

# Scale a service
docker-compose up -d --scale SERVICE_NAME=NUM_INSTANCES
```

## Advanced Docker Concepts

### Dockerfile Best Practices

```dockerfile
# Use official base images
FROM alpine:latest

# Set working directory
WORKDIR /app

# Copy only what's needed for dependency installation
COPY package*.json ./

# Combine RUN commands to reduce layers
RUN apk add --no-cache nodejs npm && \
    npm install && \
    npm cache clean --force

# Add application code
COPY . .

# Use environment variables
ENV NODE_ENV=production

# Specify a non-root user
USER node

# Document exposed ports
EXPOSE 8080

# Use ENTRYPOINT for executables
ENTRYPOINT ["node", "app.js"]

# Use CMD for default arguments
CMD ["--config", "config.json"]
```

### Multi-stage Builds

```dockerfile
# Build stage
FROM node:14 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Docker Networking

Docker provides several network drivers:
- `bridge`: Default network driver, isolated network on host
- `host`: Removes network isolation, uses host's network directly
- `overlay`: Connect multiple Docker daemons for Swarm services
- `macvlan`: Assign MAC addresses to containers, appear as physical devices
- `none`: Disable networking for container

### Docker Volumes and Data Persistence

Types of data persistence in Docker:
1. **Volumes**: Managed by Docker, stored in `/var/lib/docker/volumes/`
2. **Bind mounts**: Link to a host system directory
3. **tmpfs mounts**: Stored in host system memory only

### Resource Constraints

```bash
# Limit CPU usage
docker run --cpus=0.5 IMAGE_NAME

# Limit memory usage
docker run --memory=512m IMAGE_NAME

# Set memory and swap limit
docker run --memory=512m --memory-swap=768m IMAGE_NAME
```

### Docker Security Best Practices

1. Keep Docker updated
2. Use minimal base images (alpine, distroless)
3. Scan images for vulnerabilities
4. Don't run containers as root
5. Use read-only filesystems when possible
6. Limit capabilities
7. Use secrets management
8. Implement resource constraints
9. Apply content trust
10. Use network segmentation

### Container Orchestration

For production environments with multiple containers, consider:
- **Docker Swarm**: Docker's native clustering and orchestration tool
- **Kubernetes**: Industry standard for container orchestration
- **Amazon ECS/EKS**: AWS container services
- **Azure AKS**: Microsoft's Kubernetes service
- **Google GKE**: Google's Kubernetes service

These tools help with:
- Load balancing
- Scaling
- Service discovery
- Rolling updates
- Self-healing
- Secret management
