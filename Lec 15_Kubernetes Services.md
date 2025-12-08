# Lec 15: Kubernetes Services

---

## Why Services Exist?

**Problem:** Pods are ephemeral.
- Get new IPs on restart
- Recreated by ReplicaSets with different IPs
- Scale up/down dynamically

**Can't reliably communicate with pods directly.**

**Solution:** Services provide stable virtual IP and DNS name.

---

## What is a Service?

Service = abstraction that provides:
- Stable virtual IP (ClusterIP)
- Stable DNS name
- Load balancing across backend pods
- Traffic routing through kube-proxy
- Access from inside or outside cluster

**Think:** Service is like a permanent phone number that redirects to whoever is available.

---

## How Service Works Internally

**Flow:**
1. Create Service with pod selector: `app: myapp`
2. Kube-proxy creates load-balancing rules (iptables or IPVS)
3. Service gets stable ClusterIP
4. Traffic to ClusterIP → kube-proxy → forwards to matching pods

**Key point:** Pods change, Service IP stays constant.

---

## Service Types

| Type | Purpose | Accessible From |
|------|---------|----------------|
| **ClusterIP** (default) | Internal communication | Inside cluster only |
| **NodePort** | Expose on each node's IP | External (nodeIP:nodePort) |
| **LoadBalancer** | Expose via cloud LB | Internet or VPC |

---

## ClusterIP Service (Default)

**Most common type.** Used for internal service-to-service communication.

**Use cases:**
- Microservices calling each other
- Internal databases
- Frontend to backend communication

**YAML:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 80           # Service port
    targetPort: 8080   # Pod container port
```

**Access inside cluster:**
```bash
curl http://backend-svc
curl http://backend-svc.default.svc.cluster.local  # Full DNS
```

**Key points:**
- `port`: Port exposed by service
- `targetPort`: Pod's container port
- ClusterIP assigned automatically
- Only accessible within cluster

---

## NodePort Service

**Exposes Service on every node's IP at static port.**

**Port range:** 30000-32767

**Traffic flow:** External Client → NodeIP:NodePort → Service → Pods

**Use cases:**
- Non-cloud environments (bare metal)
- Local clusters (kubeadm, kind, minikube)
- Debugging external access

**YAML:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-nodeport
spec:
  type: NodePort
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 32080  # Optional, can be auto-assigned
```

**Access externally:**
```bash
http://<NodeIP>:32080
```

Can hit **any worker node IP**. Kubernetes forwards to right pods.

**Limitations:**
- Exposes service on all nodes (security concern)
- Fixed high port range
- No smart routing (no CDN, health checks)
- Not production-grade for public internet

---

## LoadBalancer Service (Cloud Only)

**Exposes service publicly with cloud load balancer.**

**Supported in:** AWS (ELB/NLB/ALB), GCP, Azure, DigitalOcean

**Traffic flow:** Internet → Cloud Load Balancer → NodePort → Service → Pods

**Note:** LoadBalancer internally still uses NodePort.

**YAML:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-lb
spec:
  type: LoadBalancer
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 8080
```

**Check external IP:**
```bash
kubectl get svc backend-lb
```

Shows `EXTERNAL-IP` assigned by cloud provider.

**Access:**
```bash
http://<EXTERNAL-IP>
```

---

## Service Discovery & Load Balancing

**Service Discovery:** Services discovered via labels on Pods.

**Load Balancing:** Traffic evenly distributed among all matching Pods.

**Example:** If 3 pods match selector `app: backend`, traffic splits equally.

---

## Service DNS Resolution

Every service gets DNS name:
```
<servicename>.<namespace>.svc.cluster.local
```

**Example:**
```
redis.master.svc.cluster.local
backend-svc.default.svc.cluster.local
```

**Short form (within same namespace):**
```
backend-svc
```

---

## Additional Concepts

### Headless Service (Selectorless)

**Set:** `clusterIP: None`

**Used for:**
- StatefulSets
- Databases
- DNS-based discovery without load balancing

### Multi-Port Services

Single service can expose multiple ports:
```yaml
ports:
- name: http
  port: 80
  targetPort: 8080
- name: https
  port: 443
  targetPort: 8443
```

### ExternalName Service

**Used for:** Accessing external DB or service.

```yaml
kind: Service
spec:
  type: ExternalName
  externalName: database.company.com
```

No pod selector. Just DNS alias.

---

## Which Service Type to Use?

| Need | Service Type |
|------|-------------|
| Internal microservices | ClusterIP |
| Local network access (no cloud LB) | NodePort |
| Production external access | LoadBalancer |
| StatefulSets / DBs | Headless |
| Access external DB | ExternalName |

---

## Commands

```bash
kubectl apply -f service.yaml              # Create service
kubectl get svc                            # List services
kubectl describe svc backend-svc           # Details
kubectl get endpoints backend-svc          # See pod IPs behind service
kubectl delete svc backend-svc             # Delete service
```

---

## Important Points

- Pods are ephemeral, Services are stable
- Service uses **selector** to find matching pods
- **Kube-proxy** implements service routing (iptables/IPVS)
- ClusterIP: internal only
- NodePort: exposes on all nodes (30000-32767)
- LoadBalancer: cloud only, gets public IP
- Services provide **load balancing** automatically
- DNS name: `<service>.<namespace>.svc.cluster.local`
- **Endpoints object** stores actual pod IPs for service
