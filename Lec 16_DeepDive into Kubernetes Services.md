# Lec 16: Deep Dive into Kubernetes Services & Networking

---

## Kubernetes Networking Model

**Kubernetes imposes 3 hard rules:**
1. All Pods can talk to all other Pods WITHOUT NAT
2. All Nodes can talk to all Pods WITHOUT NAT
3. Pod's IP is same inside and outside the Pod

**This means Kubernetes must solve:**
- Container-to-Container communication
- Pod-to-Pod communication
- Pod-to-Service communication
- Internet-to-Service communication

---

## Container-to-Container (Within Same Pod)

**Key concept:** Containers in same pod share network namespace.

**What they share:**
- Same IP address
- Same ports
- Same loopback interface

**Communication:** Use `localhost` and port numbers.

**Example:** Container A talks to Container B on `localhost:8080`

**Analogy:** Like multiple apps on your computer using loopback interface.

---

## Pod-to-Pod (Same Node)

**Setup:** Each pod gets its own network namespace.

**How it works:**
1. Pod's namespace uses **veth pair**: Pod eth0 ↔ vethXXX ↔ root namespace
2. All veth interfaces connect to **Linux bridge** (like cbr0)
3. Bridge uses ARP to find destination pod
4. Traffic forwarded to destination veth peer

**Flow:**
```
Pod A → eth0 → veth → bridge → veth → eth0 → Pod B
```

**Managed by:** CNI (Container Network Interface) plugins

---

## Pod-to-Pod (Different Nodes)

**Key idea:** Each node gets a Pod CIDR range.

**Example:**
- Node1: 10.0.1.0/24
- Node2: 10.0.2.0/24

**Flow:**
1. Pod → veth → bridge
2. Bridge can't find destination (different subnet) → routes to node's eth0
3. Node sends packet over network to destination node
4. Destination node receives → routes to pod via veth → namespace

**CNI handles routing:** Different plugins use different methods (overlays, BGP, VPC routing).

**Popular CNI plugins:**
- **AWS VPC CNI:** Assigns real VPC IPs to pods (no overlay)
- **Calico:** BGP-based routing
- **Flannel:** Overlay network
- **Weave:** Mesh network

---

## Pod-to-Service Networking

**Problem:** Pods come and go. IPs change frequently.

**Solution:** Service provides:
- Stable virtual IP (ClusterIP)
- Load balancer inside cluster
- DNS name

### How Service Routing Works

**Two implementations:**

**1. iptables (classic):**
- kube-proxy writes iptables rules
- Watches for packets to ServiceIP
- Randomly picks backend pod
- DNAT (Destination NAT) to selected pod
- Return traffic SNAT'ed back to Service IP

**2. IPVS (modern, default):**
- Kernel-level load balancing
- More scalable, faster
- Creates dummy interface and binds Service IP
- Better for large clusters

**Process:**
1. Service created with selector
2. Kube-proxy watches API
3. Updates iptables/IPVS rules
4. Traffic to ClusterIP → load balanced to pods

---

## Service Discovery (CoreDNS)

**Every service gets DNS record:**
```
<service>.<namespace>.svc.cluster.local
```

**Example:**
```
myservice.default.svc.cluster.local
```

**How it works:**
- CoreDNS runs as pod + service inside cluster
- Watches API for service changes
- Updates DNS records automatically
- Pods query CoreDNS for service names

**Short form (within same namespace):**
```
curl http://myservice
```

---

## Egress (Pod to Internet)

**Problem:** Internet Gateway knows only Node IPs, not Pod IPs.

**Solution:** iptables SNAT (Source NAT)

**Flow:**
1. Pod sends packet to internet (source: Pod IP)
2. iptables SNAT: Pod IP → Node IP
3. AWS Internet Gateway: Node private IP → public IP
4. Packet reaches internet

**Return traffic:** Reverse NAT using conntrack.

---

## Ingress (Internet to Service)

**Two methods:**

### 1. LoadBalancer Service (Layer 4)

**Cloud provider creates:**
- AWS ELB/NLB
- GCP Load Balancer
- Azure Load Balancer

**Flow:**
```
Internet → Cloud LB → NodePort → Service → Pods
```

**Good for:** Simple TCP/UDP services.

### 2. Ingress Controller (Layer 7)

**Examples:** NGINX, AWS ALB, Traefik, Istio Gateway

**Provides:**
- Host-based routing (app1.com, app2.com)
- Path-based routing (/api, /web)
- SSL termination
- Advanced routing rules

**Flow:**
```
Internet → Ingress Controller → Service → Pods
```

---

## Kube-Proxy Role

**Runs on every node.**

**Responsibilities:**
- Watches API for service/endpoint changes
- Updates iptables/IPVS rules
- Implements service load balancing
- Maintains network rules

**Modes:**
- **iptables:** Default, mature
- **IPVS:** Better performance, scales better
- **userspace:** Legacy, not used

---

## Endpoints

**What are they?** Dynamic list of pod IPs backing a service.

**Created automatically** when service matches pods via selector.

**Check endpoints:**
```bash
kubectl get endpoints myservice
```

**Shows:** IP:Port of each pod backing the service.

**Important:** Service without matching pods = no endpoints = traffic fails.

---

## Networking Summary Table

| Layer | What Happens |
|-------|-------------|
| **Container-to-Container** | Share network namespace, use localhost |
| **Pod-to-Pod (same node)** | veth pair → bridge → veth |
| **Pod-to-Pod (cross node)** | Routed through network based on Pod CIDR |
| **Pod-to-Service** | iptables/IPVS DNAT to pod |
| **DNS** | CoreDNS resolves service names |
| **Egress** | SNAT: Pod IP → Node IP → Internet |
| **Ingress** | LoadBalancer or Ingress Controller |

---

## Important Kubernetes Components

**API Server:**
- Central control plane
- Exposes REST API
- Backed by etcd

**Kubelet:**
- Runs on each node
- Configures container runtime
- Manages pod networking

**Kube-Proxy:**
- Runs on each node
- Maintains network rules
- Implements service abstraction

**CoreDNS:**
- Cluster DNS server
- Service discovery
- Runs as deployment in kube-system

**CNI Plugin:**
- Sets up pod networking
- Assigns IP addresses
- Configures routes

---

## Commands

```bash
kubectl get svc                           # List services
kubectl describe svc myservice            # Service details
kubectl get endpoints myservice           # Pod IPs behind service
kubectl get pods -o wide                  # Pod IPs and nodes
kubectl logs -n kube-system coredns-xxx   # CoreDNS logs
kubectl get pods -n kube-system           # System components

# Check networking
kubectl run test --image=busybox --rm -it -- sh
# Inside pod:
nslookup myservice
wget -O- http://myservice
```

---

## Key Takeaways

- Pods are ephemeral, Services provide stability
- **CNI plugins** handle pod networking (IP assignment, routing)
- **Kube-proxy** implements service load balancing (iptables/IPVS)
- **CoreDNS** provides service discovery via DNS
- Services abstract pod interfaces with stable ClusterIP
- Three communication patterns: container-to-container, pod-to-pod, pod-to-service
- Networking ensures high availability and load balancing
- All components work together: API server, kubelet, kube-proxy, CoreDNS, CNI
- Understanding networking is crucial for troubleshooting K8s clusters
