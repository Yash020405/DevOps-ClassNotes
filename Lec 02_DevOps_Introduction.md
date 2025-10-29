# DevOps – Class Notes

## Overview of DevOps

### What is DevOps?
DevOps is defined as a **set of practices, cultural philosophies, and tools** that integrate **software development and IT operations** to deliver applications and services **more rapidly and reliably**.

These three components work together to streamline and improve the delivery and reliability of products within an organization.

---

## Challenges in DevOps

- Cultural change is one of the biggest challenges in DevOps
- Change management is difficult
- Teams with long-established processes often resist adopting new DevOps practices

---

## Key Concepts in DevOps

### DevOps Philosophy
- Application of principles to unify:
  - Software Development (Dev)
  - Software Operations (Ops)

### Shift Left Approach
- Emphasizes integrating testing earlier in the development process
- Helps identify and fix issues sooner in the lifecycle

---

## Security Testing in DevOps

### Types of Security Testing

- **Static Application Security Testing (SAST)**
  - Analyzes application source code
  - Code is not executed
  - Identifies vulnerabilities early

- **Dynamic Application Security Testing (DAST)**
  - Analyzes applications in a running state
  - Identifies runtime vulnerabilities

- **Software Composition Analysis (SCA)**
  - Checks open-source and third-party libraries
  - Detects known vulnerabilities in dependencies

---

### Example of Code Vulnerability

- **SQL Injection**
  - Occurs when inputs are not properly sanitized
- Using Java `Statement` can lead to SQL injection attacks
- Using `PreparedStatement` helps prevent SQL injection by safely handling inputs

---

## Continuous Integration (CI)

### Continuous Integration Process

Steps involved in CI:

1. Code checkout from the repository
2. Dependency installation
3. Linting to ensure code quality
4. Building the application
5. Running unit tests
6. Running security checks (SAST, SCA)
7. Creating build artifacts (e.g., Docker images, binaries)
8. Deploying to a staging environment

---

## Continuous Deployment (CD)

- Deployment continues after successful CI
- CD automates the deployment process
- Manual intervention is required only when:
  - Predefined thresholds fail
  - Issues are detected
- Goal is to achieve smooth and reliable automated deployments

---

## Communication and Coordination in DevOps

### API Gateway
- Routes requests to appropriate microservices
- Handles authorization and throttling

---

### Inter-Service Communication

- **Synchronous Communication**
  - Real-time
  - Blocking calls

- **Asynchronous Communication**
  - Non-blocking
  - Uses message brokers
  - Examples: RabbitMQ, Kafka

---