# notes-api

**Owner:** team `notes` · GitHub: `homelab-selfuryon/notes-api` · Status: **scaffolded** (2026-11-07)

Example app from [rust-app-template](rust-app-template.md): a REST API for notes on
Postgres (CloudNativePG, credentials as files, migrations at startup). `GET /v1/notes/count` accepts
only callers presenting a projected ServiceAccount token with audience `notes-api` from an
allowlisted ServiceAccount — verified against the cluster's JWKS. Public on the environment's
Gateway.

Related: [notes-stats](notes-stats.md), [delivery.md](delivery.md).
