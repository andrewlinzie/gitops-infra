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
- defines which version (image tag) runs in each environment

```

CI → pushes image → updates GitOps repo → (Argo CD sync later)

```

---

## Repository Structure

```

gitops-infra/
├── dev/
│   └── api-service/
│       └── values.yaml
├── staging/
│   └── api-service/
│       └── values.yaml
├── prod/
│   └── api-service/
│       └── values.yaml
├── argocd/ (future use)
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
image:
  repository: <ecr-repo>
  tag: <git-sha>
````

Only the `tag` changes across promotions.

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
* Future Argo CD configuration

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

## Current State (Phase 5)

* Dev auto-update pipeline working
* Staging manual promotion working
* Prod promotion workflow implemented
* Argo CD NOT yet integrated

---

## Future Phases

* Argo CD installation and configuration
* Automatic reconciliation from Git to cluster
* Rollback via Git history
* Multi-service GitOps management