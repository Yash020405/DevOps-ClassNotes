# Lecture 9: Docker Volumes

## Why Volumes? The Problem

Containers are ephemeral (temporary). When we remove a container:
- Writable layer gets destroyed
- All data in `/var/lib/mysql`, `/app/logs` etc. = GONE

**Important difference:**
```bash
docker stop <id>    # Data stays (container still exists)
docker start <id>   # Data is there

docker rm <id>      # Data LOST (container removed)
```

So we need Docker Volumes to save data even after container is removed.

---

## How Container Storage Works

Basically:
- Image has read-only layers
- Container adds a writable layer on top
- When container removed → writable layer deleted → data gone

---

## Types of Volumes (4 Types)

| Type | Survives? | Where? | When to Use |
|------|-----------|--------|-------------|
| **Anonymous** | Yes | `/var/lib/docker/volumes/...` | Quick temp storage |
| **Named** | Yes | `/var/lib/docker/volumes/<name>` | Databases, important data |
| **Bind Mount** | Host | Host filesystem | Development, logs |
| **tmpfs** | No (RAM only) | RAM | Fast temp data |

---

## 1. Anonymous Volumes

Docker creates it automatically - gives it a random name.

```bash
docker run -d -v /data nginx
```

**What happens:**
- Gets random ID like `d3c9248bc5a8f9301a6e`
- For quick temporary stuff
- Gets deleted if you use `--rm` flag

**See all volumes:**
```bash
docker volume ls
```

---

## 2. Named Volumes (Most Used)

We create this ourselves with a name.

```bash
# Create volume
docker volume create scalervolume

# Check details
docker volume inspect scalervolume

# Use it
docker run -it -v scalervolume:/data ubuntu bash
```

**Cool part:** Even if we delete container, data stays!
```bash
docker rm -f <container>
docker run -it -v scalervolume:/data ubuntu bash  # Data still there!
```

**Use for:**
- Databases (MySQL, MongoDB, etc.)
- File uploads
- Any important data

---

## 3. Bind Mounts

This directly connects a folder from our computer to container.

**Format:**
```bash
-v /host/path:/container/path
```

**Example:**
```bash
docker run -d \
  -v $(pwd)/website:/usr/share/nginx/html \
  -p 8080:80 \
  nginx
```

**Why it's useful:**
- Edit files on computer → instantly appears in container (no rebuild!)
- Good for development
- But risky in production (gives container access to host)

**Use for:**
- Development (hot reload)
- Logs
- Config files

**Make it read-only:**
```bash
docker run -v /host:/container:ro nginx
```

---

## 4. tmpfs Mounts (RAM Only)

This stores data in RAM (not on disk) - super fast but temporary.

```bash
docker run -d --tmpfs /cache nginx

# or
docker run -d --mount type=tmpfs,target=/cache nginx
```

**What happens:**
- Very fast (because RAM)
- Stop container → data gone
- Never touches disk

**Use for:**
- Quick caches
- Temporary secrets
- Sensitive data that shouldn't be saved

---

## Where Data Actually Gets Stored

**Named/Anonymous volumes:**
```
/var/lib/docker/volumes/<volume_name>/_data/
```

**Bind mounts:** Whatever path you specified

**tmpfs:** RAM (nowhere on disk)

---

## Sharing Data Between Containers

Two containers can use the same volume - they'll see each other's changes!

```bash
# Container 1
docker run -it -v scalervolume:/data ubuntu bash

# Container 2
docker run -it -v scalervolume:/data ubuntu bash
```

Both can read/write same files in real-time.

---

## Common Commands

```bash
docker volume create <name>      # Create
docker volume ls                 # List all
docker volume inspect <name>     # See details
docker volume rm <name>          # Delete
docker volume prune              # Delete unused
```

---

## Quick Examples

**Anonymous:**
```bash
docker run -d -v /data nginx
```

**Named:**
```bash
docker volume create mydb
docker run -d -v mydb:/var/lib/mysql mysql:8
```

**Bind Mount:**
```bash
mkdir website
echo "Hello" > website/index.html
docker run -d -v $(pwd)/website:/usr/share/nginx/html -p 8080:80 nginx
```

**tmpfs:**
```bash
docker run -d --tmpfs /cache nginx
```

---

## When to Use What

**Named Volumes:**
- Databases
- File uploads
- Anything important

**Bind Mounts:**
- Development (live editing)
- Logs
- Config files

**tmpfs:**
- Quick cache
- Passwords (temporary)
- Fast temp storage

---

## Testing Data Persistence

```bash
# First container
docker run -it -v scalervolume:/data ubuntu bash
echo "test" > /data/file.txt
exit

# Delete container
docker rm -f <container>

# New container, same volume
docker run -it -v scalervolume:/data ubuntu bash
cat /data/file.txt  # File is still there!
```

---

## What to Use When

| Need | Use |
|------|-----|
| Production DB | Named volume |
| Development | Bind mount |
| Quick cache | tmpfs |
| Logs | Bind mount |
| Test stuff | Anonymous |

---

## Key Differences

**Named vs Anonymous:**
- Named = we choose name, reusable
- Anonymous = random ID

**Volume vs Bind Mount:**
- Volume = Docker manages it
- Bind Mount = we manage it (direct folder link)

**Persistent vs tmpfs:**
- Persistent = stays after container removed
- tmpfs = RAM only, disappears

---

## Important Points to Remember

1. `docker stop` keeps data, `docker rm` deletes it
2. Named volumes = best for production
3. Bind mounts = risky (gives container host access)
4. tmpfs = most secure (never saved to disk)
5. Multiple containers can share one volume
6. Volumes live in `/var/lib/docker/volumes/`
7. Add `:ro` for read-only

---

## Summary

**Problem:** Container removed = data lost

**Solution:** Use volumes!

**4 Types:**
1. Anonymous - random name, auto-created
2. Named - we name it, persists (use this for DBs)
3. Bind Mount - links host folder (use for dev)
4. tmpfs - RAM only, fast but temporary

**Quick Reference:**
```bash
docker volume create mydata
docker run -v mydata:/data ubuntu
docker run -v $(pwd):/app ubuntu
docker run --tmpfs /cache ubuntu
```

**Where data lives:**
- Named/Anonymous: `/var/lib/docker/volumes/`
- Bind: wherever you put it
- tmpfs: RAM
