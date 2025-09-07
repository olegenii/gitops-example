# Repository Analysis: gitops-example

## Overview
This repository is a training and demo GitOps example that demonstrates a two-cluster setup: a local `kind` control-cluster running Flux Operator and Crossplane, and a target workload cluster (named `d1-blue`) provisioned in Azure AKS by Crossplane. The repo is organized as a set of kustomize overlays, Flux/Kubernetes manifests, and helper scripts to bootstrap a local development environment (kind + Flux Operator). Secrets and credentials are stored locally in `.secrets` and `.env` as part of the training workflow.

Key intent: teach and demo modern GitOps patterns — bootstrap a control-plane with Flux Operator, manage infrastructure with Crossplane, and deploy applications using Kustomize and Flux controllers.

## Architecture

- Control plane: a local `kind` cluster (referred to in docs as `kind-control-plane`) that runs Flux Operator and Crossplane. Flux watches this repository and applies the declared Kustomizations.
- Target cluster: `d1-blue` (Azure AKS) — created/provisioned by Crossplane using provider configurations in the repo.
- Source-of-truth: this Git repository. Flux Operator / Flux controllers reconcile Kubernetes resources from this repo into the clusters.
- Packaging: Kustomize is used extensively; directories follow a base/overlay pattern for infrastructure, apps, and monitoring.

Logical flow:
- Author edits files in this repo (apps/, infrastructure/, monitoring/, clusters/).
- Flux Operator running in the control-cluster reconciles the Git state and applies Kustomizations.
- Crossplane resources in the control cluster create cloud resources (AKS, storage, identities) for the target cluster.
- Applications and controllers (cert-manager, external-secrets, ingress-nginx, kagent, etc.) are deployed via Kustomize overlays.

## Key Components

- apps/
  - `base/flux-ui/` — flux-ui application manifests and values (`flux-ui-values.yaml`, `kustomization.yaml`).
  - `base/oauth2-proxy/` — oauth2-proxy manifests and an example `es-oauth2-proxy-github-oidc.yaml` resource.
  - overlays for `control-cluster` and `d1-blue` with mcp subfolders for `flux-mcp-server` and `crawl4ai-rag` (multi-cluster patterns).
- infrastructure/
  - `base/controllers/crossplane/` — Crossplane installation kustomization and provider configuration files.
  - `base/controllers/cert-manager/`, `ingress-nginx/`, `eso/` (External Secrets Operator), and `kagent/` (agent/controller configs and values).
  - overlays for environment-specific config under `infrastructure/overlay/*`.
- monitoring/
  - `base/controllers/jaeger/` — jaeger config for tracing and monitoring.
- clusters/
  - cluster-specific manifests: `clusters/control-cluster/*.yaml` and `clusters/d1-blue/*.yaml` used to define what gets applied to each cluster.
- .secrets and .env
  - Several secret files live in `.secrets/` (ACR, Azure, GitHub, model credentials). These are used by Crossplane/CICD or local scripts; treat them as local test-only secrets.
- bootstrap scripts
  - Repo README and `start.sh`/`stop.sh` references (described in README) indicate automation to create and bootstrap `kind` and Flux Operator locally.

## Technologies Used

- Git + FluxCD / Flux Operator (Flux controllers)
- Crossplane (cloud provisioning)
- Kubernetes (kind locally and Azure AKS remotely)
- Kustomize (organization of manifests)
- Helm (referenced in kustomizations and comments)
- External Secrets Operator (ESO) — for secrets integration
- cert-manager, ingress-nginx — common cluster controllers
- Monitoring: Jaeger
- Misc: bash/zsh scripts, ngrok, GitHub App for Flux bootstrap, YAML-heavy manifests

## Data Flow

1. Developer modifies YAML/kustomize files in this Git repository.
2. Flux Operator running in the control-cluster polls or receives Git events and reconciles Kustomizations/HelmReleases defined here.
3. Crossplane resources declared in the control-cluster create cloud infrastructure (AKS cluster `d1-blue`, managed identities, ACR, DNS, etc.).
4. Flux applies application Kustomizations into the appropriate cluster contexts (control-cluster for control-plane controllers and `d1-blue` for workload apps).
5. External Secrets and `.secrets` are used to inject runtime credentials into clusters (training setup uses local secret files).

## Team and Ownership

- Primary author and maintainer observed in commits: Oleg Satalkin (<oleg.satalkin@gmail.com>). Most commits, log messages, and structural changes are authored by this single contributor.
- Ownership mapping (recommendation):
  - `infrastructure/` — Crossplane, cert-manager, ingress, ESO: infrastructure owner
  - `apps/` — application manifests and overlays: app owner
  - `monitoring/` — observability owner (Jaeger)
  - `.secrets/` — local secret management; must be reworked before sharing publicly

## Quick Recommendations

- Remove or gitignore `.secrets` before sharing or add a clear README and sample files (example.env).
- Add CONTRIBUTING.md + CI (linting/validation for Kustomize/YAML) and a brief script-health-check that validates `kustomize build` across overlays.
- Consider adding a small `make` or `tasks.json` to simplify local bootstrap steps described in the README.

--
Generated from repository layout (`apps/`, `infrastructure/`, `monitoring/`, `clusters/`) and commit history present in `.git/logs/HEAD` (primary author and key commit messages used as evidence).
