# meta — documentation of the homelab-selfuryon org

Documentation only: no code, no build. It answers **"which repository holds the thing I am looking
for?"**, **"how does a change reach the cluster?"** and **"what did we decide, and what is still
open?"**

- [`README.md`](README.md) — the index: repository table, the picture, the stack.
- [`delivery.md`](delivery.md) — flows and the exact contracts between repositories (names,
  artifacts, signing identities, `deploy.cue`, `gitops` layout). Repositories are built against it.
- `<repo>.md` — one card per repository (`dot-github.md` for `.github`).
- [`decisions.md`](decisions.md) — decision log plus open questions.
- [`security-baseline.md`](security-baseline.md) — the principles every repository follows.
- [`org.md`](org.md) — the GitHub organization and its settings.

## Using it

Start at `README.md`. Open the card before working in a repository, then that repository's own
`CLAUDE.md`: cards don't duplicate build commands or conventions, those belong next to the code. If a
card and the repository disagree, the repository is right — fix the card. A change to an interface
in `delivery.md` is made together with the repositories it affects.

## Maintaining it

- Update a card in the same change that moves a boundary, adds a subsystem or creates a repository.
- New repository: card + row in `README.md`.
- A decision goes into `decisions.md` with its date and reason. Rejected options are struck through,
  not deleted.
- Keep cards short; anything `ls` or `grep` answers doesn't belong here.
- Dates are absolute (`2026-11-07`).
- English only. Everything here is public: no hosts, IPs, account names beyond the org, or secrets.
