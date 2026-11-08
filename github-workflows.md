# github-workflows

**Owner:** platform (tier 1) · GitHub: `homelab-selfuryon/github-workflows` · Status: **scaffolded** (2026-11-07)

Reusable workflows every repository calls, tagged `v1.x`, called pinned by SHA (Renovate bumps).
Their workflow identity in the cosign certificate is what `gitops` and Flux check, so release logic
can't be changed from a product repository.

| Workflow | Caller | Does |
| --- | --- | --- |
| `rust-ci.yaml` | app PR / main | `devenv test` (treefmt, clippy, tests, cargo-deny), `cue vet` |
| `devenv-update.yaml` | every repo, weekly | `devenv update` → PR (Renovate has no devenv manager) |
| `app-preview.yaml` | app PR | preview image + render + `manifests:pr-<n>`, signed (preview identity) |
| `app-release.yaml` | app tag `v*` | image, SBOM, provenance, render staging+prod, `manifests:<version>`, sign, PR to `gitops` (auto-merge) |
| `app-promote.yaml` | app `workflow_dispatch` on a tag | verify, PR to `gitops` prod (team approves) |
| `release-please.yaml` | app main | release PR and tag, token from octo-sts |
| `app-register.yaml` | app main | registration PR to `gitops` |
| `lint-workflows.yaml` | everyone | zizmor, actionlint |

Only workflows shared by several repositories live here. The `gitops` PR checks
(`gitops/.github/workflows/verify.yaml`) and the `platform-components` checks (`check.yaml`) live in
their repositories, like their publishing jobs.

Publishing the artifacts Flux verifies is **not** here either: `gitops` and `platform-components` sign in
their own `publish.yaml` / `release.yaml`, so the certificate names their `main` branch. Bot tokens
come from the public octo-sts.

Contracts: [delivery.md](delivery.md).
