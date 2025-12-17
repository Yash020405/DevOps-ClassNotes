# Lec 18: Intro to CI/CD

---

## What is CI/CD?

**CI** = Continuous Integration  
**CD** = Continuous Delivery/Deployment

**CI:** Automatically build and test code when pushed.  
**CD:** Automatically deploy to staging/production.

**Goal:** Ship code faster with fewer bugs.

---

## Waterfall Model (Old Approach)

**Process:** Requirements → Design → Development → Testing → Deployment

**Problems:**
- Takes months to release
- Testing happens at end
- Bugs found late = expensive fixes
- No flexibility for changes

**Example:** Plan for 6 months, build for 6 months, test, then deploy. By then, requirements changed.

---

## CI/CD Pipeline

**Modern approach:** Automate everything.

**Flow:**
```
Code Push → Build → Test → Security Scan → Package → Deploy
```

**Benefits:**
- Fast feedback (minutes, not months)
- Catch bugs early
- Deploy multiple times daily
- Less manual work

---

## Continuous Integration (CI)

**What it does:**
1. Developer pushes code to Git
2. CI server detects change
3. Builds application
4. Runs automated tests
5. Reports pass/fail

**Tools:** Jenkins, GitHub Actions, GitLab CI, CircleCI, Travis CI

**Key practices:**
- Commit code frequently (daily)
- Automated builds
- Fast tests
- Fix broken builds immediately

---

## Continuous Delivery vs Deployment

**Continuous Delivery:**
- Automate up to production
- Manual approval for production deploy
- Production-ready always

**Continuous Deployment:**
- Fully automated
- No manual approval
- Auto-deploy to production if tests pass

**Most companies:** Use Continuous Delivery (manual prod approval).

---

## Unit Testing

**What:** Test individual functions/methods in isolation.

**Example:**
```python
def add(a, b):
    return a + b

# Unit test
assert add(2, 3) == 5
```

**Purpose:** Catch bugs in smallest code units.

**Fast:** Runs in milliseconds.

---

## Integration Testing

**What:** Test how components work together.

**Example:** API calls database, returns JSON. Test entire flow.

**Purpose:** Catch issues in component interaction.

**Slower than unit tests** but catches different bugs.

---

## Linting

**What:** Automated code quality checker.

**Checks:**
- Code style consistency
- Syntax errors
- Common mistakes
- Best practices violations

**Tools:**
- **Python:** pylint, flake8
- **JavaScript:** ESLint
- **Java:** Checkstyle

**Example:** Catch unused variables, missing semicolons, wrong indentation.

**Runs early in pipeline** (before build).

---

## SAST (Static Application Security Testing)

**What:** Analyze source code for security vulnerabilities WITHOUT running it.

**Finds:**
- SQL injection risks
- Hardcoded passwords
- Insecure crypto
- Code vulnerabilities

**Tools:** SonarQube, Checkmarx, Veracode

**When:** During CI build, before deployment.

---

## SCA (Software Composition Analysis)

**What:** Scan third-party libraries/dependencies for vulnerabilities.

**Why important:** 80% of code is dependencies (npm, pip, maven).

**Checks:**
- Known vulnerabilities (CVEs)
- Outdated packages
- License issues

**Tools:** Snyk, WhiteSource, Dependabot

**Example:** Finds vulnerable version of log4j.

---

## DevSecOps

**Concept:** Security integrated into DevOps from start.

**Traditional:** Security team checks at end (bottleneck).

**DevSecOps:** Security automated in pipeline.

**Practices:**
- SAST in CI
- SCA for dependencies
- Container scanning
- Secret management (no hardcoded passwords)
- Automated compliance checks

**Shift left:** Find security issues early.

---

## Feature Flagging

**What:** Control feature visibility without redeploying.

**How:** Wrap code in if statement controlled remotely.

**Example:**
```python
if feature_flag_enabled("new_checkout"):
    new_checkout_flow()
else:
    old_checkout_flow()
```

**Use cases:**
- Test in production with 10% users
- Quick rollback (toggle off)
- A/B testing
- Gradual rollout

**Tools:** LaunchDarkly, Unleash, Split

---

## Artifact Management

**What:** Store build outputs (JARs, WARs, Docker images).

**Why needed:** 
- Reuse builds across environments
- Version control for binaries
- Faster deployments

**Tools:**
- **Generic:** Nexus, Artifactory
- **Docker:** Docker Hub, ECR, GCR, Harbor
- **NPM:** npm registry
- **Maven:** Maven Central

**Example:** Build once, deploy same artifact to dev/staging/prod.

---

## Dockerization

**What:** Package app + dependencies into container.

**Benefits:**
- Works same everywhere (dev, prod)
- Lightweight vs VMs
- Fast startup
- Easy scaling

**Dockerfile example:**
```dockerfile
FROM node:16
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "app.js"]
```

**In CI/CD:** Build Docker image → Push to registry → Deploy container.

---

## GDPR (General Data Protection Regulation)

**What:** EU law for data privacy.

**Key requirements:**
- User consent for data collection
- Right to delete data
- Data breach notification (72 hours)
- Data encryption
- Privacy by design

**Impact on CI/CD:**
- No production data in test environments
- Mask sensitive data in logs
- Secure artifact storage
- Compliance audits in pipeline

---

## CI/CD Pipeline Example

**Complete flow:**

1. **Code Push** → Triggers pipeline
2. **Linting** → Code style check
3. **Build** → Compile code
4. **Unit Tests** → Fast tests
5. **SAST** → Security scan
6. **SCA** → Dependency scan
7. **Integration Tests** → Component tests
8. **Docker Build** → Create image
9. **Push to Registry** → Store artifact
10. **Deploy to Staging** → Auto deploy
11. **Smoke Tests** → Basic checks
12. **Deploy to Prod** → Manual approval

---

## Popular CI/CD Tools

**CI/CD Platforms:**
- Jenkins (most popular, open source)
- GitHub Actions
- GitLab CI
- CircleCI
- Travis CI
- Azure DevOps

**Deployment:**
- Kubernetes
- AWS CodeDeploy
- Terraform
- Ansible

---

## Best Practices

- **Commit frequently** (integrate often)
- **Keep builds fast** (under 10 minutes)
- **Fail fast** (run quick tests first)
- **Automated testing** (no manual QA for basic checks)
- **Immutable artifacts** (build once, deploy many times)
- **Infrastructure as Code** (version control everything)
- **Monitor everything** (logs, metrics, alerts)

---

## Common Mistakes

- No tests (defeats CI purpose)
- Slow builds (developers wait hours)
- Flaky tests (random failures)
- Deploying different builds to each environment
- Hardcoded secrets in code
- Skipping security scans
- No rollback plan

---

## Key Takeaways

- CI/CD automates build, test, deploy
- Catch bugs early = cheaper fixes
- Security integrated (DevSecOps)
- Test at multiple levels (unit, integration)
- Use feature flags for safe rollouts
- Docker for consistent environments
- Store artifacts in central registry
- GDPR compliance throughout pipeline
- Automation reduces human error
- Fast feedback loop for developers
