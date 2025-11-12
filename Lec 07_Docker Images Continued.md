# Lecture 7: Docker Images Continued

## Docker Image Basics

A **Docker image** is a read-only, layered blueprint for creating containers. Running `docker run <image>` creates a container with a writable layer on top.

**Key Points:**
- Images composed of layers
- Each Dockerfile instruction creates a new layer
- Containers add a thin writable layer

## Dockerfile

A text file with instructions to build an image.

## Building Images

```bash
# Custom filename
docker build -t imagename -f MyDockerfile .

# Default "Dockerfile"
docker build -t imagename .
```

- Image name must be lowercase
- `-t` = tag (name)
- `.` = build context

**Build Process:** Reads Dockerfile → Downloads base image → Executes line by line → Creates layers → Final image

Stored under: `/var/lib/docker`

## Build Time vs Runtime Commands

| Phase | Commands |
|-------|----------|
| **Build Time** | FROM, RUN, COPY, ADD, ENV, WORKDIR |
| **Run Time** | CMD, ENTRYPOINT |

---

## Dockerfile Instructions

### FROM - Base Image

```dockerfile
FROM <image>[:<tag>]
FROM ubuntu:22.04
FROM python:3.12-slim
```

- Mandatory (must be first)
- Use minimal images (-alpine, -slim)
- Pin versions for reproducibility

### RUN - Execute During Build

```dockerfile
RUN apt-get update && apt-get install -y curl
RUN pip install flask
```

**Best Practice:** Combine commands to reduce layers
```dockerfile
RUN apt-get update && apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

### CMD - Default Runtime Command

```dockerfile
CMD ["python", "app.py"]  # Recommended
```

- Only last CMD is used
- Can be overridden: `docker run myimage ls`
- Use JSON format

### ENTRYPOINT - Fixed Main Command

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

- Cannot be easily overridden
- Always runs

### ENTRYPOINT + CMD Together

```dockerfile
ENTRYPOINT ["echo"]
CMD ["Hello World"]
```

```bash
docker run myimage           # Output: Hello World
docker run myimage Goodbye   # Output: Goodbye
```

**Concept:** ENTRYPOINT = command, CMD = default arguments

### COPY vs ADD

| Feature | COPY | ADD |
|---------|------|-----|
| Local files | Yes | Yes |
| URLs | No | Yes |
| Auto-extract tar | No | Yes |
| **Recommended** | Normal use | Special cases only |

```dockerfile
COPY requirements.txt /app/
ADD app.tar.gz /app/        # Auto-extracts
```

### Other Instructions

```dockerfile
WORKDIR /app              # Set working directory
ENV PORT=8080             # Environment variable
EXPOSE 8080               # Document port (metadata)
```

---

## Flask Web App Example

### Files

**app.py:**
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from Docker!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

**requirements.txt:**
```
flask==3.0.2
```

**Dockerfile:**
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
ENV PORT=8080
EXPOSE 8080
ENTRYPOINT ["python"]
CMD ["app.py"]
```

**Build & Run:**
```bash
docker build -t flask-app .
docker run -p 8080:8080 flask-app
```

---

## Best Practices

| Category | Practice | Benefit |
|----------|----------|---------|
| Base Image | Use -alpine/-slim | Smaller size |
| Layers | Combine RUNs | Fewer layers |
| Caching | Copy dependencies first | Faster rebuilds |
| Cleanup | Remove caches | Reduced bloat |
| Security | Run as non-root | Safer |
| Versioning | Pin versions | Reproducible |

### Secure Dockerfile

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt && \
    useradd -m appuser
COPY . .
USER appuser
EXPOSE 8080
ENTRYPOINT ["python"]
CMD ["app.py"]
```

---

## Multi-Stage Builds

Reduce final image size by keeping only necessary components.

**Java Application Example:**
```dockerfile
# Stage 1: Build
FROM openjdk:17-jdk AS builder
WORKDIR /app
COPY . .
RUN ./mvnw package

# Stage 2: Run
FROM openjdk:17-jre
WORKDIR /app
COPY --from=builder /app/target/app.jar .
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Benefit:** Build stage has JDK, final stage has only JRE (smaller).

---

## Performance Optimization

### 1. Reduce Layers
```dockerfile
# Bad - 3 layers
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*

# Good - 1 layer
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

### 2. Optimize Caching
```dockerfile
COPY requirements.txt .      # Changes rarely
RUN pip install -r requirements.txt
COPY . .                     # Changes often
```

### 3. Use .dockerignore
```
__pycache__
*.pyc
.git
node_modules
```

---

## Image Inspection

```bash
# View layers
docker history <image>

# Inspect metadata
docker inspect <image>

# Scan vulnerabilities
docker scan <image>
trivy image <image>

# View size
docker images
```

---

## Summary

| Instruction | Purpose |
|-------------|---------|
| FROM | Base image (mandatory) |
| RUN | Execute during build |
| COPY/ADD | Copy files |
| WORKDIR | Set directory |
| ENV | Environment variables |
| EXPOSE | Document port |
| CMD | Default command (overridable) |
| ENTRYPOINT | Fixed command |

**Key Concepts:**
- ENTRYPOINT + CMD: command + arguments
- Combine RUN commands to reduce layers
- Copy dependencies first for caching
- Multi-stage builds for smaller images
- Run as non-root in production
- Use .dockerignore to exclude files
