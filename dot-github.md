# .github

**Owner:** platform · GitHub: `homelab-selfuryon/.github` · Status: **scaffolded** (2026-11-07)

The repository name GitHub treats specially for the whole org:

- `profile/README.md` — shown on the org page; points to `meta`.
- Default community health files used by every repository without its own: `SECURITY.md`
  (private vulnerability reporting), `CONTRIBUTING.md`, issue and PR templates.
- `default.json` — the shared Renovate preset (`local>homelab-selfuryon/.github`): actions pinned by
  digest, automerge for minor/patch (0.x minors treated as breaking), `minimumReleaseAge`, OSV
  alerts, custom managers for `cue.mod` dependencies and the reusable workflows.

Related: [meta](README.md), [platform-terraform](platform-terraform.md).
