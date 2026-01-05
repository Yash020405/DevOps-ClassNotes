# Lec 20: Intro to DevSecOps

---

## What is DevSecOps?

**DevSecOps** = DevOps + Security

**Traditional approach:** Security team checks at end (bottleneck).

**DevSecOps approach:** Security automated in CI/CD pipeline from start.

**Shift left:** Find security issues early (during development, not production).

---

## Why DevSecOps?

**Problems with traditional security:**
- Security checks at end = expensive fixes
- Delays deployment
- Manual reviews = slow
- Production vulnerabilities

**DevSecOps benefits:**
- Catch vulnerabilities early
- Automated security checks
- Faster releases
- Reduced risk

---

## Security Layers in CI/CD

**Complete security coverage:**

1. **Code level** - SAST
2. **Dependencies** - SCA
3. **Container** - Image scanning
4. **Runtime** - DAST

---

## SAST (Static Application Security Testing)

**What:** Analyze source code for vulnerabilities WITHOUT running it.

**Scans for:**
- SQL injection
- Command injection
- Hardcoded passwords
- Insecure crypto
- OWASP Top 10 issues

**Tools:** CodeQL, SonarQube, Checkmarx

**When:** During build in CI pipeline.

**Example in GitHub Actions:**
```yaml
- name: CodeQL Analysis
  uses: github/codeql-action/analyze@v2
```

---

## SCA (Software Composition Analysis)

**What:** Scan third-party libraries and dependencies for known vulnerabilities.

**Why important:** 80% of application code is dependencies.

**Scans for:**
- Known CVEs (Common Vulnerabilities and Exposures)
- Outdated packages
- License compliance issues

**Tools:** OWASP Dependency Check, Snyk, WhiteSource

**Example:**
```yaml
- name: OWASP Dependency Check
  uses: dependency-check/Dependency-Check_Action@main
```

---

## Container Image Scanning

**What:** Scan Docker images for vulnerabilities in OS packages and libraries.

**Scans:**
- Base image vulnerabilities
- Installed packages
- Application dependencies inside container

**Tools:** Trivy, Aqua, Clair

**Example:**
```yaml
- name: Trivy Image Scan
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myapp:latest'
    severity: 'CRITICAL,HIGH'
```

**Critical/High vulnerabilities → pipeline fails.**

---

## Linting

**What:** Code quality and style checker.

**Checks:**
- Code standards
- Bad practices
- Potential bugs
- Style violations

**Java tool:** Checkstyle

**Example:**
```yaml
- name: Checkstyle
  run: mvn checkstyle:check
```

---

## DevSecOps Pipeline Example

**Complete flow:**

```yaml
name: DevSecOps Pipeline

on:
  push:
    branches: [master]

jobs:
  security-pipeline:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Java
      uses: actions/setup-java@v3
      with:
        java-version: '11'
    
    - name: Linting
      run: mvn checkstyle:check
    
    - name: SAST - CodeQL
      uses: github/codeql-action/analyze@v2
    
    - name: SCA - Dependency Check
      uses: dependency-check/Dependency-Check_Action@main
    
    - name: Unit Tests
      run: mvn test
    
    - name: Build
      run: mvn clean package
    
    - name: Build Docker Image
      run: docker build -t myapp:latest .
    
    - name: Scan Docker Image
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'myapp:latest'
    
    - name: Push to Registry
      run: docker push myapp:latest
```

---

## Secrets Management

**Never hardcode credentials in code or YAML.**

**GitHub Secrets:**
1. Repo → Settings → Secrets → New secret
2. Add: `DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`

**Use in workflow:**
```yaml
- name: Login to DockerHub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

---

## Security Gates

**Quality gates:** Pipeline fails if security threshold crossed.

**Example:**
```yaml
- name: Trivy Scan
  run: trivy image --severity CRITICAL,HIGH --exit-code 1 myapp:latest
```

**exit-code 1:** Fails pipeline if critical/high vulnerabilities found.

**Prevents insecure code from reaching production.**

---

## What Makes Pipeline "DevSecOps"?

**Regular CI:**
- Build
- Test
- Deploy

**DevSecOps CI:**
- Build
- **Linting**
- **SAST**
- Test
- **SCA**
- Package
- **Container Scan**
- Deploy

Security integrated at every stage.

---

## Key Concepts

**Shift Left:**
- Find bugs early (cheaper to fix)
- Security in development phase

**Supply Chain Security:**
- Scan all dependencies
- Verify third-party code

**Immutable Artifacts:**
- Docker images tagged and versioned
- Same artifact promoted through environments

**Zero Trust:**
- No hardcoded secrets
- Everything authenticated

---

## GitHub Security Integration

**Security alerts visible in GitHub:**

Settings → Security → Code scanning alerts

**Shows:**
- SAST findings (CodeQL)
- SCA findings (Dependency Check)
- Container vulnerabilities (Trivy)

**Centralized vulnerability management.**

---

## Pipeline Permissions

```yaml
permissions:
  contents: read           # Read code
  security-events: write   # Upload security findings
```

Allows pipeline to upload vulnerability reports to GitHub Security tab.

---

## Real-World Example

**Complete DevSecOps workflow:**

1. Developer pushes code
2. GitHub Actions triggers
3. Code linted (Checkstyle)
4. SAST scan (CodeQL)
5. Unit tests run
6. Dependencies scanned (OWASP)
7. App built (Maven)
8. Docker image created
9. Image scanned (Trivy)
10. Container tested (smoke test)
11. Image pushed to DockerHub (if all pass)

**Result:** Only secure, tested code reaches registry.

---

## Important Notes

- Security automated, not manual
- Fail fast on critical vulnerabilities
- Scan at multiple layers (code, dependencies, container)
- Use secrets for credentials
- Security integrated in CI, not separate
- Shift left = early detection
- DevSecOps is DevOps done securely
- Every commit scanned automatically

---