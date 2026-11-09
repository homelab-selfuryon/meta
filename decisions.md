# Decisions

Settled decisions first, then open questions. Each entry has the date it was settled. Rejected options
stay, struck through, with the reason. How the decisions fit together end to end is in
[delivery.md](delivery.md).

## Decided

### Organization

- **2026-11-07 — Separate GitHub org `homelab-selfuryon`.** Org-level policy (rulesets, custom
  properties, teams, OIDC subject prefix, one GHCR namespace) is the point of the example; a
  personal account has none of it. See [org.md](org.md).
- **2026-11-07 — Public repositories, GitHub Free.** Rulesets, environment protection and secret
  scanning are free for public repositories only. Consequence: nothing sensitive is committed,
  encrypted or not.
- **2026-11-07 — Org settings applied by hand once:** base permission `none`, members can't create
  repositories. Everything after that goes through `platform-terraform/github`.
- **2026-11-07 — Org as code: OpenTofu in `platform-terraform`.** One root module per provider in a
  subfolder (`github/`, `openbao/`, later `cloudflare/`), each with its own **local** state in
  `.gitignore`. Every resource has an `import {}` block, so a lost state is rebuilt from code. The
  owner applies locally; CI runs `fmt`/`validate`/tflint/zizmor without credentials.
- **2026-11-07 — Rulesets per repository, one required check `final-check`.** GitHub Free has no
  org-level rulesets or required workflows (API 404), so `platform-terraform/github` creates the
  same two rulesets in every repository (default branch; immutable `v*` tags). Every repository's
  CI has a job `final-check` that fails unless all its other jobs succeeded or were skipped.
- **2026-11-07 — Admin bypass through a PR only, while there is one maintainer.** Code-owner review
  can't be satisfied by the PR author, so repository admins may merge their own PR; direct pushes
  stay blocked. `ruleset_admin_bypass = false` once there is a second maintainer.
  ~~`org`, `github-configuration`, `github-config`~~ — one repository for all provider config
  instead of one per provider.
- **2026-11-07 — `.github` repository kept** for what only that name does: org profile
  (`profile/README.md`), default community health files, workflow templates. It also holds the
  shared Renovate preset (`default.json`). Docs live in `meta`.
- **2026-11-07 — Docs repository `meta`.** The workspace map, decisions, delivery flow and security
  baseline, changed by PR so the decision history is the git history.

### Paved road

- **2026-11-07 — CUE, no per-app Helm charts, no IDP.** One platform CUE module validates and
  renders `deploy.cue`. Rejected at this scale: Backstage/OpenChoreo-style IDPs, KubeVela (CUE +
  GitOps gives the same without CRDs), Kratix, KusionStack, Humanitec, Kubero, Devtron. Timoni is a
  valid alternative but still 0.x.
- **2026-11-07 — Rust only** for application code.
- **2026-11-08 — A product team deploys an "app", not a "service".** `#App`, `app-*` workflows,
  `apps.json`, `kind=app`, `rust-app-template`. "Service" collided with the Kubernetes `Service` the
  schema renders and would be wrong for workers and CronJobs; `apps/` in `gitops` already used the
  word. Renamed before the first release, while workflow names (signing identities) were still free.
- **2026-11-07 — Thin template, logic in versioned dependencies.** `rust-app-template` is copied
  once; everything that must evolve comes as a pinned, Renovate-bumped dependency: the CUE module
  and devenv module from `platform-kit`, reusable workflows from `github-workflows`, the Renovate preset
  from `.github`. After creation the Cargo files belong to the team. No template sync tool.
- **2026-11-08 — CUE 0.18 pre-release everywhere:** language version `v0.18.0`, the latest
  upstream release candidate binary from devenv (`v0.18.0-rc.3` today), the same in CI through
  `devenv shell`. Back to the nixpkgs release once 0.18.0 is out.
- **2026-11-08 — devenv everywhere, no Nix flakes; images from a Containerfile.** Every repository
  gets its tools from devenv (`devenv.nix`, `devenv.yaml`, `devenv.lock`); CI runs the same commands
  through `devenv shell`, so laptops and CI use the same versions. Apps import a devenv module
  from `platform-kit` (toolchain, treefmt, git hooks, local Postgres). We ship OCI images to
  Kubernetes and need no Nix packages, so images are built from a multi-stage `Containerfile`
  (distroless runtime, base images pinned by digest, Renovate bumps them) with buildx, which also
  produces SBOM and provenance. Cost: `devenv.lock` has no Renovate manager — a scheduled
  `devenv update` workflow opens the PRs. ~~Flake devshells; Nix-built images (crane +
  `dockerTools`)~~ — more machinery than a shell and an image need.
