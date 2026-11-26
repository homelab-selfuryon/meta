# catalog

**Owner:** platform; each team owns its `teams/<team>/` · GitHub: `homelab-selfuryon/catalog` · Status: **in use** (2026-11-26)

The one place where teams and apps are declared. Everything a team or an app needs elsewhere — the
GitHub team, repositories and environments, OpenBao roles, namespaces and quotas, Argo CD
registration, `CODEOWNERS` lines — is generated from it, so onboarding is one PR here instead of a
PR in every repository.

- `schema/` — `#Team`, `#App`, quota sizes, defaults. A team file says only how it differs.
- `teams/<team>/team.cue` — the team: description, size, quota overrides, GitHub permission.
- `teams/<team>/<app>.cue` — one file per app: tier, lifecycle, environments, database, previews.
- `catalog.cue` — imports every team package; a layout check keeps it and the directories in step.
- `cue cmd generate` — writes `catalog.json`, defaults resolved: the contract in
  [delivery.md](delivery.md#catalog-catalogjson).

`platform-terraform`, `platform-components` and `gitops` read `catalog.json` (the table in
[delivery.md](delivery.md#catalog-catalogjson)). Next: on merge to `main`, CI generates it and opens
a PR with it in each of them (octo-sts tokens, branch `catalog/sync`), without waiting for any of
them; until then the platform team copies `out/catalog.json` by hand. The owner merges
`platform-terraform` first and applies it, then the others. Each repository's own checks validate
what arrives.

Decision: [decisions.md](decisions.md) (2026-11-26).
