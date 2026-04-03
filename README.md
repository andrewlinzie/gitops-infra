# gitops-infra

## Purpose
Stores the desired deployment state for Kubernetes workloads across environments using GitOps.

## Contains
- Environment folders (dev, staging, prod)
- Image tags for deployments
- Helm values overrides
- Kubernetes manifests
- Argo CD application definitions

## Does Not Contain
- Application source code
- Dockerfiles
- Terraform infrastructure code
- Jenkinsfiles

## Future Phases
- Argo CD integration
- Environment promotion workflows
- Deployment version tracking
