# Lec 17: Horizontal Pod Autoscaler (HPA)

---

## What is HPA?

**HPA** = Horizontal Pod Autoscaler

Automatically scales number of pod replicas based on CPU/memory usage.

**Horizontal scaling:** Add more pods (not increase pod size).

**Example:** 1 pod at 80% CPU → HPA creates more pods → load distributed.

---

## Why HPA?

Manual scaling not practical. Can't constantly monitor and react to traffic spikes.

**HPA watches metrics and scales automatically.**

Real-world: E-commerce during sale → traffic spikes 10x → HPA scales automatically.

---

## How HPA Works

**Process:**
1. Metrics Server collects CPU/memory from pods
2. HPA controller queries metrics
3. Calculates: `Current Usage / Target Usage`
4. Scale up if ratio > 1, scale down if ratio < 1
5. Updates Deployment replica count

**Formula:**
```
Desired Replicas = Current Replicas × (Current Metric / Target Metric)
```

**Example:** 2 pods at 100% CPU, target 50% → Desired = 2 × (100/50) = 4 pods

---

## Prerequisites: Metrics Server

**Mandatory.** HPA needs live CPU/memory data.

**Install:**
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

**Verify:**
```bash
kubectl get pods -n kube-system | grep metrics-server
kubectl top nodes
```

---

## Step 1: Create Deployment

**Important:** Must specify `resources.requests.cpu` (HPA uses this as baseline).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hpa-demo
  template:
    metadata:
      labels:
        app: hpa-demo
    spec:
      containers:
      - name: app
        image: k8s.gcr.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 200m
          limits:
            cpu: 500m
```

```bash
kubectl apply -f deployment.yaml
```

---

## Step 2: Create Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hpa-demo
spec:
  ports:
  - port: 80
  selector:
    app: hpa-demo
```

```bash
kubectl apply -f service.yaml
```

---

## Step 3: Create HPA

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-demo
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-demo
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

**What this does:**
- Never go below 1 pod
- Never exceed 10 pods
- Maintain average 50% CPU across pods

```bash
kubectl apply -f hpa.yaml
kubectl get hpa
```

---

## Step 4: Generate Load

```bash
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- \
  /bin/sh -c "while sleep 0.01; do wget -q -O- http://hpa-demo; done"
```

Continuously hits service, increasing CPU load.

---

## Step 5: Watch Scaling

```bash
kubectl get hpa -w
watch kubectl get pods
```

**What happens:**
1. CPU rises to 80-100%
2. HPA calculates needed replicas
3. Deployment scaled to multiple pods
4. New pods created
5. CPU distributed, average drops to ~50%

---

## Scaling Behavior

**Scale Up:** Fast (30 seconds), immediate response to load

**Scale Down:** Slow (5 minutes cooldown), prevents flapping

---

## Commands

```bash
# Monitor
kubectl get hpa
kubectl describe hpa hpa-demo
kubectl top pods

# Load test
kubectl run load-generator --image=busybox --rm -it -- \
  /bin/sh -c "while sleep 0.01; do wget -q -O- http://hpa-demo; done"

# Manual scale (for comparison)
kubectl scale deployment hpa-demo --replicas=5

# Events
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

## Common Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| CPU shows 0% | Metrics Server missing | Install metrics-server |
| No scaling | CPU request missing | Add `resources.requests.cpu` |
| HPA not working | Wrong selector | Match labels |

---

## Scaling Types

**Horizontal (HPA):** Adds more pods (best for stateless apps)

**Vertical (VPA):** Increases pod resources (best for stateful apps)

**Cluster (CA):** Adds more nodes

---

## Important Notes

- Metrics Server mandatory for HPA
- CPU requests required in deployment
- Scale up: fast, scale down: slow (prevents flapping)
- Default cooldown: 5 minutes
- Works with Deployments, ReplicaSets, StatefulSets
- Can use custom metrics (memory, Prometheus)

---