- **2026-11-08 — Shared devenv layers from `platform-kit`:** `base.nix` (secret and private-key
  hooks, formatters, workflow linters) for every repository, `cue.nix` for CUE repositories,
  `rust-app.nix` for apps. One copy of the CUE build and the hooks instead of one per repository.
- **2026-11-08 — Renovate config in `.github/renovate.json`** in every repository (the preset itself
  stays `default.json` at the root of `.github`).
- **2026-11-07 — Reusable workflows in their own repository `github-workflows`**, tagged `v1.x`,
  called pinned by SHA (Renovate bumps). Their workflow identity is the signing identity checked at
  delivery, so release logic lives outside the product repository.

### Delivery

- **2026-11-07 — Source repositories release, `gitops` only pins.** Each source repository renders
  and publishes its own signed, content-addressed OCI manifest artifact and keeps its own input lock
  (`Cargo.lock` + `cue.mod` in an app, `pins.json` per cluster in `platform-components`). Its CI
  opens a PR to `gitops` that moves one digest. `gitops` holds the output locks — what runs where.
  Rollback = revert in `gitops`.
  - An app artifact contains the render for every environment; the environment is chosen by
    path. Promotion moves the same digest.
  - Cost accepted: a `platform-kit` change reaches an environment only when each app
    re-releases (Renovate bumps it everywhere).
  - ~~Input and output locks both in `gitops`, render in `gitops` CI~~ — two steps per change and
    render logic split from its source. ~~Rendered YAML committed to git~~. ~~`gitops` polls GHCR
    instead of cross-repo PRs~~.
- **2026-11-07 — `gitops` holds data, renders thin objects, is itself a signed OCI artifact.**
  Version files are pure data (`{version, digest}`); the Flux and Argo CD objects are rendered from
  them by CUE in `gitops`, so a release PR can change a digest and nothing else. On merge to `main`
  the render is pushed as a signed OCI artifact per cluster; Flux pulls it with cosign
  verification. Argo CD has no git access at all.
- **2026-11-07 — Two lanes: Flux for the platform, Argo CD for tenants.** Flux applies platform
  artifacts and the `Application` objects. Argo CD syncs tenant artifacts (OCI by digest) with
  per-team AppProject and sync impersonation. Argo is not used for platform components: it would
  need cluster-admin and can't verify signatures. Argo can't widen its own rights because Flux
  installs it.
- **2026-11-07 — One cluster, one Argo CD, three environments:** `preview` (ephemeral, per PR),
  `staging`, `prod`, as namespaces `<team>-<env>` (`<team>-pr-<n>` for preview). An environment is
  data in `platform-kit`; the design stays cluster- and environment-agnostic. No argocd-agent.
- **2026-11-07 — Promotion by tags.**
  - `preview`: every PR builds a preview artifact (tag `pr-<n>`), signed by a separate workflow
    identity that staging and prod don't accept; an ApplicationSet creates it per open PR.
  - `staging`: a `v*` tag → release → PR in `gitops` → auto-merged after checks. A merge to `main`
    alone deploys nothing.
  - `prod`: a person runs the `promote` workflow **on the tag** → PR moving prod to that version's
    digest → team CODEOWNERS approval.
  - Tags come from release-please (changelog and semver from Conventional Commits).
- **2026-11-07 — GitHub Environments `staging` and `prod` in each app repository, for
  identity, not as a gate.** Deployment rule "tags `v*` only"; the OIDC `sub` carries
  `environment:<env>`, so octo-sts issues the `gitops` token only to that workflow and environment.
  No required reviewers: the gate is the `gitops` PR.
- **2026-11-08 — Onboarding an app: a PR to `platform-terraform`, then self-registration.**
  `apps.json` creates the repository from the template with its properties, environments,
  rulesets and OpenBao roles; the template's `register.yaml` then opens the `gitops` registration PR,
  which the platform approves. Rejected: an always-on onboarding service with a GitHub App
  (that is an IDP), creating repositories by hand (drift, forgotten environments).
- **2026-11-08 — Custom properties in the OIDC token:** `owner-team` and `kind`
  (`app`/`platform`/`docs`), so trust policies check what a repository is, not only its name.
- **2026-11-07 — Cross-repository PRs through octo-sts** (GitHub OIDC → short-lived token for
  `gitops`), trust policies in `gitops/.github/chainguard/`. No stored keys or PATs.
