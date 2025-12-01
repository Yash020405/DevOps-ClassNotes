# Lecture 12: Deep Dive into Kubernetes Pods

## What is a Pod?

**Smallest deployable unit** in Kubernetes.

Pod = one or more containers that are:
- **Co-located:** Share same network/storage
- **Co-scheduled:** Always on same node
- **Co-managed:** Created/deleted together

**Key characteristics:**
- Share network namespace (same IP, localhost communication)
- Share IPC namespace
- Share volumes
- Ephemeral (disposable, not persistent)

---

## Pod Creation Flow

```
kubectl → API Server → etcd → Scheduler → Kubelet → Runtime
```

**Steps:**

**1. kubectl apply:** POST to `/api/v1/namespaces/<ns>/pods`

**2. API Server:**
- AuthN/AuthZ (verify identity + permissions)
- Admission (mutate → validate)
- Write to etcd: `/registry/pods/<ns>/<pod>`

**3. Scheduler:**
- Watch for `spec.nodeName == ""`
- Filter nodes (resources, taints, affinity)
- Score remaining nodes
- Write binding (set nodeName)

**4. Kubelet:**
- Watch API for assigned pods
- CNI → allocate Pod IP
- CRI → create pause container
- Pull images
- Create/start containers (init first, then main)

**5. Status:**
- Kubelet updates `/status` subresource
- Phase, podIP, containerStatuses
- Readiness probe → Pod Ready → added to Service

---

## Control Plane

### API Server
Single entry point - REST API for all operations.

**Does:** AuthN/AuthZ, admission, persist to etcd, watch streams

**Subresources:** `/status`, `/binding`, `/ephemeralcontainers`

### etcd
Key-value store (Raft consensus) at `/registry/pods/<ns>/<pod>`

**Tracks:** resourceVersion (concurrency), spec (desired), status (actual)

**Concurrency:** Optimistic locking, stale writes rejected (409)

### Scheduler
Assigns pods to nodes.

**Process:** Watch unscheduled → Filter (resources/taints/affinity) → Score → Bind

Can preempt lower priority pods.

### Controller Manager
Ensures desired = actual state.

**Controllers:** Node, ReplicaSet, Deployment, Endpoints, Job

**Loop:** Watch → Read desired → Read actual → Reconcile

### Cloud Controller
Cloud integration (AWS/Azure/GCP): Load balancers, volumes, routes

---

## Data Plane

### Kubelet
Node agent - watches API, calls CNI/CRI/CSI, reports status, runs probes, restarts containers.

### Container Runtime
Executes containers (containerd, CRI-O) via CRI.

**Operations:** RunPodSandbox, PullImage, CreateContainer, StartContainer

### Kube-Proxy
Programs iptables/ipvs for Service → Pod routing. Modes: iptables, ipvs.

### CNI Plugin
Allocates Pod IP, creates veth pair, sets up network namespace.

Popular: Calico, Weave, Cilium, Flannel

### CSI Driver
Storage: CreateVolume → Attach → Mount to pod

---

## Namespaces

Logical isolation within cluster.

**Default namespaces:**
- `default` - user workloads
- `kube-system` - K8s components
- `kube-public` - public accessible
- `kube-node-lease` - node heartbeats

**Usage:**
```bash
kubectl get pods -n kube-system
kubectl create namespace myapp
```

---

## Pod States & Conditions

### Phases
- **Pending:** Accepted but not running yet
- **Running:** At least one container running
- **Succeeded:** All containers terminated successfully
- **Failed:** At least one container failed
- **Unknown:** Cannot determine state

### Conditions
- `PodScheduled` - scheduler assigned node
- `Initialized` - init containers completed
- `Ready` - pod can serve traffic
- `ContainersReady` - all containers ready
- `Unschedulable` - scheduler can't place pod

### Container States
- `Waiting` - pulling image, waiting for init
- `Running` - executing
- `Terminated` - finished or crashed

---

## Admission Controllers

**Mutating (first):** Modify pod - inject sidecars, add defaults, imagePullSecrets

