# github-workflows

**Owner:** platform (tier 1) · GitHub: `homelab-selfuryon/github-workflows` · Status: **scaffolded** (2026-11-07)

Reusable workflows every repository calls, tagged `v1.x`, called pinned by SHA (Renovate bumps).
Their workflow identity in the cosign certificate is what `gitops` and Flux check, so release logic
can't be changed from a product repository.

| Workflow | Caller | Does |
| --- | --- | --- |
| `rust-ci.yaml` | service PR / main | `devenv test` (treefmt, clippy, tests, cargo-deny), `cue vet` |
| `devenv-update.yaml` | every repo, weekly | `devenv update` → PR (Renovate has no devenv manager) |
| `service-preview.yaml` | service PR | preview image + render + `manifests:pr-<n>`, signed (preview identity) |
| `service-release.yaml` | service tag `v*` | image, SBOM, provenance, render staging+prod, `manifests:<version>`, sign, PR to `gitops` (auto-merge) |
| `service-promote.yaml` | service `workflow_dispatch` on a tag | verify, PR to `gitops` prod (team approves) |
| `release-please.yaml` | service main | release PR and tag, token from octo-sts |
| `components-check.yaml` | platform-components | render, whitelist, Kyverno, diff |
| `gitops-verify.yaml` | gitops | verify PRs |
| `service-register.yaml` | service main | registration PR to `gitops` |
| `lint-workflows.yaml` | everyone | zizmor, actionlint |

Publishing the artifacts Flux verifies is **not** here: `gitops` and `platform-components` sign in
their own `publish.yaml` / `release.yaml`, so the certificate names their `main` branch. Bot tokens
come from the public octo-sts.

Contracts: [delivery.md](delivery.md).