- **2026-11-08 — Public octo-sts, accepted risk.** Its App has broad permissions (including
  organization administration write) and a bot identity shared with every other user; a compromise
  of the service could act on our installed repositories. Acceptable for a homelab showcase: rulesets
  still block pushes to `main` (the App is no bypass actor and can't edit repository rulesets),
  releases need our own workflow identity and a tag on `main`, and the gitops verify checks every
  artifact. Considered: ~~self-hosted octo-sts (Cloud Run + KMS)~~ — the right next step for a real
  org, not worth running here; ~~own App + key in an environment secret~~ — long-lived bearer key;
  ~~Renovate as the promoter~~ — promotion on Renovate's schedule; ~~Kargo~~ — git credential in the
  cluster, no generic OCI subscriptions.
- **2026-11-08 — Artifacts Flux verifies are signed in the publishing repository's own job.**
  `gitops` and `platform-components` sign in a workflow defined in themselves, so the certificate
  subject ends in `@refs/heads/main` and Flux rejects anything published from a branch. A reusable
  workflow puts `github-workflows` in the subject and lets any branch of the caller publish.
- **2026-11-07 — Kargo: not now.** The layout stays compatible with it.

### Platform components

- **2026-11-07 — Platform has clusters, not environments.** Each cluster has its own component set
  and its own pins (`clusters/<c>/pins.json`); versions move between clusters by promotion.
- **2026-11-07 — Upstream charts rendered in CI (variant B), `HelmRelease` only as a listed
  exception (A).** B: `helm template` in CI from typed CUE values; the artifact holds plain objects;
  no Helm in the cluster. A: for charts B can't handle (hooks, `lookup`), the artifact holds a
  `HelmRelease` with the chart pinned by digest; CI still renders the chart for the PR diff, the
  whitelist and Kyverno, without publishing that render, and marks the diff as approximate.
  2026-11-09: Kyverno moved to A (its post-upgrade and pre-delete hooks are needed). The
  `HelmRelease` lives in `flux-system` next to the ServiceAccounts and installs into the
  component's namespace; A is limited to privileged components until component ServiceAccounts get
  rights in `flux-system`.
- **2026-11-09 — Policies before the platform components.** Set order `rbac`, `system`, `policy`,
  `platform`, `tenants`; `policy` holds only the Kyverno policies and the host firewall. With
  `wait: true` a set is ready only when all its components are, so with `policy` after `platform`
  a component waiting for a secret (external-dns at bootstrap) kept the policies out of the
  cluster. ClusterIssuers and Gateways moved to `platform`, next to the operators they need.
- **2026-11-09 — alpha-01 runs without disk encryption for now.** The node was installed
  before the platform design with Secure Boot off; Talos encrypts a volume only when it creates it,
  so TPM-sealed LUKS2 needs a reinstall with Secure Boot on. Bootstrapped on the existing install
  instead; `talos/patches/encryption.yaml` stays ready (`ENCRYPT=1`). Do the reinstall before
  anything that matters lives on the node (OpenBao data, databases), or back those up first.
- **2026-11-09 — alpha-01's networks are committed.** Private ranges only (an IPv6 ULA prefix and
  the RFC 1918 LAN): they reveal next to nothing and the host firewall needs them in the published
  render. The node's public IPv6 prefix and the install disk's serial stay out of git
  (`.secrets/talos/<cluster>/local.yaml`). The domain is still a placeholder.
