# Lecture 5: Introduction to Docker

## What is Docker?

Docker is an open-source platform to build, package, ship, and run applications in **containers**.

A **container** is a lightweight, standalone package that includes everything needed to run an application (code, runtime, libraries, system tools, configuration).

**Philosophy: "If It Fits, It Ships"** - Once packaged into a container, it can be deployed to any machine running Docker.

---

## Why Docker?

| Before Docker | With Docker |
|---------------|-------------|
| Apps installed directly on OS | Each app runs in isolated container |
| Library version conflicts | All dependencies bundled together |
| Manual setup on each machine | Consistent environment everywhere |

---

## Key Concepts

| Concept | Description |
|---------|-------------|
| **Image** | Read-only template to create containers (Image = Class) |
| **Container** | Running instance of an image (Container = Object) |
| **Dockerfile** | Text file with instructions to build an image |
| **Docker Hub** | Public repository of images |

- Containers share host OS kernel (lightweight compared to VMs)
- Images can be local or pulled from Docker Hub

---

## Docker Architecture

```
Docker CLI (Client)  →  Docker Daemon (Server)  →  Containers/Images
```

- **Docker Client**: CLI tool (`docker`) that sends commands
- **Docker Daemon**: Background service that pulls images, creates and manages containers

---

## Installation

```bash
# Using convenience script
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Verify
docker --version
docker run hello-world
```

---

## Essential Commands

| Command | Description |
|---------|-------------|
| `docker run hello-world` | Run a container |
| `docker run -it ubuntu bash` | Interactive container with shell |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker images` | List images |
| `docker pull nginx` | Download image |
| `docker stop <id>` | Stop container |
| `docker rm <id>` | Remove container |
| `docker rmi <id>` | Remove image |

**Flags for interactive mode:**
- `-i` → interactive
- `-t` → terminal (tty)

---

## Connecting to AWS EC2

```bash
cd /path/to/your/pem-file
ssh -i your-key.pem ubuntu@ec2-xx-xx-xx-xx.compute.amazonaws.com
```

---

## Inside a Container

```bash
docker run -it ubuntu bash
```

- Hostname changes to container ID (e.g., `root@f21106bca534:/#`)
- Run `ps -ef` to see isolated processes inside container

---

## Docker Image Layers

Images are built from multiple read-only layers (Union File System).

```dockerfile
FROM ubuntu:22.04           # Base layer
RUN apt-get install nginx   # New layer
COPY . /var/www/html        # New layer
CMD ["nginx"]               # Metadata layer
```

**Container adds a writable layer on top** - changes happen only in this layer (copy-on-write).

### Layer Caching
- Layers identified by SHA256 hashes
- Shared layers are not duplicated (faster builds, saves space)

```bash
docker history <image_name>   # View layers
```

---

## How Docker Works (Flow)

1. `docker run hello-world`
2. Client sends request to Daemon
3. Daemon checks for local image → pulls from Docker Hub if missing
4. Daemon creates and runs container
5. Output sent back to terminal

---

## Key Takeaways

- **Image** = blueprint, **Container** = running instance
- `docker run` creates a new container each time
- Containers share host kernel but have isolated processes
- Use `-it` flags for interactive shell access
- Check daemon status: `systemctl status docker`
