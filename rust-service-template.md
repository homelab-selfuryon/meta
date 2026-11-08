# rust-service-template

**Owner:** platform · GitHub: `homelab-selfuryon/rust-service-template` (template repository)
· Status: **scaffolded** (2026-11-07)

The starting point for a Rust service. Deliberately thin: everything that must evolve comes from
[platform-kit](platform-kit.md) (CUE module, devenv module) and [github-workflows](github-workflows.md),
pinned and bumped by Renovate. After "Use this template" the code belongs to the team.

Contents: axum + tokio, JSON `tracing` logs, `/healthz` and `/readyz`, graceful shutdown; config from
env (non-secret) and secrets from files wrapped in `secrecy`; verification of projected
ServiceAccount tokens for service-to-service calls; `devenv.yaml` importing the platform-kit module; a `Containerfile`;
`deploy/deploy.cue`; thin workflow callers (`ci`, `release-please`, `release`, `promote`);
`CODEOWNERS`; Renovate config extending the org preset.

First services made from it: [notes-api](notes-api.md), [notes-stats](notes-stats.md).
