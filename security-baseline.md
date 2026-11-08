# Security baseline

The principles every repository and cluster in the org follows. When a design choice is unclear,
these decide it.

## Principles

1. **Assume any single thing is compromised** — a workflow, a maintainer account, a laptop, a
   delivery controller, an upstream chart. None of them alone reaches other tenants' secrets or
   changes the platform.
2. **Identity over secrets.** Short-lived, audience-bound tokens: GitHub OIDC, projected
   ServiceAccount tokens, OpenBao Kubernetes auth. No stored Actions secrets, no personal access
   tokens, no shared API keys.
3. **Pull, don't push.** CI publishes signed artifacts; clusters pull and verify them. No cluster
   credentials exist outside the cluster.
4. **Release logic lives outside the product repository.** Reusable workflows from
   `github-workflows`, pinned by SHA; the signing identity is checked at delivery, so a product
   repository can't sign something the platform accepts by itself.
5. **Deny by default** — network (default-deny per namespace, host firewall), RBAC, admission,
   egress. Every allow is explicit and lives next to its owner.
6. **The paved road is the secure road.** `#Service` defaults are the hardened ones; an exception is
   explicit in `deploy.cue` and admission decides whether it is allowed.
7. **Schema is convenience, admission is the fence.** Kyverno and delivery impersonation enforce what
   the CUE schema only suggests.
8. **Secrets are files, not Kubernetes Secrets,** wherever possible: mounted from OpenBao by the CSI
   driver. Nobody but named platform components can read Secrets cluster-wide.
9. **No shell next to the key:** no exec, attach, port-forward, ephemeral containers or `nodes/proxy`
   for tenants; distroless images without a shell.
10. **Everything as code, reviewed and signed** — including GitHub settings and OpenBao policy.
    Click-ops is drift.
11. **Detect what can't be prevented:** API audit log off the cluster; alerts on exec, new bindings,
    Secret reads.
12. **Revocation is a tested procedure** — documented and rehearsed for each system.

## Concrete defaults

| Area | Default | Where |
| --- | --- | --- |
| GitHub | rulesets, signed commits, OIDC only, zizmor, actions pinned by SHA | [platform-terraform](platform-terraform.md), [github-workflows](github-workflows.md) |
| Supply chain | distroless images from a Containerfile (bases pinned by digest), SBOM, provenance, cosign keyless, digests everywhere | [github-workflows](github-workflows.md), [platform-kit](platform-kit.md) |
| Workloads | PSA restricted, read-only rootfs, drop ALL, no token automount, image by digest | `#Service` + Kyverno |
| Network | default-deny ingress and egress per namespace, Cilium host firewall, no pod → kube-apiserver unless declared | [platform-components](platform-components.md) |
| Delivery | platform: Flux with cosign verification, per-component ServiceAccount; tenants: Argo CD by digest with per-team impersonation | [gitops](gitops.md) |
| Secrets | OpenBao, Kubernetes auth per service and environment, files via CSI | [platform-terraform](platform-terraform.md) |
| Access | kube-apiserver, Talos and OpenBao APIs not public; personal identity; offline break-glass | [platform-components](platform-components.md) |
| App code | secrets from files, `secrecy`, deny-by-default authorization, `cargo deny` | [rust-service-template](rust-service-template.md) |
