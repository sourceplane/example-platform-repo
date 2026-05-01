# example-platform-repo

This sample models a small but realistic platform repo with multiple component types, domains, and environments using gluon component discovery.

For a detailed breakdown of the repo structure and a provider-style OCI composition playbook, see [`GLUON_PROVIDER_STYLE_REPO_GUIDE.md`](./GLUON_PROVIDER_STYLE_REPO_GUIDE.md).

`gluon` compiles the intent in `intent.yaml` into an immutable execution DAG of terraform, helm-values, helm-chart, Cloudflare Worker, Cloudflare Pages, and Turbo package jobs. Environment policy still lives in the intent, while each owned directory carries its own `component.yaml` manifest. The sample now demonstrates two of the newer gluon operating patterns:

- intent-driven composition source resolution from `intent.yaml`
- direct GitHub Actions `use:` steps inside packaged composition jobs so each composition bootstraps its own toolchain

The repo layout splits the sample into dedicated areas:

- `apps/*` for Turbo-style Cloudflare Workers and Pages apps with app-local component manifests
- `infra/*` for Terraform stacks
- `deploy/*` directories that carry Helm values components and values files
- `charts/*/chart` for Helm chart components
- `packages/*` for shared Turbo package components that validate from the workspace root
- `schemas/` for intent and job schema definitions
- `website/` for the Docusaurus docs site and its direct-upload component

`intent.yaml` pins the published OCI composition package from [`sourceplane/sumo-platform-ops-stack`](https://github.com/sourceplane/sumo-platform-ops-stack). That provider exports one composition per component type:

- `terraform` installs Terraform through `hashicorp/setup-terraform` before running `fmt`, `init`, and `validate`
- `helm-values` installs Helm through `azure/setup-helm` before linting and templating the paired chart path
- `helm-chart` installs Helm through `azure/setup-helm` before linting and templating the paired values file
- `cloudflare-worker-turbo` installs Node.js and pnpm, builds a Worker app from the monorepo root, then deploys it with Wrangler on `main`
- `cloudflare-pages` installs Node.js, builds the site, and direct-uploads static assets to Cloudflare Pages with Wrangler when running from `main`
- `cloudflare-pages-turbo` installs Node.js and pnpm, builds a Turbo app from the monorepo root, and direct-uploads the app build to Cloudflare Pages
- `cloudflare-pages-terraform` installs Node.js and Terraform, verifies the same site locally, then reconciles a Git-backed Cloudflare Pages project through `cloudflare_pages_project`
- `cloudflare-pages-turbo-terraform` installs Node.js, pnpm, and Terraform, verifies a Turbo app locally, then reconciles a Git-backed Cloudflare Pages project that builds from the monorepo root
- `turbo-package` installs Node.js and pnpm, then validates a shared package from the monorepo root without deploying infrastructure

`intent.yaml` declares discovery roots for `apps/`, `infra/`, `deploy/`, `charts/`, `packages/`, and `website/`. The discovered manifests use `spec.subscribe.environments` so the sample does not duplicate environment membership in inline component lists. Because manifests live beside the code they own, gluon infers the working directory automatically when `spec.path` is omitted.

The workspace manifest in `kiox.yaml` pins `ghcr.io/sourceplane/orun:v0.9.6`. The tool setup logic lives in the composition jobs themselves, which keeps the workspace surface small and the CI runner portable.

The CI workflow has two lanes:

- pull requests compile a dependency-focused review plan from `intent.yaml`
- pull requests and pushes render a full plan and execute it with `gluon run --gha`

That split gives reviewers a focused planning view without sacrificing a full deterministic execution smoke test. Because the compositions install their own binaries through GitHub Actions-compatible `use:` steps, the workflow can stay on `ubuntu-latest` instead of depending on a pre-baked tool image or macOS-only setup providers. The execute lane now passes `CLOUDFLARE_ACCOUNT_ID` and `CLOUDFLARE_API_TOKEN` through to the Cloudflare deploy jobs so review branches can still verify builds while `main` retains the only side-effecting publish or apply path.

The Cloudflare examples are now split across app-local and infra-backed patterns:

- `website/` exercises direct upload with the docs site build and keeps the component manifest beside the site
- `infra/cloudflare-pages-terraform/` manages a Git-connected Pages project that builds the same site inside Cloudflare
- `apps/api-edge/`, `apps/identity-worker/`, and `apps/projects-worker/` show Worker deploys that only need a `component.yaml` dropped into the app directory in a reference Turbo repo
- `apps/web-console/` shows a direct-upload Pages app that installs and builds from the workspace root
- `apps/admin-console/` plus `infra/cloudflare-pages-admin-console/` show a Git-backed Pages project where the app manifest stays local to the app but the Terraform contract lives under `infra/`
- `packages/sdk/` and `packages/shared/` show non-deploying package components that still participate in Gluon planning and verification

To run the sample locally with `gluon` on your `PATH`:

```bash
gluon validate \
	--intent intent.yaml

gluon plan \
	--intent intent.yaml \
	--output plan.json \
	--view dag

gluon run \
	--plan plan.json \
	--execute \
	--gha
```

If you want per-component toolchain pinning, the composition schemas now expose optional `terraformVersion` and `helmVersion` inputs alongside the existing workload inputs.
