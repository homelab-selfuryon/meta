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
- `teams/` — onboarding: which teams exist comes from the [catalog](catalog.md)
  (`teams/catalog.json`, a `catalog/sync` PR); namespaces per environment, quotas (size `small` by
  default in `schema.cue`, a team's own values in `teams/<team>.cue`), AppProjects, deploy
  ServiceAccounts, RBAC for the team group.

Each cluster's components are grouped into sets (`rbac`, `system`, `platform`, `policy`,
`tenants`); the cluster is released as one signed artifact (a directory per set) and pinned in `gitops`. Details:
[delivery.md](delivery.md#platform-layout-platform-components).

Related: [gitops](gitops.md), [platform-terraform](platform-terraform.md).
