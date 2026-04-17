# example-platform-repo

This sample models a small but realistic platform repo with multiple component types, domains, and environments using lite-ci component discovery.

`lite-ci` v0.3.0 compiles the intent in `intent.yaml` into a DAG of terraform, helm-values, and helm-chart jobs. Environment policy still lives in the intent, while each owned directory now carries its own `component.yaml` manifest.

The repo layout splits the sample into dedicated areas:

- `infra/*` for Terraform stacks
- `deploy/*` directories that carry Helm values components and values files
- `charts/*/chart` for Helm chart components

Each composition has its own job registry under `assets/config/compositions`:

- `terraform` runs `terraform fmt -check`, `terraform init -backend=false`, and `terraform validate`
- `helm-values` runs `helm lint` and `helm template` against the paired chart path
- `helm-chart` runs `helm lint` and `helm template` against the paired values file

`intent.yaml` declares discovery roots for `infra/`, `deploy/`, and `charts/`. The discovered manifests use `spec.subscribe.environments` so the sample no longer duplicates environment membership in inline component lists. Because manifests live beside the code they own, lite-ci infers the working directory automatically when `spec.path` is omitted.

The workspace manifest in `tinx.yaml` provides:

- `lite-ci` v0.3.0 for intent compilation and execution
- `torkflow` for workflow graph inspection, sourced from a local OCI layout built from `sourceplane/torkflow`
- `kubectl`, `helm`, `terraform`, `az`, and `kustomize` through their setup providers

The CI workflow initializes the workspace with `sourceplane/tinx-action`, lists the available compositions, validates and plans with `lite-ci`, uploads both `plan.json` and the generated `.liteci/component-tree.yaml`, then executes the compiled plan. That gives PRs a direct artifact proving the discovery-based component graph that lite-ci resolved.

Before the workspace is initialized, CI checks out `sourceplane/torkflow` and packages its provider to `.providers/torkflow/oci` so the workspace can consume `torkflow` through a local OCI source. The workflow runs on `macos-latest` because the current Azure CLI setup provider publishes macOS binaries but not Linux ones.
