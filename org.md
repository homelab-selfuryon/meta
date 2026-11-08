# GitHub organization `homelab-selfuryon`

**URL:** https://github.com/homelab-selfuryon · **Plan:** Free · **Created:** before 2026-11-07
· Settings managed as code in `platform-terraform/github`

## State (`gh api orgs/homelab-selfuryon`)

| Setting | Now (2026-11-07) | Target |
| --- | --- | --- |
| Repositories | created empty 2026-11-07 | see [README](README.md) |
| 2FA required | **off** — not settable via REST API, enable in UI: Settings → Authentication security | on |
| Base repository permission | `none` (set 2026-11-07) | `none` |
| Members can create repositories | off (set 2026-11-07); owners still can. Free plan rejects the "private only" policy, so it is all or nothing | off (repos only via `platform-terraform`) |

## Why a separate org

An org is the unit GitHub applies policy to: org rulesets, custom properties, teams, Actions policy,
OIDC subject prefix (`repo:homelab-selfuryon/*`), and one GHCR namespace for images, deploy
artifacts and CUE modules. A personal account has none of these, so the example would not be
realistic there.

## Free plan constraints

Rulesets, environment required reviewers, secret scanning push protection and code scanning are free
**for public repositories only**. Hence: public by default, nothing sensitive in git.

## Target baseline (in `platform-terraform/github`)

- Rulesets in every repository (org-level rulesets are not on Free): signed commits; PR +
  CODEOWNERS on the default branch; no force-push/deletion; immutable `v*` tags; required check
  `final-check`. Admins may bypass through a PR only, while there is one maintainer.
- Custom properties: `tier` (`tier-1` for platform-kit, platform-components, github-workflows, gitops, platform-terraform), `owner-team`,
  `lifecycle` (schema org-level, values per repository). Adding them to the Actions OIDC token is
  not supported by the Terraform provider yet.
- Actions: read-only `GITHUB_TOKEN`, allowlisted actions pinned by SHA, Actions can't approve PRs,
  fork PR workflows need approval, `pull_request_target`/`workflow_run` banned (zizmor required).
- Secret scanning + push protection, Dependabot alerts, private vulnerability reporting.
- No stored Actions secrets at all; everything via OIDC or `GITHUB_TOKEN`.
- Teams: `platform`, one per product team; CODEOWNERS reference teams, not people.

## Solo-maintainer caveat

Two-person rules (2 approvals, "last push approved by someone else", environment reviewers with
prevent-self-review) can't be satisfied by one person. Either a second maintainer account or the
rule is documented and set to 1 with a note. Open in [decisions.md](decisions.md).
