---
title: Getting Started
---

## Local build

Run the docs site the same way the compositions do:

```bash
cd website
npm ci
npm run docs:build
npm run docs:serve
```

The production build is written to `website/docs-build/`.

## Why Docusaurus here

Docusaurus is a good fit for this sample because it produces a plain static directory while still
exercising a realistic Node-based documentation toolchain. That makes it useful for both:

- direct upload workflows that publish prebuilt assets
- Git-connected Pages projects that build inside Cloudflare

## Credentials used in CI

The repository expects these secrets in GitHub Actions:

- `CLOUDFLARE_ACCOUNT_ID`
- `CLOUDFLARE_API_TOKEN`

The direct-upload job only uses them when it is allowed to publish. The Terraform-backed job uses
the same variables when it needs to plan against or apply Cloudflare resources.