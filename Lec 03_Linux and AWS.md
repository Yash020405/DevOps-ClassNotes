# CI/CD, Containers, and Deployment – Class Notes

## Introduction to CI/CD

### What is CI/CD?
CI/CD stands for **Continuous Integration and Continuous Deployment**.

- It is a practice where applications are:
  - Automatically built
  - Tested
  - Deployed
- Goal:
  - Faster integration of code changes
  - Minimal manual intervention
  - Reliable deployments

---

## Continuous Integration (CI)

### CI Trigger
- When a user checks in code to a repository
- An automated pipeline is triggered

---

### CI Pipeline Steps

1. **Linting**
   - Ensures code quality
   - Checks syntax and basic errors
   - Some organizations skip it, but it is recommended

2. **Building**
   - Downloads required packages
   - Compiles and packages the code
   - Can include unit tests if configured

3. **Unit Testing**
   - Tests individual units of the application
   - Mocking is used when functions depend on external calls

4. **SCA and SAST**
   - SCA checks third-party and open-source dependencies
   - SAST scans source code for vulnerabilities

---

## Artifacts in CI

- Artifacts are **binary files** generated during the build process
- Examples:
  - JAR
  - WAR
  - EXE
  - Docker image
- Artifacts are used for testing and deployment

---

## Introduction to Containers and Docker

### Dockerization
- Docker packages applications into containers
- A **Docker image** is a static artifact
- The same image can be deployed across environments

---

### Docker Image Creation
- Docker images are built in layers
- Starts from a base image
- Application layers are added on top
- Each layer is cached to improve build speed

---

### Pushing Docker Images
- After creation, Docker images are pushed to an artifact repository
- Used for versioning and deployment

---

## Continuous Deployment (CD)

### Deployment Pipeline
- Deployment starts from the artifact repository
- Can be:
  - A continuation of CI
  - A separate pipeline

---

### System Integration Testing (SIT)
- First environment where deployment happens
- All modules are tested together
- Focuses on system-level behavior, not individual components

---

## IP Addresses in Deployment

### Public IP
- Temporarily assigned
- Changes when an instance is stopped or started

### Elastic IP
- Remains constant
- Does not change across restarts
- Useful for fixed endpoints in production

---

## Summary

- CI/CD automates integration and deployment
- Docker enables portable and consistent deployments
- Artifacts are central to CI/CD pipelines
- Elastic IPs help maintain stable access for redeployed services