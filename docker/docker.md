# Docker - Complete Hands-On Guide

**Folder:** `docker-wordpress-mysql/`

A complete Docker guide covering image pull/push, container creation, Dockerfile keywords, Docker networking, and running a **WordPress + MySQL** multi-container application on a shared custom network.

---

##  What You'll Learn

- Pull and Push Docker images
- Create and manage containers
- Write a Dockerfile and understand its keywords
- Create Docker networks
- Run WordPress + MySQL on the same Docker network

---

##  Prerequisites

- Docker installed on your machine or EC2 instance
- Docker Hub account (for push)
- Basic Linux command knowledge

---

## 1.  Pull a Docker Image

Pull an image from Docker Hub to your local machine.

```bash
# Syntax
docker pull <image-name>:<tag>

# Examples
docker pull ubuntu
docker pull nginx:latest
docker pull mysql:8.0
docker pull wordpress:latest
```

**Verify the pulled image:**
```bash
docker images
```

**Output:**
```
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
wordpress     latest    abc123...      2 days ago     615MB
mysql         8.0       def456...      3 days ago     578MB
ubuntu        latest    ghi789...      5 days ago     77.8MB
```

---

## 2.  Push a Docker Image to Docker Hub

### Step 1 — Login to Docker Hub
```bash
docker login
# Enter your Docker Hub username and password
```

### Step 2 — Tag your image
```bash
# Syntax
docker tag <local-image>:<tag> <dockerhub-username>/<repo-name>:<tag>

# Example
docker tag myapp:latest sumitkalamkar/myapp:v1
```

### Step 3 — Push the image
```bash
docker push sumitkalamkar/myapp:v1
```

### Step 4 — Verify on Docker Hub
Visit: `https://hub.docker.com/r/sumitkalamkar/myapp`

---

## 3.  Create and Run a Container

```bash
# Basic run
docker run ubuntu

# Run with interactive terminal
docker run -it ubuntu bash

# Run in background (detached)
docker run -d nginx

# Run with name + port mapping
docker run -d --name my-nginx -p 8080:80 nginx

# Run with environment variables
docker run -d \
  --name my-mysql \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  mysql:8.0
```

**Useful Container Commands:**
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop a container
docker stop my-nginx

# Start a stopped container
docker start my-nginx

# Remove a container
docker rm my-nginx

# View container logs
docker logs my-nginx

# Execute a command inside running container
docker exec -it my-mysql bash
```

---

## 4.  Dockerfile — Keywords Explained

A `Dockerfile` is a text file with instructions to build a custom Docker image.

**Sample Dockerfile:**
```dockerfile
# Use an official base image
FROM ubuntu:22.04

# Maintainer info
LABEL maintainer="sumitkalamkar@example.com"

# Set environment variable
ENV APP_PORT=3000

# Set working directory inside container
WORKDIR /app

# Copy files from host to container
COPY . .

# Run commands during image build
RUN apt-get update && apt-get install -y nodejs npm

# Expose the port the app runs on
EXPOSE 3000

# Command that runs when the container starts
CMD ["node", "server.js"]
```

###  Dockerfile Keywords Reference

| Keyword | Purpose | Example |
|---------|---------|---------|
| `FROM` | Base image to build upon | `FROM ubuntu:22.04` |
| `LABEL` | Add metadata (author, version) | `LABEL version="1.0"` |
| `ENV` | Set environment variables | `ENV NODE_ENV=production` |
| `WORKDIR` | Set working directory in container | `WORKDIR /app` |
| `COPY` | Copy files from host into image | `COPY ./src /app/src` |
| `ADD` | Like COPY but supports URLs & tar extraction | `ADD archive.tar.gz /app/` |
| `RUN` | Execute commands during build | `RUN apt-get install -y curl` |
| `EXPOSE` | Document which port the app uses | `EXPOSE 8080` |
| `CMD` | Default command when container starts (overridable) | `CMD ["node", "app.js"]` |
| `ENTRYPOINT` | Fixed command, can't be overridden easily | `ENTRYPOINT ["python"]` |
| `ARG` | Build-time variable | `ARG VERSION=1.0` |
| `VOLUME` | Mount point for persistent storage | `VOLUME /var/lib/mysql` |
| `USER` | Switch to non-root user | `USER appuser` |

###  KEY DIFFERENCE: CMD vs ENTRYPOINT

```dockerfile
# CMD — can be replaced at runtime
CMD ["node", "server.js"]
# Override: docker run myimage python app.py  ← replaces CMD

# ENTRYPOINT — always runs, CMD becomes arguments
ENTRYPOINT ["node"]
CMD ["server.js"]
# Result: docker run myimage  → runs "node server.js"
# Override args: docker run myimage app.js  → runs "node app.js"
```

###  Build the Docker Image
```bash
# Build from Dockerfile in current directory
docker build -t myapp:v1 .

