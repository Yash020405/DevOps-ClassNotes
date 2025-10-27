# Microservices – Class Notes

## 1. Monolithic vs Microservices

- **Monolithic Architecture**
  - Everything is in a single codebase
  - Tightly coupled
  - Harder to scale

- **Microservices Architecture**
  - System is divided into multiple small, independent services
  - Each service can scale individually

---

## 2. DNS (Domain Name System)

- DNS maps a domain name to an IP address
- When a user enters a domain name:
  - The browser contacts the DNS server
  - DNS returns the IP address of the machine hosting the service

---

## 3. Load Balancers

- Load balancers distribute incoming traffic among multiple service instances

### Types of Routing
- **Path-based routing**
  - Example: `/login → Authentication Service`
- **Instance-based routing**
  - Decides which service instance should handle the request

### Additional Responsibilities
- Service discovery (adding/removing instances dynamically)
- Traffic distribution across instances

---

## 4. Authentication vs Authorization

- **Authentication**
  - Identity check
  - “Are you a valid user?”

- **Authorization**
  - Permission check
  - “Are you allowed to perform this action?”

---

## 5. Three Axes of Scalability

- **X-axis** – Horizontal scaling (adding more instances)
- **Y-axis** – Splitting services by functionality
- **Z-axis** – Sharding data or users by region/type

---

## 6. Communication Between Microservices

### Synchronous Communication
- Direct, blocking calls
- Examples: HTTP, gRPC
- Requires a load balancer

### Asynchronous Communication
- Uses messaging systems
- Examples: Kafka, RabbitMQ, Pub/Sub
- Event-based, no direct request-response

---

## 7. API Gateway

- Single entry point for all client requests
- Routes requests to the appropriate microservice
- Can handle authentication, rate limiting, and logging

---

## 8. Logging in Microservices

- Logs should be externalized
- Logs should not be stored locally on services

### Tools Mentioned
- ELK Stack (Elasticsearch, Logstash, Kibana)
- Jaeger / Zipkin (distributed tracing)
- Datadog, Dynatrace, Splunk

- Helps with debugging and tracking across services

---

## 9. Typical Request Flow

DNS → Load Balancer → Authentication / Authorization
→ Redis / Memcache → Service → Database


- Synchronous communication uses load balancers
- Asynchronous communication uses message brokers like Kafka

---

## 10. 12-Factor App Principles (Microservices)

- Best practices for scalable and maintainable services

### Key Points Covered
- Explicitly declare and isolate dependencies
- Keep services stateless
- Store state in external databases or caches
- Externalize logs and configuration

---

## 11. Miscellaneous

- **Dogfooding** – Using your own product for testing
- **Catfooding** – Testing with another team’s product
- **Richardson Maturity Model** – Levels of RESTful API maturity