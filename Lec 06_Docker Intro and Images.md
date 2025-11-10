# Lecture 6: Docker Introduction and Images

## What is Docker?

Docker is an open platform for developing, shipping, and running applications. It enables you to separate applications from infrastructure and deliver software quickly.

**Key Benefit**: Package an application with all its dependencies (libraries, system tools, configuration) and ship it as one package that runs reliably across environments.

---

## Architecture Evolution

### 1. Bare Metal Architecture

```
Hardware → Kernel → Libraries → Binaries → Application
```

- Application runs directly on hardware
- No virtualization layer
- Full performance but only one OS at a time

### 2. Hypervisor-Based Virtualization

**Hypervisor** is software that creates and runs multiple Virtual Machines (VMs) on a single physical machine.

#### Types of Hypervisors

| Type | Description | Examples |
|------|-------------|----------|
| **Type-1 (Bare Metal)** | Directly on hardware, no OS underneath | VMware ESXi, Hyper-V, Xen |
| **Type-2 (Hosted)** | Runs on top of host OS | VirtualBox, VMware Workstation |

**Each VM has:**
- Virtual CPU, RAM, Disk, Network
- Its own Operating System
- Its own Kernel

**Why VMs are Heavy:**
Each VM boots its own full OS with complete boot process:
```
BIOS → MBR → Bootloader → Kernel → INIT → Running Processes
```

### 3. Docker Architecture

```
Hardware → Host OS Kernel → Docker Engine → Containers
```

**Docker containers DO NOT have:**
- BIOS
- MBR
- Bootloader
- Separate Kernel
- Full OS

**Docker uses Host OS Kernel directly**

Startup time: **Milliseconds** (no boot sequence)

---

## Docker Components

### Docker Client
Commands you run:
- `docker build`
- `docker pull`
- `docker run`
- `docker ps`

Client sends requests to Docker Daemon via REST API.

### Docker Daemon (`dockerd`)
The brain of Docker. Responsible for:
- Building images
- Running/stopping containers
- Managing storage, networking, volumes

### Docker Host
Contains:
- Images (read-only templates)
- Containers (running instances)
- Storage and Networking

### Docker Registry (Docker Hub)
Public repository for pulling/pushing images (Ubuntu, Nginx, Node.js, etc.)

---

## Why Docker is Lightweight

| Feature | VMs | Docker Containers |
|---------|-----|-------------------|
| OS Boot | Full boot process | No OS boot |
| Kernel | Separate for each VM | Shared host kernel |
| Startup Time | Minutes | Milliseconds |
| Size | GBs | MBs |
| Resource Usage | Heavy | Lightweight |

---

## Docker Run Flow

1. User runs: `docker run <image-name>`
2. Docker Client sends request to Docker Daemon
3. Daemon checks for local image
4. If not found, downloads from Docker Hub
5. Daemon creates container with:
   - Isolated filesystem
   - Isolated network
   - Isolated process tree
   - Shared kernel
6. Container starts instantly

---

## Docker Installation

### Using Convenience Script
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

### Manual Installation (Linux)
```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
```

### Verify
```bash
docker --version
docker run hello-world
```

---

## Essential Docker Commands

### Container Management

| Command | Description |
|---------|-------------|
| `docker run <image>` | Create and start new container |
| `docker run -d <image>` | Run in detached mode (background) |
| `docker run -it ubuntu bash` | Interactive terminal in container |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (including stopped) |
| `docker start <id>` | Start stopped container |
| `docker stop <id>` | Stop running container |
| `docker rm <id>` | Remove container |

### Executing Commands

**docker run vs docker exec:**
- `docker run`: Creates NEW container
- `docker exec`: Runs command in EXISTING running container

```bash
# Enter existing running container
docker exec -it <container_id> bash

# Create and enter new container
docker run -it ubuntu bash
```

### Container Inspection

```bash
# Get detailed container info (including IP address)
docker container inspect <container_id>

# View processes inside container
docker exec -it <container_id> ps -ef
```

---

## Working with Docker Images

### Image Basics

- **Image** = read-only template (blueprint)
- **Container** = running instance of image
- Multiple containers can be created from single image

### Image Commands

```bash
# List images
docker images

# Pull image from Docker Hub
docker pull nginx

# Remove image
docker rmi <image_id>

# View image layers and history
docker history <image_name>

# Inspect image details
docker image inspect <image_name>
```

### Creating Custom Images

#### Method 1: From Running Container
```bash
# Make changes in container, then commit
docker commit -m "description" <container_id> <username>/<imagename>
```

#### Method 2: Using Dockerfile
```dockerfile
FROM ubuntu:22.04           # Base layer
RUN apt-get install nginx   # New layer
COPY . /var/www/html        # New layer
CMD ["nginx"]               # Metadata layer
```

---

## Docker Image Layers

Docker images use a **layered file system** (Union File System).

### Layer Architecture

```
+------------------------------------+
| Writable Container Layer           | ← changes while running
+------------------------------------+
| CMD / ENTRYPOINT (metadata)        |
| COPY app files                     |
| RUN apt-get install                |
| FROM ubuntu:22.04 (base)           |
+------------------------------------+
           ↑
     Read-only Layers
```

### How Layers Work

- Each `RUN`, `COPY`, `ADD` command creates a new layer
- Layers are stacked using Union File System
- Layers are read-only
- Container adds writable layer on top

### Copy-on-Write Mechanism

When a file is modified in a container:
1. File is copied from read-only layer to writable layer
2. Modifications happen only in writable layer
3. Original image layer remains unchanged

### Layer Storage Location

On Linux (overlay2 driver):
```
/var/lib/docker/overlay2/
```

### Layer Reuse and Caching

- Layers identified by SHA256 hashes
- If two images share base layers, Docker doesn't duplicate them
- Makes builds faster and space-efficient

### Inspecting Layers

```bash
# View layer details
docker image inspect <image_name> --format='{{json .RootFS.Layers}}' | jq

# View layer history
docker history <image_name>
```

Example output:
```
IMAGE       CREATED BY                      SIZE
<missing>   CMD ["nginx"]                   0B
<missing>   RUN apt-get install nginx       25MB
<missing>   RUN apt-get update              1.2MB
<missing>   FROM ubuntu:22.04               
```

---

## Docker Hub Integration

### Login
```bash
docker login -u <username>
```

### Push Image
```bash
# Image name must start with Docker Hub username
docker push <username>/<imagename>
```

**Important**: Image must be tagged with your username to push to Docker Hub.

---

## Container Networking

### Finding Container IP
```bash
docker container inspect <container_id> | grep IPAddress
```

### Communicating Between Containers
Once you have the IP address, use tools like `curl` to communicate:
```bash
curl http://<container_ip>:<port>
```

---

## Detached vs Attached Modes

| Mode | Command | Description |
|------|---------|-------------|
| **Attached** | `docker run nginx` | Container runs in foreground, logs visible |
| **Detached** | `docker run -d nginx` | Container runs in background |

---

## Key Takeaways

- **Docker containers are lightweight** - no OS boot, share host kernel
- **Images are layered** - each instruction creates new layer
- **Containers are ephemeral** - writable layer is temporary
- **docker run creates new container** - use docker exec for existing containers
- **Layer caching makes builds efficient** - reuses unchanged layers
- **Union FS merges layers** - provides single unified view
- **Copy-on-write saves space** - modifications don't affect original layers
