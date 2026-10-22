---
description: Guide to deploying Acme pipelines in production, covering single server, Docker, Kubernetes, and CI/CD integration with GitHub Actions.
---

# Deployment

This guide covers deploying Acme pipelines to production environments — from a simple server to containerized infrastructure.

## Deployment options

```mermaid
graph TD
    A[Acme Pipeline] --> B{Where to deploy?}
    B --> C[Single Server]
    B --> D[Docker / Compose]
    B --> E[Kubernetes]
    B --> F[Acme Cloud]

    style A fill:#818cf8,stroke:#4f46e5,color:#fff
    style F fill:#34d399,stroke:#059669,color:#fff
```

## Option 1: Single server

The simplest deployment — install Acme on a server and run pipelines with the scheduler.

```bash
# Install on your server
pip install acme-cli

# Start the scheduler daemon
acme scheduler start --daemon
```

> [!warning] Single point of failure
> A single server deployment has no redundancy. If the server goes down, pipelines stop running. Consider Docker or Kubernetes for production workloads.

## Option 2: Docker

### Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

# Install Acme
RUN pip install acme-cli==2.4.0

# Copy pipeline definitions
COPY acme.yml .
COPY pipelines/ ./pipelines/
COPY transforms/ ./transforms/

# Start the scheduler
CMD ["acme", "scheduler", "start"]
```

### Docker Compose

```yaml
# docker-compose.yml
version: "3.8"

services:
  acme:
    build: .
    env_file: .env
    volumes:
      - ./pipelines:/app/pipelines
      - acme-state:/app/.acme
    restart: unless-stopped

  acme-api:
    build: .
    command: acme api start --port 8080
    ports:
      - "8080:8080"
    env_file: .env

volumes:
  acme-state:
```

```bash
docker-compose up -d
```

## Option 3: Kubernetes

### Deployment manifest

```yaml
# k8s/deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: acme-scheduler
spec:
  replicas: 1
  selector:
    matchLabels:
      app: acme
  template:
    metadata:
      labels:
        app: acme
    spec:
      containers:
        - name: acme
          image: acme/acme:2.4.0
          command: ["acme", "scheduler", "start"]
          envFrom:
            - secretRef:
                name: acme-secrets
          volumeMounts:
            - name: pipelines
              mountPath: /app/pipelines
      volumes:
        - name: pipelines
          configMap:
            name: acme-pipelines
```

> [!tip] GitOps workflow
> Store your pipeline definitions in a Git repository and use ArgoCD or Flux to automatically deploy changes. This gives you version history and rollback for free.

## CI/CD integration

### GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy Pipelines

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install acme-cli
      - run: acme test tests/

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t acme:${{ github.sha }} .
      - run: docker push myregistry/acme:${{ github.sha }}
      - run: kubectl set image deployment/acme acme=myregistry/acme:${{ github.sha }}
```

## Environment configuration

Use environment-specific configuration files:

```bash
# Development
acme run --env development

# Staging
acme run --env staging

# Production
acme run --env production
```

Each environment loads from `.env.{environment}`. See [[Configuration/Environment Variables|Environment Variables]] for details.

## Related

- [[Guides/Monitoring|Monitoring]] — monitor deployed pipelines
- [[Guides/Authentication|Authentication]] — secure your API in production
- [[Configuration/Configuration File Reference|Configuration]] — production configuration settings
