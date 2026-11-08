# gitops

**Owner:** platform; each team owns its `prod.json` files · GitHub: `homelab-selfuryon/gitops` · Status: **scaffolded** (2026-11-07)

What runs where. Holds only data — digests of platform sets per cluster and of service manifests
per environment — plus a small CUE render that turns the data into Flux `OCIRepository` /
`Kustomization` objects and Argo CD `Application` / `ApplicationSet` objects. On merge the render is
published per cluster as a signed OCI artifact that Flux pulls.

Changes arrive as PRs opened by the source repositories' release workflows (identity via octo-sts).
Staging version files have no code owner and are auto-merged after checks; prod version files are
owned by the team. Rollback = revert.

Layout and checks: [delivery.md](delivery.md#gitops-layout).
