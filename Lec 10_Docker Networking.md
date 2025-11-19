# Lecture 10: Docker Networking

## Basic Networking Stuff

### IP Address
Unique address for device on network.
- IPv4 = 32-bit, written as `192.168.10.24`
- Each number: 0-255

### NIC (Network Interface Card)
Hardware connecting computer to network.
- Has MAC address
- Everything (browsers, containers) sends traffic through NIC

### Switch
Layer-2 device connecting devices in LAN.
- Forwards frames using MAC addresses
- Maintains MAC table

### ARP Protocol
Maps IP address to MAC address.

Process:
```
A: "Who has 192.168.1.20?"
B: "I'm 192.168.1.20, MAC = aa:bb:cc:dd"
A: Stores in cache
```

### Gateway
Router to reach networks outside your subnet.

Example: Your IP `192.168.1.10/24` needs `8.8.8.8` → traffic goes to gateway `192.168.1.1`

### NAT
**SNAT:** Private IP → Public IP (outgoing)  
**DNAT:** Public request → Private IP (incoming, port forwarding)

---

## Subnetting

Dividing large network into smaller ones.

### Math
`192.168.10.0/24` → 32-24 = 8 → 2^8 = 256 IPs

### Subnet Table
| Subnet | IPs | Hosts |
|--------|-----|-------|
| /24 | 256 | 254 |
| /25 | 128 | 126 |
| /26 | 64 | 62 |
| /27 | 32 | 30 |
| /28 | 16 | 14 |

### Example: 3 Networks (Eng, HR, Reception)
Divide /24 into /26 subnets (64 IPs each):

- **Engineering:** `192.168.10.0/26` (0-63)
- **HR:** `192.168.10.64/26` (64-127)
- **Reception:** `192.168.10.128/26` (128-191)

---

## Docker Networking

Docker uses: namespaces, veth pairs, bridges, iptables.

### Network Namespace
Each container gets isolated:
- Own `eth0`
- Own routing table
- Own firewall rules

### veth Pairs
Virtual cable connecting container to Docker bridge.
```
[Container eth0] <----> [vethXYZ] ---- (docker0)
```

### docker0 Bridge
Default bridge: `172.17.0.1/16`
- Containers talk to each other
- NATs traffic to internet

---

## Network Types

| Type | Use Case |
|------|----------|
| **bridge** | Default, local communication |
| **host** | Shares host network |
| **none** | No networking |
| **overlay** | Multi-host (Swarm) |
| **macvlan** | Real LAN devices |

### Bridge Network
```bash
docker network create mynet
docker run -d --network mynet --name app1 nginx
docker run -d --network mynet --name app2 busybox
```
Containers can ping by name: `ping app1`

### Host Network
```bash
docker run --network host nginx
```
Best performance, no isolation.

### None Network
```bash
docker run --network none alpine
```
Total isolation.

### Overlay Network
```bash
docker network create -d overlay myoverlay
```
Multi-host communication.

---

## Container Internet Access

```
Container → docker0 → host eth0 (NAT) → Internet
```

Docker uses iptables to NAT container traffic.

---

## Port Mapping

```bash
docker run -p 8080:80 nginx
```
Host 8080 → Container 80 (uses DNAT)

---

## Network Commands

```bash
docker network ls                    # List networks
docker network inspect bridge        # Inspect
docker network create mynet          # Create
docker network rm mynet              # Remove
docker inspect <container>           # Container details
```

---

## Custom Bridge vs Default

**Default:** No DNS, must use IPs  
**Custom:** Built-in DNS, use container names (recommended!)

---

## Practical: App + DB

```bash
docker network create backend
docker run -d --name db --network backend mysql
docker run -d --name app --network backend myapp
```

App connects: `db:3306` (hostname = container name)

---

## Architecture

```
         Internet
            |
      Host eth0 (NAT)
            |
      docker0 (172.17.0.1)
       /     |     \
     C1     C2     C3
```

**Container-to-container:** via docker0  
**Container-to-internet:** docker0 → NAT → internet

---

## Important Notes

- Network ID = first IP (reserved)
- Broadcast ID = last IP (reserved)
- `/32` = 1 IP, `/31` = 2 IPs, `/30` = 4 IPs
- Subnet must start from aligned block

---

## Quick Calculations

```
/24 → 2^8 = 256 IPs
/26 → 2^6 = 64 IPs
/28 → 2^4 = 16 IPs
```

Example: `192.168.10.0/23` → 512 IPs (range: 192.168.10.0 to 192.168.11.255)

---

## Debugging

```bash
docker inspect <container> | grep IPAddress    # Get IP
docker exec <container> route -n               # Check routing
docker exec <container> ping <other>           # Test connectivity
```

---

## Summary

**Networking Basics:** IP, NIC, Switch, ARP, Gateway, NAT  
**Subnetting:** Formula `2^(32-prefix)`, divide networks  
**Docker:** Namespaces, veth pairs, docker0 bridge  
**Network Types:** bridge, host, none, overlay, macvlan  
**Best Practice:** Use custom bridges with DNS
