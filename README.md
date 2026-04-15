# example-platform-repo

This sample keeps the platform design intentionally small and provider-native.

`lite-ci` compiles a single `workspace` component from `intent.yaml` into one smoke-test job. That job executes dry-run-safe commands through a tinx workspace so the flow verifies the workspace-installed tools instead of hard-coded runner dependencies.

The workspace manifest in `tinx.yaml` provides:

- `lite-ci` for intent compilation and execution
- `wtorkflow` for workflow graph inspection, sourced from a local OCI layout built from `sourceplane/torkflow`
- `kubectl`, `helm`, `terraform`, `az`, and `kustomize` through their setup providers

The CI workflow initializes the workspace with `sourceplane/tinx-action@v2.1.1`, validates and plans with `lite-ci`, then executes the compiled plan. Each job step is dry-run safe:

- `kubectl version --client`
- `helm template`
- `terraform fmt -check`
- `az version`
- `kustomize build`
- `wtorkflow view`

Before the workspace is initialized, CI checks out `sourceplane/torkflow` and packages its provider to `.providers/torkflow/oci` so the workspace can consume `wtorkflow` through a local OCI source. The workflow runs on `macos-latest` because the current Azure CLI setup provider publishes macOS binaries but not Linux ones.