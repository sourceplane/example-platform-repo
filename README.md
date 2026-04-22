# example-platform-repo

This sample models a small but realistic platform repo with multiple component types, domains, and environments using gluon component discovery.

`gluon` compiles the intent in `intent.yaml` into an immutable execution DAG of terraform, helm-values, and helm-chart jobs. Environment policy still lives in the intent, while each owned directory carries its own `component.yaml` manifest. The sample now demonstrates two of the newer gluon operating patterns:

- review-scoped planning from explicit changed file lists in pull requests
- direct GitHub Actions `use:` steps inside `JobRegistry` definitions so each composition bootstraps its own toolchain

The repo layout splits the sample into dedicated areas:

- `infra/*` for Terraform stacks
- `deploy/*` directories that carry Helm values components and values files
- `charts/*/chart` for Helm chart components

Each composition has its own job registry under `assets/config/compositions`:

- `terraform` installs Terraform through `hashicorp/setup-terraform` before running `fmt`, `init`, and `validate`
- `helm-values` installs Helm through `azure/setup-helm` before linting and templating the paired chart path
- `helm-chart` installs Helm through `azure/setup-helm` before linting and templating the paired values file

`intent.yaml` declares discovery roots for `infra/`, `deploy/`, and `charts/`. The discovered manifests use `spec.subscribe.environments` so the sample does not duplicate environment membership in inline component lists. Because manifests live beside the code they own, gluon infers the working directory automatically when `spec.path` is omitted.

The workspace manifest in `kiox.yaml` now pins only `ghcr.io/sourceplane/gluon:v0.8.0`. The tool setup logic moved into the job registries themselves, which keeps the workspace surface small and the CI runner portable.

The CI workflow has two lanes:

- pull requests compile a review-scoped plan from the explicit changed file list and emit `.artifacts/pr-plan.json` plus `.gluon/component-tree.yaml`
- pull requests and pushes render a full `.artifacts/plan.json` and execute it with `gluon run --gha`

That split gives reviewers a focused plan artifact without sacrificing a full deterministic execution smoke test. Because the compositions install their own binaries through GitHub Actions-compatible `use:` steps, the workflow can stay on `ubuntu-latest` instead of depending on a pre-baked tool image or macOS-only setup providers.

To run the sample locally with `gluon` on your `PATH`:

```bash
gluon validate \
	--intent intent.yaml \
	--config-dir assets/config/compositions

gluon plan \
	--intent intent.yaml \
	--config-dir assets/config/compositions \
	--output plan.json \
	--view dag

gluon run \
	--plan plan.json \
	--execute \
	--gha
```

If you want per-component toolchain pinning, the composition schemas now expose optional `terraformVersion` and `helmVersion` inputs alongside the existing workload inputs.
