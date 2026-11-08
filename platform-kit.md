# platform-kit

**Owner:** platform (tier 1) · GitHub: `homelab-selfuryon/platform-kit` · Status: **scaffolded** (2026-11-07)

The paved road product teams consume:

1. **CUE module** `github.com/homelab-selfuryon/platform-kit@v0`, published to the GHCR CUE
   registry on `v*` tags. Package `service`: `#Service` (the schema of `deploy.cue`) and its render
   to Kubernetes objects with hardened defaults. Package `env`: environment definitions (namespace
   pattern, hostnames, replicas and sizes per environment). Interface in
   [delivery.md](delivery.md#service-contract-deploycue).
2. **devenv module** (`devenv/`), imported from a service's `devenv.yaml`: Rust toolchain from
   `rust-toolchain.toml`, cue, cargo-deny, treefmt (rustfmt, nixfmt, `cue fmt`), git hooks, optional local
   Postgres, and an option to switch CUE to an upstream pre-release binary. Fixes reach every service
   by bumping the input instead of re-templating.

Semver; a breaking schema change bumps the major (`@v1`).

Related: [rust-service-template](rust-service-template.md), [github-workflows](github-workflows.md).
