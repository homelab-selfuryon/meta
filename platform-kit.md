# platform-kit

**Owner:** platform (tier 1) · GitHub: `homelab-selfuryon/platform-kit` · Status: **scaffolded** (2026-11-07)

The paved road product teams consume:

1. **CUE module** `github.com/homelab-selfuryon/platform-kit@v0`, published to the GHCR CUE
   registry on `v*` tags. Package `app`: `#App` (the schema of `deploy.cue`) and its render
   to Kubernetes objects with hardened defaults. Package `env`: environment definitions (namespace
   pattern, hostnames, replicas and sizes per environment). Interface in
   [delivery.md](delivery.md#app-contract-deploycue).
2. **devenv modules** (`devenv/`), imported from `devenv.yaml` of every repository, in three layers:
   - `base.nix` — git hooks (ripsecrets, detect-private-keys, treefmt), treefmt (nixfmt, prettier,
     taplo), workflow linters. Every repository.
   - `cue.nix` — base + CUE (option for an upstream pre-release binary), `CUE_REGISTRY`, `cue fmt`.
     `platform-components`, `gitops`, `github-workflows`.
   - `rust-app.nix` — cue + Rust toolchain from `rust-toolchain.toml`, cargo-deny, clippy, optional
     local Postgres, the app's `devenv test`. Apps.

   A repository's own `devenv.nix` holds only its own tools. Changes reach everyone by bumping the
   tag in `devenv.yaml` (Renovate) instead of copying.

Semver; a breaking schema change bumps the major (`@v1`).

Related: [rust-app-template](rust-app-template.md), [github-workflows](github-workflows.md).
