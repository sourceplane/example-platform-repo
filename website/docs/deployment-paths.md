---
title: Deployment Paths
---

## Wrangler direct upload {#wrangler-direct-upload}

The `cloudflare-pages` composition builds the site in CI and then uses `wrangler pages deploy` to
push the generated `docs-build/` directory to a Direct Upload Pages project.

Key inputs:

- `siteDir`
- `installCommand`
- `buildCommand`
- `outputDir`
- `projectName`
- `productionBranch`

That path is useful when you want deployment to follow the build artifact produced by Gluon.

## Terraform Git source {#terraform-git-source}

The `cloudflare-pages-terraform` composition validates the same local site build, then reconciles a
`cloudflare_pages_project` resource that points Cloudflare Pages at this GitHub repository.

Key inputs:

- `terraformDir`
- `rootDir`
- `cloudflareBuildCommand`
- `repoOwner`
- `repoName`
- `projectName`

That path is useful when you want the Pages project itself versioned as infrastructure and let
Cloudflare build from Git after the project is connected.

## Choosing between them

- Use Wrangler when Gluon should own the build artifact and publish it directly.
- Use Terraform when Gluon should manage the Pages project contract and let Cloudflare own the Git-triggered deploys.