**Validating (after):** Accept/reject - ResourceQuota, PodSecurity, custom rules

**Important:** Final pod may differ from original YAML!

---

## Networking

**Pod IP:** CNI creates veth pair, assigns IP, kubelet updates status

**Service Discovery:** CoreDNS resolves `myapp.default.svc.cluster.local` → ClusterIP

**Endpoints:** Controller watches labeled pods, updates Endpoints (only Ready pods)

**Flow:** Pod A → Service → kube-proxy (iptables) → Pod B

---

## Storage

**Types:** emptyDir (temp), hostPath (node fs), PersistentVolume (durable)

**CSI Flow:** PVC created → Provision → Attach to node → Mount to pod

---

## Init Containers

Run **before** main containers (sequential). Must succeed. Use for dependencies, setup, migrations.

---

## Lifecycle & Probes

**PostStart:** Runs after container starts  
**PreStop:** Runs before termination

**Graceful termination:** Delete → PreStop → SIGTERM → wait 30s → SIGKILL

**Liveness:** Restart if fails  
**Readiness:** Remove from Service if fails  
**Startup:** For slow-starting containers

**Types:** HTTP, TCP, Exec

---

## Debugging Commands

```bash
# List pods
kubectl get pods -A
kubectl get pods -o wide  # with IPs/nodes

# Pod details
kubectl describe pod <name>
kubectl get pod <name> -o yaml

# Events
kubectl get events --sort-by='.lastTimestamp'
kubectl get events --field-selector involvedObject.name=<pod>

# Logs
kubectl logs <pod>
kubectl logs <pod> -c <container>
kubectl logs <pod> --previous  # crashed container

# Exec into pod
kubectl exec -it <pod> -- /bin/bash

# Check scheduling
kubectl get pods --field-selector=status.phase=Pending

# Endpoints
kubectl get endpoints <service>

# Node details
kubectl describe node <node>
```

---

## Common Failures

**Pending:** Can't schedule - check resources/taints/affinity via `kubectl describe pod`

**ImagePullBackOff:** Can't pull image - check name, registry, pullSecrets

**CrashLoopBackOff:** Container crashing - check `kubectl logs <pod> --previous`

**ContainerCreating:** CNI/volume issue - check events in `describe pod`

**Not Ready:** Readiness probe failing - check probe config and app logs

**Node NotReady:** Kubelet issue - check `kubectl describe node`, kubelet status

---

## Resource Version

**resourceVersion** = etcd modRevision for watches & optimistic concurrency.

Example: Created (v10) → Bound (v12) → Status (v14)

Watch events: ADDED, MODIFIED, DELETED with versions.

---

## Setup Commands

```bash
sudo su
set-hostname control-plane
kubeadm init
kubectl apply -f weave.yaml  # CNI
kubectl get nodes
kubectl get pods -A
```

---

## Best Practices

1. Use Deployments/StatefulSets (not raw pods)
2. Set resource requests/limits
3. Use readiness probes
4. Use liveness probes
5. Define affinity rules
6. Use PodDisruptionBudget
7. Use `/status` subresource
8. Use init containers for setup
9. Label everything
10. Use namespaces for isolation

---

## Architecture Recap

```
       User (kubectl)
            ↓
       API Server
      /     |     \
   etcd  Scheduler  Controller
            |
      (assigns node)
            ↓
        Kubelet (on node)
       /    |    \
     CNI   CRI   CSI
      |     |     |
   Network Pod  Volume
```

---

## Summary

**Pod = smallest unit:** Co-located containers sharing network/storage

**Creation flow:** kubectl → API → etcd → Scheduler → Kubelet → Runtime

**Control plane:** API Server (gateway), etcd (storage), Scheduler (placement), Controllers (reconciliation)

**Data plane:** Kubelet (agent), Runtime (containers), Kube-proxy (networking), CNI (pod IPs), CSI (volumes)

**Key concepts:** ResourceVersion (concurrency), Admission (mutate/validate), Probes (health), Init containers (setup)

**Debug:** describe/logs/events commands, check phases/conditions/events