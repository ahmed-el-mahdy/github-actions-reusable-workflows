# GitHub Actions Reusable Workflows

Production-ready, DRY CI/CD templates for .NET microservices deployed to Azure Kubernetes Service. Reusable workflows eliminate pipeline duplication across multiple repositories.

![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-Reusable_Workflows-2088FF?logo=githubactions)
![.NET](https://img.shields.io/badge/.NET-8.0-512BD4?logo=dotnet)
![Docker](https://img.shields.io/badge/Docker-ACR-2496ED?logo=docker)
![License](https://img.shields.io/badge/License-MIT-green)

## Overview

Instead of duplicating CI/CD logic across 7+ microservice repos, these reusable workflows provide a single source of truth for:

- **Build & Test** — .NET restore, build, test with coverage
- **Security Scan** — Trivy container scanning + CodeQL
- **Docker Build** — Multi-stage build, push to ACR
- **Deploy to AKS** — Kustomize-based deployment per environment
- **APIM Import** — Auto-import OpenAPI spec to Azure API Management

## Workflow Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  Calling Repo (e.g. backend-api-service)                     │
│  .github/workflows/ci-cd.yml                                 │
│    uses: org/github-actions-reusable-workflows/.github/...    │
└──────────────────────────┬──────────────────────────────────┘
                           │ calls
    ┌──────────────────────┼──────────────────────┐
    ▼                      ▼                      ▼
┌──────────┐      ┌──────────────┐      ┌──────────────┐
│  Build   │ ───► │ Docker+Push  │ ───► │  Deploy AKS  │
│  & Test  │      │   to ACR     │      │  + APIM Imp  │
└──────────┘      └──────────────┘      └──────────────┘
```

## Available Workflows

| Workflow | File | Purpose |
|----------|------|---------|
| Build & Test | `build-test.yml` | .NET build, test, coverage |
| Docker Build & Push | `docker-build-push.yml` | Build image, push to ACR |
| Deploy to AKS | `deploy-aks.yml` | Kustomize apply to cluster |
| APIM Import | `apim-import.yml` | Import OpenAPI to APIM |
| Full Pipeline | `full-pipeline.yml` | Orchestrates all steps |

## Quick Start — Calling Workflow

In your microservice repo, create `.github/workflows/ci-cd.yml`:

```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [dev, qa, staging, prod]

jobs:
  pipeline:
    uses: your-org/github-actions-reusable-workflows/.github/workflows/full-pipeline.yml@main
    with:
      dotnet-version: "8.0"
      project-path: "src/MyService.API"
      docker-context: "."
      dockerfile: "dockerfile"
      service-name: "my-service"
      k8s-overlay-path: "k8s/overlays"
    secrets: inherit
```

## Required Secrets (Org/Repo Level)

| Secret | Description |
|--------|-------------|
| `AZURE_CREDENTIALS_DEV` | SP credentials for dev subscription |
| `AZURE_CREDENTIALS_QA` | SP credentials for QA subscription |
| `AZURE_CREDENTIALS_STAGING` | SP credentials for staging |
| `AZURE_CREDENTIALS_PROD` | SP credentials for production |

## Required Variables (Org/Repo Level)

| Variable | Example |
|----------|---------|
| `ACR_LOGIN_SERVER` | `myplatformacr.azurecr.io` |
| `AKS_CLUSTER_NAME_DEV` | `platform-dev-aks` |
| `AKS_RESOURCE_GROUP_DEV` | `platform-dev-rg` |
| `APIM_NAME_DEV` | `platform-dev-apim` |

## Features

- **Branch-based environment mapping** — `dev` branch → dev cluster, `prod` branch → prod cluster
- **DRY** — Write once, use across all microservice repos
- **Security** — Trivy scan on every image build, fails on HIGH/CRITICAL
- **Tagging** — Images tagged with `{env}-{branch}-{short-sha}`
- **Manual deploy** — `workflow_dispatch` with environment selector
- **APIM auto-import** — Swagger/OpenAPI spec imported after successful deploy

## License

MIT
