# Lecture 8: Docker Containers

## What is a Container?

A container is **NOT** a VM, but a Linux process with:
- Temporary filesystem + namespaces + cgroups
- Shares host kernel
- Own filesystem, network, hostname, process tree
- All via Linux kernel features

---

## Container vs VM

| Feature | VM | Container |
|---------|----|-----------|
| **Kernel** | Own kernel | Shares host |
| **Boot** | BIOS → MBR → Kernel → INIT | Direct process |
| **Startup** | Minutes | ~50ms |
| **Size** | GBs | MBs |
| **Isolation** | Hypervisor | Namespaces |

---

## Container = Process

`docker run nginx` launches a Linux process under isolation (not a VM).

```
Host: systemd → containerd → shim → nginx (container)
```

---

## Why Ephemeral?

Containers have read-only image layers + temporary writable layer (deleted on stop).

**Persist data:** Volumes, bind mounts, external storage.

---

## Container Architecture

```
docker → containerd → containerd-shim → runc → your process
```

- **containerd**: Manages containers, images, networking
- **containerd-shim**: Parent process, allows containerd restart
- **runc**: Uses Linux syscalls, sets up namespaces/cgroups

---

## Container Creation Steps

1. Pull image → 2. Unpack layers (OverlayFS) → 3. Create namespaces → 4. Apply cgroups → 5. chroot → 6. Launch process

---

## Linux Namespaces - Isolation

Namespaces create illusions - filter what process can see.

| Namespace | Isolation |
|-----------|----------|
| **PID** | Process sees itself as PID 1 (on host: different PID) |
| **Network** | Own IP, routing table, firewall rules |
| **Mount** | Own root filesystem (/) |
| **UTS** | Own hostname |
| **IPC** | Own shared memory, semaphores |
| **User** | Root inside ≠ real root on host |

---

## Linux Cgroups - Resource Limits

**Namespaces** = Isolation | **Cgroups** = Resource Limits

Cgroups limit: CPU, Memory, Disk I/O, PIDs

```bash
docker run -m 256m --cpus=0.5 nginx  # Limit RAM and CPU
```

---

## Why Pseudo-Isolation?

**Limitations:**
- Share host kernel
- Processes visible in host `ps -ef`
- Root in container ≠ real root
- Can escape under kernel vulnerabilities

**Result:** Process-level isolation, NOT OS-level like VMs.

---

## Container Lifecycle

Container lifecycle = PID 1 lifecycle

**PID 1 exits → shim detects → writable layer deleted → networking/cgroups cleaned → container gone**

---

## Why Fast & Lightweight?

**Fast:** No OS boot, just `runc → clone → process` (~50ms)
**Lightweight:** Share host kernel/OS, only isolate filesystem + processes

---

## Port Mapping & Cross-Platform

```bash
docker run -d -p 80:80 nginx  # host:container (one host port per container)
```

**Linux**: Native | **Windows/MacOS**: Requires VM

---

## Key Behavior

- Container dies when main process exits
- Processes visible: inside with `docker exec`, on host with `ps -ef`
- Containers: shared kernel, namespaces | VMs: separate kernel, strong isolation

---

## Essential Commands

```bash
# Install
curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh

# Run
docker run -d -p 80:80 nginx              # Detached with port
docker run -m 256m --cpus=0.5 nginx       # With limits
docker run -it ubuntu bash                # Interactive

# View
docker ps           # Running
docker ps -a        # All

# Inspect
docker inspect <container>
docker logs <container>
docker exec -it <container> bash
```

---

## Summary

1. Container = process + isolation (not VM)
2. Isolation: namespaces (PID, NET, MNT, UTS, IPC, USER)
3. Limits: cgroups (CPU, memory, I/O)
4. Architecture: docker → containerd → shim → runc → process
5. PID 1 dies → container stops
6. Ephemeral: writable layer deleted on stop
7. Fast: no OS boot, share kernel (~50ms startup)
8. Lightweight: share host kernel/OS
9. Pseudo-isolation: processes visible on host
10. Persist data: use volumes
