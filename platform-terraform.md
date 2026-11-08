# platform-terraform

**Owner:** platform · GitHub: `homelab-selfuryon/platform-terraform` · Status: **scaffolded** (2026-11-07)

OpenTofu for everything outside the cluster that has an API, one root module per provider:

- `github/` — org settings, teams, repositories and their settings, environments (`staging`, `prod`
  with tag-only deployment rules) for service repositories, rulesets, custom properties, Actions
  policy.
- `openbao/` — Kubernetes auth, the `kv/` mount, one policy and role per service × environment, the
  platform component roles. Applied over the private access path.
- later `cloudflare/` (zone, scoped API tokens).

Each folder has its own **local** state, ignored by git; every resource has an `import {}` block so
the state can be rebuilt from code. The owner applies locally (`GITHUB_TOKEN=$(gh auth token)`); CI
only formats, validates and lints — it holds no credentials.

Related: [org.md](org.md), [decisions.md](decisions.md).
