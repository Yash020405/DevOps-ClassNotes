# Lec 19: GitHub Actions

---

## What is GitHub Actions?

**GitHub Actions** = built-in CI/CD tool in GitHub.

Automatically runs tasks when events happen in your repo.

**Examples:** Code pushed → build app, PR created → run tests

Defined in YAML files.

---

## Key Concepts

| Term | Meaning |
|------|---------|
| **Workflow** | Automated process (YAML file) |
| **Job** | Set of steps that run together |
| **Step** | Single action or command |
| **Runner** | VM that executes job (Linux, Windows, Mac) |
| **Action** | Reusable code block (checkout, setup Java) |

---

## Project Structure

```
repo/
├── src/
│   └── Main.java
├── pom.xml
└── .github/
    └── workflows/
        └── build.yml
```

**Important:** `.github/workflows/` directory name is exact.

---

## Basic Workflow Example

**.github/workflows/build.yml:**

```yaml
name: Java Build Workflow

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
    
    - name: Build with Maven
      run: mvn clean package
```

---

## Breaking Down Workflow

**Trigger:**
```yaml
on:
  push:
    branches:
      - main
```
Runs when code pushed to main.

**Job:**
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
```
Job named "build" runs on Ubuntu VM.

**Steps:**

**1. Checkout code**
```yaml
- uses: actions/checkout@v4
```
Downloads repo code.

**2. Setup Java**
```yaml
- uses: actions/setup-java@v4
  with:
    java-version: '17'
```
Installs Java 17.

**3. Build**
```yaml
- run: mvn clean package
```
Runs Maven build.

---

## Execution Flow

1. GitHub detects push to main
2. Linux VM created
3. Code checked out
4. Java installed
5. Maven builds project
6. Success → green check, Failure → red cross

---

## Common Triggers

```yaml
on:
  push:                    # On push
  pull_request:            # On PR
  workflow_dispatch:       # Manual trigger
```

---

## Multiple Jobs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: mvn package
  
  test:
    runs-on: ubuntu-latest
    needs: build           # Runs after build
    steps:
      - run: mvn test
```

**needs:** Makes jobs sequential.

---

## Environment Variables

```yaml
env:
  JAVA_VERSION: '17'

steps:
  - run: echo $JAVA_VERSION
```

---

## Caching

```yaml
- name: Cache Maven packages
  uses: actions/cache@v3
  with:
    path: ~/.m2
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
```

Speeds up builds by caching dependencies.

---

## Secrets

**Store secrets in GitHub:**
Repo → Settings → Secrets → New secret

**Use in workflow:**
```yaml
- run: echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u user --password-stdin
```

Never hardcode passwords in YAML.

---

## Matrix Builds

```yaml
strategy:
  matrix:
    java: [11, 17, 21]

steps:
  - uses: actions/setup-java@v4
    with:
      java-version: ${{ matrix.java }}
```

Tests multiple Java versions.

---

## View Results

**GitHub repo → Actions tab → Click workflow run → Expand steps to see logs**

Green check = success  
Red cross = failure

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Wrong folder name | Must be `.github/workflows/` |
| YAML indentation | Use 2 spaces, not tabs |
| Missing checkout | Always checkout code first |
| Java version mismatch | Match pom.xml version |

---

## Popular Actions

- `actions/checkout@v4` - Checkout code
- `actions/setup-java@v4` - Install Java
- `actions/setup-node@v4` - Install Node.js
- `docker/build-push-action@v5` - Build Docker images

---

## Important Notes

- YAML files in `.github/workflows/`
- Always checkout code first
- Jobs run in parallel unless `needs` specified
- Each job runs on fresh VM
- Free for public repos
- Ubuntu runner most common