- **2026-11-07 — Cluster-scoped whitelist per component.** A component declares which
  cluster-scoped kinds and names it may create; tenants may create none. Enforced three times: CI
  check of the render, a per-component ServiceAccount for Flux (RBAC by kind) and an admission
  policy bound to that SA (names and API groups, which RBAC `create` can't restrict).
- **2026-11-07 — Flux without blanket cluster-admin.** One `Kustomization` per component with its own
  `serviceAccountName`; the default SA has no rights. Contains a compromised or broken upstream
  chart and attributes changes in the audit log. Not a defence against a compromised Flux
  controller. Exception: the core components (Flux, Cilium, Kyverno, the bootstrap RBAC set) run
  with cluster-admin, marked `privileged: true` with a reason; their render still goes through the
  whitelist check.
- **2026-11-07 — `*-system` namespaces** for every platform component: only Flux writes there,
  label `homelab-selfuryon.github.io/managed-by: platform`, default-deny plus a per-component
  CiliumNetworkPolicy, PSA `restricted` unless the component is a listed exception. The platform
  team gets view rights plus break-glass, like any other team.
- **2026-11-07 — Kyverno policies live in `platform-components`**, CEL policies
  (`policies.kyverno.io/v1`), Enforce from day one. `gitops` PR checks run the policy version the
  target cluster currently runs.
- **2026-11-07 — Ingress: Envoy Gateway** (Gateway API) rather than Cilium's Gateway: JWT/OIDC, rate
  limits, `BackendTrafficPolicy`. One Gateway per environment in `envoy-gateway-system`, merged
  (`mergeGateways`). Teams only write `HTTPRoute` through `#App`.
- **2026-11-07 — Databases: CloudNativePG**, one cluster per app per environment.

### Secrets

- **2026-11-07 — OpenBao in the cluster from the start; no External Secrets Operator.**
  - Applications get secrets as **files** through the Secrets Store CSI driver with the OpenBao
    provider, authenticated by their own ServiceAccount (Kubernetes auth, in-cluster TokenReview).
    No Kubernetes Secret is created for tenant secrets.
  - OpenBao layout: KV v2 at `kv/`, paths `kv/<team>/<env>/<app>/<key>`; one Kubernetes auth
    role per app and environment, bound to that app's SA in `<team>-<env>`, with a policy
    for its own path only. Platform secrets under `kv/platform/<cluster>/<component>/`.
  - Platform components that insist on a Kubernetes Secret (DNS provider token for external-dns and
    cert-manager) get one through the CSI driver's `secretObjects` sync — allowed only in
    `*-system` namespaces, a listed exception.
  - OpenBao config (auth, mounts, policies, roles) is code in `platform-terraform/openbao`, applied
    by the owner over the private access path.
  - Unseal: static seal (OpenBao ≥ 2.4); the seal key is a Secret created at bootstrap from the
    break-glass bundle, never in git; Talos disk encryption sealed to TPM; an offline copy of the
    key and the recovery keys. Raft snapshots off the cluster.
  - Database credentials: CloudNativePG's generated `<cluster>-app` Secret mounted as a file for
    now (a known exception); OpenBao dynamic Postgres credentials later.
  - ~~Infisical Cloud + ESO~~ — needs a static credential in the cluster (Kubernetes auth would make
    Infisical call our private API server) and an extra moving part. ~~Bitwarden Secrets
    Manager~~, ~~HCP Vault Secrets~~ (end of life), ~~SOPS/Sealed Secrets for app secrets~~ —
    encrypted secrets in public git are still out.

### Access

- **2026-11-07 — Break-glass independent of the private access network:** offline age-encrypted
  talosconfig and admin kubeconfig (key on a hardware token) + home LAN path; every use logged and
  reviewed.

## Deviations from the security defaults, on purpose

- Argo CD can't verify OCI signatures. The digest pinned in reviewed `gitops` is the integrity
  check; the signature is verified by the `gitops` PR check. Flux keeps cosign verification.
- Flux matches only issuer and subject of the signature; that is enough because the subject names the
  publishing repository and `@refs/heads/main` (see the 2026-11-08 signing decision).
- GHCR packages are public, so Flux and Argo CD pull without credentials.
- CloudNativePG application credentials are a Kubernetes Secret (see Secrets).
- Prod sync is automated with self-heal; the gate is the PR, not a manual sync button (that would
  need human write access to Argo CD for prod).

## Open

### Delivery

- Preview ApplicationSet uses the `pullRequest` generator without a token (public repositories, 60
  requests/hour, long requeue). Revisit with a GitHub App token from OpenBao if rate-limited.

### Paved road

- What `#App` covers beyond v0 (`port`, `size`, `public`, `database`, `secrets`, `network`,
  `env`): workers, CronJobs, queues.

### Platform

- **ClusterRole contents of a component are not inspected:** the whitelist pins names, so a
  compromised chart could ship a stronger ClusterRole under a whitelisted name. Candidate: a CI
  diff gate on RBAC rules, or a per-component max-rules declaration.
- **metrics-server uses `--kubelet-insecure-tls`** until Talos `rotate-server-certificates` + a CSR
  approver are in place.
- **Domain.** The render uses a placeholder (`homelab.example`) until a real domain is chosen;
  wildcard DNS and cert-manager DNS-01 need it.
- **Private access: NetBird** (cloud Free: 5 users, 100 machines): Kubernetes operator with API
  server proxy, or a routing peer to 6443/50000. Break-glass never depends on it.
- **Human identity** for web UIs (Argo CD, Grafana, Hubble, OpenBao): GitHub via Dex, Pocket ID,
  Kanidm, or the private network identity only.
- **Observability:** VictoriaMetrics, VictoriaLogs + Vector, Grafana; API audit log shipped where
  admins can't delete it.
- **Backups / DR** for etcd, PVs, CloudNativePG (Barman Cloud plugin), OpenBao snapshots; object
  storage (Garage in-cluster vs R2).
- **Public exposure** of the example apps: Cloudflare Tunnel or private-network only.
- **Renovate runner:** Mend-hosted app now; self-hosted CronJob with the App key in OpenBao later.
- **Runners:** GitHub-hosted (fine for public repositories); self-hosted in Kata later.

### Org

- **Solo maintainer vs two-person rules:** second account, or documented 1-approval rules.
- **2FA requirement:** enable in the UI (not settable via the API).
