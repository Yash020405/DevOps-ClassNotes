# Lecture 13: Deep Dive into Kubernetes ReplicaSet

## K8s YAML Structure

**4 components:** apiVersion, kind, metadata, spec

**Data types:** Key-value, List (-), Map (nested)

---

## Pod Recap

Pod crashes → K8s replaces automatically (like movie hero dying and being replaced!)

---

## What is a ReplicaSet?

**Controller ensuring specified number of pod replicas always running.**

ReplicaSet continuously compares:
```
Desired state (spec.replicas)
vs
Current state (actual running pods)
```

**Mismatch?**
- Create new pods
- Delete excess pods
- Replace failed pods

---

## ReplicaSet YAML

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```

**Key parts:**

**replicas:** How many pods you want (e.g., 3)

**selector:** Matches pods with specific labels
- Uses `matchLabels`
- Must match template labels

**template:** Pod definition
- Includes labels
- Container specs

**Important:** Selector labels MUST match template labels!

---

## Creation Flow

**1. kubectl apply** → API Server  
**2. API Server** → validates, writes to etcd  
**3. RS Controller** → sees desired=3, current=0 → creates 3 pods  
**4. etcd stores** pods with `ownerReferences` pointing to RS  
**5. Scheduler** → assigns pods to nodes  
**6. Kubelet** → pulls image, creates containers

**Reconciliation loop:**
```
List pods matching selector
Compare: desired vs current
Mismatch? Create/delete pods
```

---

## Failure Scenarios

**Pod crashes:** Kubelet updates status → RS creates new pod

**Node dies:** After ~5 min → RS deletes unknown pods → creates replacements

**Manual delete:** `kubectl delete pod` → RS immediately creates new pod (different name)

---

## ReplicaSet vs Pod

**Creating raw pod:**
```bash
kubectl run nginx --image=nginx
```
- Pod dies → stays dead (no auto-recovery)

**Creating via ReplicaSet:**
```yaml
replicas: 1
```
- Pod dies → ReplicaSet creates new one

**Key difference:** ReplicaSet provides **self-healing**.

---

## Labels & Selectors

Labels = key-value pairs for organizing objects.

**Why important?**
- ReplicaSet uses selector to find its pods
- Services use selectors to route traffic
- LLabels & Selectors

Labels = key-value pairs for organizing objects (like AWS tags).

**Selector matches pods:**
```yaml
selector:
  matchLabels:
    app: web
```

RS manages all pods with matching labels.

**Important:** Selector MUST match template labels!
    ├── node2
    └── node3


---

## Component Roles

| Component | Role |
|-----------|------|
| **kubectl** | Sends YAML to API server |
| **API Server** | Validates, stores in etcd |
| **etcd** | Stores desired state & actual status |
| **ReplicaSet Controller** | Creates/deletes pods to match desired count |
| **Scheduler** | Assigns pods to nodes |
| **Kubelet** | Creates containers, reports status |
| **Container Runtime** | Actually runs containers |

**ReplicaSets never run pods** - they only ensure correct count!

---

## Commands

```bash
# Create ReplicaSet
kubectl apply -f replicaset.yaml

# List ReplicaSets
kubectl get rs
kubectl get replicasets

# Describe ReplicaSet
kubectl describe rs nginx-rs

# List pods
kubectl get pods

# Check pod labels
kubectl get pods --show-labels

# Filter by label
kubectl get pods -l app=web

# Scale ReplicaSet
kubectl scale rs nginx-rs --replicas=5

# Delete ReplicaSet (deletes pods too)
kubectl delete rs nginx-rs

# Delete ReplicaSet but keep pods
kubectl delete rs nginx-rs --cascade=orphan
```
---

**Components:** API Server, etcd, RS Controller, Scheduler, Kubelet, Runtime

**Remember:** ReplicaSets only ensure count, Deployments add update management!
- RS is a controller (control plane), never runs pods
- Label selector must match template
- ownerReferences link pods to RS
- Pods ephemeral - new names/IPs on replacement
- **Use Deployments** instead (better features: rolling updates, rollback)