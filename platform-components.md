# platform-components

**Owner:** platform (tier 1) · GitHub: `homelab-selfuryon/platform-components` · Status: **scaffolded** (2026-11-07)

Everything cluster-scoped:

- `clusters/<cluster>/` — cluster definition, `pins.json`, Talos patches.
- `components/<category>/<name>/` — one CUE package per component: upstream chart + typed values
  (rendered with `helm template` in CI) and/or plain objects, its namespace, network policy, and its
  **cluster-scoped whitelist**.
- `registry/latest.json` — newest versions (Renovate).
- `policies/` — Kyverno CEL policies (tenant restrictions, image verification, generated
  default-deny and LimitRange, Flux ServiceAccount binding).
- `teams/*.cue` — onboarding: namespaces per environment, quotas, AppProjects, deploy
  ServiceAccounts, RBAC for the team group.

Each cluster's components are grouped into sets (`rbac`, `system`, `platform`, `policy`,
`tenants`); each set is released as one signed artifact and pinned in `gitops`. Details:
[delivery.md](delivery.md#platform-layout-platform-components).

Related: [gitops](gitops.md), [platform-terraform](platform-terraform.md).
