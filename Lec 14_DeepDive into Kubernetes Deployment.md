# Lec 14: Deep Dive into Kubernetes Deployment

---

## What is Deployment?

**Deployment** = highest controller for managing apps in production.

**Hierarchy:** Deployment → ReplicaSet → Pods → Containers

**Important:** Never manage Pods directly. Always use Deployment.

---

## Deployment vs ReplicaSet

YAML looks identical. Only difference: `kind: Deployment` vs `kind: ReplicaSet`

**Why?** Deployment wraps ReplicaSet and adds:
- Rolling updates
- Rollback
- Revision history
- Pause/resume

ReplicaSet can't do updates or rollbacks.

---

## Advantage Chain

**Container crashes** → kubelet restarts  
**Pod crashes** → RS recreates  
**Node crashes** → RS recreates on another node  
**Need updates/rollback** → Deployment handles it

Each layer inherits advantages from below and adds more.

---

## Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: app
        image: nginx:1.27
        ports:
        - containerPort: 80
```

**Important:** Template must match selector labels.

---

## Creation Flow

**Flow:** kubectl apply → API validates → etcd stores Deployment → Deployment Controller creates RS → RS Controller creates Pods → Scheduler assigns to nodes → Kubelet starts containers

**Storage:** All state stored in etcd at `/registry/deployments/`, `/registry/replicasets/`, `/registry/pods/`

---

## Rolling Update Strategy

```yaml
strategy:
  type: RollingUpdate  # default
  rollingUpdate:
    maxUnavailable: 25%
    maxSurge: 25%
```

**Result:** Gradual replacement. Zero downtime.

**Recreate strategy:** Delete all first, then create new. Used for stateful apps.

---

## Updating Deployment

```bash
kubectl set image deployment/webapp app=nginx:1.28
```

**What happens:** New RS created → old RS scaled down (kept for rollback) → new pods started gradually → revision incremented

---

## Rollout Commands

```bash
kubectl rollout status deployment/webapp              # Check rollout
kubectl rollout history deployment/webapp             # View revisions
kubectl rollout undo deployment/webapp                # Rollback
kubectl rollout undo deployment/webapp --to-revision=2  # Specific version
kubectl rollout pause deployment/webapp               # Pause (for canary)
kubectl rollout resume deployment/webapp              # Resume
```

---

## Scaling

```bash
kubectl scale deployment webapp --replicas=10
```

**Updates:** Deployment → ReplicaSet → Pods created automatically.

**Scaling types:**
- **HPA** (Horizontal Pod Autoscaler): scales pod count by CPU/memory
- **VPA** (Vertical Pod Autoscaler): changes CPU/RAM limits
- **CA** (Cluster Autoscaler): adds/removes nodes

---

## Revision & Rollback

**Tracking:** Every update increments `deployment.kubernetes.io/revision` annotation.

**How it works:** New RS created for each update. Old RS scaled down but kept.

**Rollback:** Point Deployment to older RS and scale it up.

---

## Reconciliation Loop

**Process:** Controller continuously compares desired state (etcd) vs actual state (kubelet reports).

**Mismatch?** Controller fixes it immediately.

**Example:** Pod dies → RS sees desired=3, actual=2 → creates new pod.

This is K8s self-healing.

---

## etcd Storage

**Deployment:** `/registry/deployments/default/webapp`  
**ReplicaSet:** `/registry/replicasets/default/webapp-xxxxx`  
**Pod:** `/registry/pods/default/webapp-xxxxx-yyyyy`

---

## Cluster Setup

```bash
hostnamectl set-hostname controlplane     # Change hostname
kubeadm init                              # Init master (starts API, scheduler, etcd)
kubectl get nodes                         # Shows NotReady (no CNI)
kubectl apply -f weave-daemonset.yaml    # Install Weave CNI
watch kubectl get pod -n kube-system     # Wait for CoreDNS
```

---

## Working with Deployment

```bash
kubectl apply -f deployment.yaml          # Create
kubectl get po,rs,deploy                  # View all
watch -n 1 kubectl get po,rs,deploy      # Watch continuously
kubectl scale deploy webapp --replicas=5  # Scale
kubectl delete deployment webapp          # Delete (deletes RS and pods)
```

---

## Important Notes

- Deployment **owns ReplicaSets**
- Each update **creates new RS**, old RS scaled down (not deleted)
- **Selector must match** template labels
- All stored in **etcd**
- Rollback just **points to older RS**
- Use **Deployments for stateless apps**, not raw RS or Pods
- **Declarative approach** recommended over imperative commands
