# example-platform-repo

This sample now models a small but realistic platform repo with multiple component types, domains, and environments.

`lite-ci` compiles the intent in `intent.yaml` into a DAG of terraform, helm-values, and helm-chart jobs. The current example splits the platform into `platform-foundation`, `customer-identity`, and `commerce-checkout` domains across `development`, `staging`, and `production`.

The repo layout now splits the sample into dedicated areas:

- `infra/*` for Terraform stacks
- `deploy/*` directories that carry Helm values files
- `charts/*/chart` for Helm chart components

Each composition has its own job registry under `assets/config/compositions`:

- `terraform` runs `terraform fmt -check`, `terraform init -backend=false`, and `terraform validate`
- `helm-values` runs `helm lint` and `helm template` against the paired chart path
- `helm-chart` runs `helm lint` and `helm template` against the paired values file

The workspace manifest in `tinx.yaml` provides:

- `lite-ci` for intent compilation and execution
- `torkflow` for workflow graph inspection, sourced from a local OCI layout built from `sourceplane/torkflow`
- `kubectl`, `helm`, `terraform`, `az`, and `kustomize` through their setup providers

The CI workflow initializes the workspace with `sourceplane/tinx-action@v2.1.1`, lists the available compositions, validates and plans with `lite-ci`, then executes the compiled plan. The original `workspace` composition still exists under `assets/config/compositions/workspace` as a legacy fixture, but the primary sample now fans out across multiple component types and environment-specific selections.

Before the workspace is initialized, CI checks out `sourceplane/torkflow` and packages its provider to `.providers/torkflow/oci` so the workspace can consume `torkflow` through a local OCI source. The workflow runs on `macos-latest` because the current Azure CLI setup provider publishes macOS binaries but not Linux ones.
