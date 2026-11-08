# notes-stats

**Owner:** team `notes` · GitHub: `homelab-selfuryon/notes-stats` · Status: **scaffolded** (2026-11-07)

Example service showing service-to-service auth without shared keys: `GET /v1/stats` calls
`notes-api` with its projected ServiceAccount token (audience `notes-api`, re-read on every call
because the kubelet rotates it). No database, not public.

Related: [notes-api](notes-api.md).
