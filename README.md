# gitops-infra

## Purpose
Stores the desired deployment state for all services across environments using a GitOps model.

This repository acts as the **single source of truth for deployment state**.

---

## Core Concept

Application repositories:
- build artifacts (Docker images)
- publish images to ECR

This repository:
- defines the full deployment configuration for each service in each environment, including:
  - image version (tag)
  - replica counts
  - resource requests/limits
  - health probes
  - environment variables
  - autoscaling (HPA)

Image tag updates are the primary mechanism for promotion across environments.

```
CI -> pushes image -> updates GitOps repo -> ArgoCD detects change -> deploys to cluster
```
---

## Deployment Model (Helm + ArgoCD)

This repository uses a Helm-based GitOps model:

- `charts/`
  - Contains reusable Helm charts for each service
- `dev/`, `staging/`, `prod/`
  - Provide environment-specific values for each service
- `argocd/`
  - Defines ArgoCD Application manifests per service and environment

ArgoCD deploys services by:
- referencing the Helm chart
- applying the environment-specific values file

---

## Role in GitOps Deployment

This repository is continuously monitored by ArgoCD.

ArgoCD:
- Uses Application manifests defined in `argocd/`
- References Helm charts from `charts/`
- Applies environment-specific values from `dev/`, `staging/`, and `prod/`
- Reconciles the Kubernetes cluster to match the desired state in Git

This enables a pull-based deployment model where:
- Git defines the desired state
- ArgoCD ensures the cluster matches that state

---

## Repository Structure

```
gitops-infra/
├── argocd/
│   ├── dev-api-service.yaml
│   ├── dev-ai-inference-service.yaml
│   ├── staging-api-service.yaml
│   ├── staging-ai-inference-service.yaml
│   ├── prod-api-service.yaml
│   └── prod-ai-inference-service.yaml
├── charts/
│   ├── api-service/
│   └── ai-inference-service/
├── dev/
│   ├── api-service/
│   │   └── values.yaml
│   └── ai-inference-service/
│       └── values.yaml
├── staging/
│   ├── api-service/
│   │   └── values.yaml
│   └── ai-inference-service/
│       └── values.yaml
├── prod/
│   ├── api-service/
│   │   └── values.yaml
│   └── ai-inference-service/
│       └── values.yaml
└── README.md
````

---

## Environment Model

### Dev
- Auto-updated by CI pipeline
- Fast feedback loop
- Tracks latest successful build from `main`

### Staging
- Manual promotion only
- Uses previously built image (no rebuild)
- Used for validation before production

### Prod
- Manual promotion only
- Same artifact promoted from staging
- Stable, controlled environment

---

## Values File Structure

Each service defines its deployment state via:

```yaml

replicaCount: 3

image:
  repository: <ecr-repo>
  tag: <git-sha>

resources:
  requests:
  limits:

readinessProbe:
livenessProbe:

env:

hpa:
  enabled: true

```

Only the image `tag` changes across promotions. Other configuration values remain environment-specific.

---

## Promotion Workflow

Promotion is performed by updating the GitOps repo:

* Dev → automatic via CI
* Staging → manual workflow
* Prod → manual workflow

No environment rebuilds occur.

**Principle:**

```
build once → promote same artifact
```

---

## CI/CD Integration

The `api-service` pipeline:

* builds image
* pushes to ECR
* updates `dev/api-service/values.yaml`

Manual promotion workflow:

* updates staging/prod values files
* commits changes to this repo

Each commit represents a deployment event.

---

## Contains

* Environment-specific deployment state
* Image tag references
* Helm values overrides
* ArgoCD Application manifests

---

## Does Not Contain

* Application source code
* Dockerfiles
* Terraform infrastructure
* CI pipelines
* Secrets

---

## Key Architectural Decisions

* Git as the source of truth for deployment state
* Strict separation of:
  * build (service repos)
  * deployment state (GitOps repo)
* Immutable artifact promotion across environments
* Environment isolation via directory structure

---

## Current State

- Dev environment auto-updates via CI pipeline
- Staging and prod promotion workflows are manual
- ArgoCD is integrated and continuously reconciles this repository into the EKS cluster
- Each commit to this repository represents a deployment event

---

## Future Enhancements

- Multi-service GitOps standardization and templating
- Policy enforcement and validation (e.g., preventing invalid promotions)
- Enhanced promotion workflows across environments

---

## Observability Integration

This repository now includes observability configuration as part of the GitOps model.

### Components

- Prometheus (Helm deployment via ArgoCD)
- Grafana dashboards
- Prometheus alert rules (`PrometheusRule` resources)

### Key Principle

Observability is managed through GitOps:

- No manual `kubectl` configuration
- All monitoring and alerting configuration is version-controlled
- ArgoCD continuously reconciles observability state into the cluster

### Structure

```
observability/
├── prometheus/
├── grafana/
└── alerts/
```

This ensures monitoring, alerting, and dashboards are treated as part of the platform, not external tooling.