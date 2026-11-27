# catalog

**Owner:** platform; each team owns its `teams/<team>.cue` · GitHub: `homelab-selfuryon/catalog` · Status: **in use** (2026-11-26)

The one place where teams and apps are declared: the facts every repository must agree on (which
teams and apps exist, ownership, tier, lifecycle, environments, cluster). From them each repository
builds what a team or an app gets by default — the GitHub team, repositories and environments,
OpenBao roles, namespaces and quotas, Argo CD objects, `CODEOWNERS` lines — so onboarding is one PR
here instead of a PR in every repository. Settings of one system (quotas, GitHub permission) live in
that system's repository (decisions.md, 2026-11-26).

- `schema/` — `#Team`, `#App`, defaults. A team file says only how it differs.
- `teams/<team>.cue` — one file per team (package `teams`): the team (description; `tenant: false`
  for a team without namespaces) and its apps (description, tier, lifecycle, environments,
  cluster). A layout check makes each file hold only its own team.
- `catalog.cue` — builds the catalog from the `teams` package.
- `views.cue` — per repository, the fields it reads; it receives only that view.
- `cue cmd generate` — writes `catalog.json`, defaults resolved, and `<repository>.json` per view: the
  contract in [delivery.md](delivery.md#catalog-catalogjson).

`platform-terraform`, `platform-components` and `gitops` read `catalog.json` (the table in
[delivery.md](delivery.md#catalog-catalogjson)). On `main`, after the tests, CI (job `sync`) opens or
updates a PR with its tested view in each of them (branch `catalog/sync`, octo-sts identity
`catalog-sync` with its policy in each repository), without waiting for any of them; in `gitops` the
PR also carries what `gitops_checks.py sync` derives. The owner merges `platform-terraform` first and
applies it, then the others. Each repository's own checks validate what arrives.

Decision: [decisions.md](decisions.md) (2026-11-26).
