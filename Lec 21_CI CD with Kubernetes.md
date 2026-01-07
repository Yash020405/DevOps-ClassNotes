# Lec 21: CI/CD with Kubernetes

---

## Why Separate CI and CD Pipelines?

**Best practice:** Keep CI and CD separate.

**Flow:**
```
CI Pipeline: Code → Build → Test → Create Docker Image → Push to Registry
       ↓
Manual/Automated Approval
       ↓
CD Pipeline: Pull Image → Deploy to K8s → Verify
```

**Benefits:**
- Different teams can own each
- Manual approval gates between stages
- Different failure domains

---

## Self-Hosted GitHub Runners

**Why needed for K8s?**
- Access to **private Kubernetes cluster**
- No network restrictions
- kubectl already installed on K8s node

**Setup on K8s node:**
```bash
# Download runner
mkdir actions-runner && cd actions-runner
curl -o actions-runner-linux-x64-2.311.0.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz
tar xzf ./actions-runner-linux-x64-2.311.0.tar.gz

# Configure with GitHub repo
./config.sh --url https://github.com/YOUR_REPO --token YOUR_TOKEN

# Start runner
./run.sh
```

**Use in workflow:**
```yaml
jobs:
  deploy:
    runs-on: self-hosted  # Uses your K8s node as runner
```

---

## Complete CI/CD Pipeline for K8s

**CI Workflow (.github/workflows/ci.yml):**
```yaml
name: CI

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build
        run: mvn clean package
      
      - name: Test
        run: mvn test
      
      - name: Build Docker Image
        run: docker build -t myapp:${{ github.sha }} .
      
      - name: Push to Registry
        run: |
          docker login -u ${{ secrets.DOCKER_USER }} -p ${{ secrets.DOCKER_PASSWORD }}
          docker push myapp:${{ github.sha }}
```

**CD Workflow (.github/workflows/cd.yml):**
```yaml
name: CD

on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]

jobs:
  deploy:
    runs-on: self-hosted  # K8s node
    steps:
      - uses: actions/checkout@v4
      
      - name: Update image tag
        run: |
          sed -i "s|image:.*|image: myapp:${{ github.sha }}|" k8s/deployment.yaml
      
      - name: Deploy to Kubernetes
        run: |
          kubectl apply -f k8s/deployment.yaml
          kubectl apply -f k8s/service.yaml
      
      - name: Verify deployment
        run: kubectl rollout status deployment/myapp
```

**Key points:**
- CI uses GitHub-hosted runner (ubuntu-latest)
- CD uses self-hosted runner (has kubectl access)
- `workflow_run` trigger: CD runs after CI completes
- Image tagged with commit SHA for versioning

---

## Deployment to Multiple Environments

**Using branches:**
```yaml
name: Deploy to Environments

on:
  push:
    branches:
      - dev        # Deploy to dev namespace
      - staging    # Deploy to staging namespace
      - main       # Deploy to prod namespace

jobs:
  deploy:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to namespace
        run: |
          NAMESPACE=${{ github.ref_name }}
          kubectl apply -f k8s/deployment.yaml -n $NAMESPACE
```

**Or using separate workflows:**
- `.github/workflows/deploy-dev.yml` → dev namespace
- `.github/workflows/deploy-staging.yml` → staging namespace
- `.github/workflows/deploy-prod.yml` → prod namespace

---

## Important Notes

**CI/CD separation:**
- CI = Build + Test + Push image
- CD = Deploy image to K8s

**Self-hosted runner:**
- Installed on K8s master/node
- Has kubectl access
- Connects to GitHub repo

**Image versioning:**
- Tag with `${{ github.sha }}` (commit hash)
- Ensures exact version deployed
- Easy rollback to previous commit

**Deployment verification:**
- Always run `kubectl rollout status`
- Check if new pods are running
- Automated rollback if deployment fails