# Build with custom Dockerfile path
docker build -f /path/to/Dockerfile -t myapp:v1 .

# View built images
docker images
```

---

## 5.  Docker Networking

Docker networking lets containers communicate with each other securely.

### Network Types

| Type | Description |
|------|-------------|
| `bridge` | Default network. Containers on same bridge can communicate |
| `host` | Container shares host's network stack |
| `none` | No network access |
| `custom bridge` | User-defined bridge — best for multi-container apps |

### Create a Custom Network

```bash
# Create network
docker network create my-network

# Create with custom subnet
docker network create --subnet=172.20.0.0/16 my-network

# List all networks
docker network ls

# Inspect a network (shows connected containers)
docker network inspect my-network

# Remove a network
docker network rm my-network
```

---

## 6.  WordPress + MySQL on Same Docker Network

This is the complete setup to run WordPress connected to MySQL — both on the same Docker network.

### Architecture

```
Browser (port 8080)
       ↓
┌──────────────────┐        ┌─────────────────────┐
│  WordPress        │◄──────►│  MySQL               │
│  Container        │        │  Container           │
│  Port: 80         │        │  Port: 3306          │
└──────────────────┘        └─────────────────────┘
       └──────────── wp-network ────────────────────┘
```

### Step 1 — Create a Custom Network
```bash
docker network create wp-network
```

### Step 2 — Run MySQL Container
```bash
docker run -d \
  --name mysql-db \
  --network wp-network \
  -e MYSQL_ROOT_PASSWORD=rootpassword \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wpuser \
  -e MYSQL_PASSWORD=wppassword \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0
```

**Environment variables explained:**

| Variable | Value | Description |
|----------|-------|-------------|
| `MYSQL_ROOT_PASSWORD` | rootpassword | Root password for MySQL |
| `MYSQL_DATABASE` | wordpress | Database name WordPress will use |
| `MYSQL_USER` | wpuser | MySQL user for WordPress |
| `MYSQL_PASSWORD` | wppassword | Password for the MySQL user |

### Step 3 — Run WordPress Container
```bash
docker run -d \
  --name wordpress-app \
  --network wp-network \
  -e WORDPRESS_DB_HOST=mysql-db:3306 \
  -e WORDPRESS_DB_NAME=wordpress \
  -e WORDPRESS_DB_USER=wpuser \
  -e WORDPRESS_DB_PASSWORD=wppassword \
  -p 8080:80 \
  -v wordpress-data:/var/www/html \
  wordpress:latest
```

**Environment variables explained:**

| Variable | Value | Description |
|----------|-------|-------------|
| `WORDPRESS_DB_HOST` | mysql-db:3306 | MySQL container name (DNS on same network) |
| `WORDPRESS_DB_NAME` | wordpress | Database to connect to |
| `WORDPRESS_DB_USER` | wpuser | DB username |
| `WORDPRESS_DB_PASSWORD` | wppassword | DB password |

>  **Key concept:** `WORDPRESS_DB_HOST=mysql-db` — WordPress uses the **container name** as hostname. This works because both containers are on the same `wp-network`. Docker's built-in DNS resolves container names automatically.

### Step 4 — Verify Everything is Running
```bash
docker ps
```

**Expected output:**
```
CONTAINER ID   IMAGE             STATUS         PORTS                  NAMES
a1b2c3d4e5f6   wordpress:latest  Up 2 minutes   0.0.0.0:8080->80/tcp   wordpress-app
f6e5d4c3b2a1   mysql:8.0         Up 3 minutes   3306/tcp               mysql-db
```

### Step 5 — Access WordPress
Open your browser:
```
http://localhost:8080
# or if on EC2:
http://<EC2-Public-IP>:8080
```

You should see the WordPress installation wizard! 

### Step 6 — Check Network Connections
```bash
docker network inspect wp-network
```

This shows both containers connected to `wp-network`.

---

##  Cleanup

```bash
# Stop containers
docker stop wordpress-app mysql-db

# Remove containers
docker rm wordpress-app mysql-db

# Remove volumes
docker volume rm mysql-data wordpress-data

# Remove network
docker network rm wp-network

# Remove images (optional)
docker rmi wordpress:latest mysql:8.0
```

---

##  Project Structure

```
docker-wordpress-mysql/
│
├── README.md              ← This file
└── Dockerfile             ← Sample custom Dockerfile
```

---

##  Summary

| Concept | Command |
|---------|---------|
| Pull image | `docker pull <image>` |
| Push image | `docker push <username>/<image>:<tag>` |
| Run container | `docker run -d --name <name> <image>` |
| Build image | `docker build -t <name>:<tag> .` |
| Create network | `docker network create <name>` |
| Connect to network | `--network <name>` flag on `docker run` |
| Container-to-container DNS | Use container name as hostname |

---

## Author

**Sumit Kalamkar**  
AWS Cloud Enthusiast | Hands-on Builder  
 [GitHub](https://github.com/Sumitkalamkar)
