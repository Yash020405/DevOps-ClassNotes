# Lecture 11: Kubernetes

## What is Kubernetes?

Kubernetes (K8s) = container orchestrator managing containers at scale.

**Does:** Deploys/scales/heals, networking, storage, updates, load balancing

Think: "OS for your data center."

---

## Key Features

**1. Automatic Bin Packing**
- Create **pods** (not containers directly)
- K8s decides node placement, optimal utilization

**2. Self-Healing**
- Pod crashes → auto-replaced
- Desired = actual state

**3. Scaling**
- HPA: Scale pods (CPU/memory)
- VPA: Adjust resources
- Cluster Autoscaler: Add/remove nodes

**4. Load Balancing**
- Services provide stable endpoints
- Pods use DNS/ClusterIP

**5. Jobs & Secrets**
- Jobs/CronJobs for tasks
- Secrets for passwords/tokens/certs

---

## Architecture

**Two types of nodes:**
1. **Control Plane** (brain) - manages
2. **Worker Nodes** (muscles) - run pods

```
    Control Plane
   [API | etcd | Scheduler | Controller]
            |
      Worker Nodes
     [Kubelet | Proxy | Runtime]
        (Pods run here)
```

---

## Control Plane

### API Server
Central entry point for everything in K8s.

**What it does:**
- Accepts REST requests from kubectl
- Authentication (who are you?)
- Authorization (what can you do?)
- Validation (is request valid?)
- Writes/reads from etcd
- Notifies scheduler/controller
- Stateless (can run multiple replicas)

Everything flows through API server.

### etcd
Distributed key-value store (uses Raft consensus).

**Stores:**
- All cluster data (nodes, pods, configs, secrets)
- Desired state
- Current state

**Critical:** If lost, cluster is gone. Backup regularly!

### Scheduler
Decides **which node** a pod should run on.

**Doesn't run pods** - just picks the best node.

**Considers:**
- CPU/memory available
- Taints & tolerations
- Node affinity
- Pod topology
- Existing workloads

**Event-driven:** etcd triggers it (no manual invoke).

### Controller Manager
Collection of control loops ensuring desired = actual.

**Key controllers:**
- Node controller (monitors node health)
- Deployment controller (manages ReplicaSets)
- ReplicaSet controller (maintains pod count)
- Job controller (batch workloads)
- ServiceAccount controller

**Loop:**
```
Read desired from etcd
Read actual from cluster
Mismatch? Fix it
Repeat
```

### Cloud Controller Manager
Integrates with cloud providers (AWS, Azure, GCP).

**Handles:**
- Cloud load balancers (ELB, ALB)
- Storage volumes (EBS, Azure Disk)
- Network routes
- VM lifecycle

---

## Worker Nodes

Where actual workloads run.

### Kubelet
Agent on every node.

**Responsibilities:**
- Communicates with API server
- Ensures containers run in pods
- Monitors pod health
- Reports status back
- Pulls container images
- Mounts volumes
- Executes liveness/readiness probes

### Kube-Proxy
Manages networking rules.

**Does:**
- Maintains iptables/ipvs rules
- Load balances across pod endpoints
- Handles Service → Pod routing
- Enables internal cluster communication

**Modes:**
- iptables (default)
- ipvs (better performance)

### Container Runtime
Executes containers.

**Popular runtimes:**
- **containerd** (most common, CNCF)
- **CRI-O** (Red Hat/OpenShift)
- **Docker** (deprecated as runtime, images work)

**Uses CRI** (Container Runtime Interface).

**Tasks:**
- Pulls images from registry
- Creates container namespaces
- Starts/stops containers
- Manages cgroups for resource limits

### Plugins

**CNI (Container Network Interface):**
- Assigns IP addresses to pods
- Configures routes
- Options: Calico, Weave, Cilium, Flannel

**CSI (Container Storage Interface):**
- Provisions persistent volumes
- Options: EBS, Azure Disk, NFS, Ceph

---

## How K8s Handles Requests

Example: `kubectl run diptippod --image=nginx`

**Flow:**
1. kubectl → API Server (REST)
2. API Server: auth → validation → writes to etcd
3. Scheduler detects unscheduled pod → selects node
4. Kubelet on node → Container Runtime → creates container
5. Pod runs
6. Controller ensures desired = actual state
7. If pod crashes → recreated automatically

---

## What is a Pod?

**Smallest deployable unit** in K8s.

Pod = group of one or more containers running together.

**Containers in same pod share:**
- Network namespace (same IP, communicate via localhost)
- Storage volumes
- IPC namespace
- Lifecycle (created/deleted together)

**Key points:**
- Pods are **ephemeral** (disposable)
- Not meant to be durable
- Get new IPs when recreated
- Managed by higher-level objects (Deployments, StatefulSets)

**Example use case:**
- Main app container + sidecar (logging, proxy)
- Both share same network/storage

---

## K8s Philosophy

**Declarative:** `replicas: 3` → K8s ensures 3 pods always running.

Pod crashes → Controller recreates  
Node dies → Scheduler reschedules

---

## Basic Commands

```bash
# Node operations
kubectl get nodes              # List all nodes
kubectl describe node <name>   # Node details

# Pod operations
kubectl run mypod --image=nginx           # Create pod
kubectl get pods                          # List pods
kubectl get pods -o wide                  # Pods with IPs/nodes
kubectl describe pod <name>               # Pod details
kubectl logs <pod>                        # View logs
kubectl exec -it <pod> -- /bin/bash       # Shell into pod
kubectl delete pod <name>                 # Delete pod

# Cluster info
kubectl cluster-info           # Cluster details
kubectl get all                # All resources
```

---

## Real-World Example

Deploy Nginx with 3 replicas:

```bash
kubectl create deployment nginx --image=nginx --replicas=3
kubectl get deployments
kubectl get pods
```

**What happens:**
1. Deployment creates ReplicaSet
2. ReplicaSet creates 3 pods
3. Scheduler assigns pods to nodes
4. Kubelet starts containers
5. If pod dies, ReplicaSet recreates it

---

## Important Notes

**Control plane on worker nodes:**
> "Data plane components also present on control plane."

**Scheduler:**
- Event-driven, not polling
- etcd change → scheduler reacts
- No manual intervention needed

**etcd criticality:**
- Single source of truth
- Uses Raft protocol (distributed consensus)
- Regular backups essential
- Typically 3 or 5 nodes for HA

**Pods:**
- Not persistent
- IP changes on recreation
- Use Services for stable endpoints

---

## Summary

**Control Plane:** API Server, etcd, Scheduler, Controller  
**Worker Nodes:** Kubelet, Kube-Proxy, Runtime  
**Flow:** kubectl → API → etcd → Scheduler → Kubelet → Runtime  
**Philosophy:** Declarative, self-healing, event-driven  
**Everything through API Server, etcd = source of truth**
