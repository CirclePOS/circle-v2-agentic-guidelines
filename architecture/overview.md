# Architecture Overview — Circle V2

## What It Is

Circle V2 is a full-stack POS (Point of Sale) platform — a 3-in-1 system serving bookshop clients with a website, POS terminal, and inventory management.

- **Backend:** Rails 8 app serving REST API (`/api/v1/`) and views
- **Frontend (New):** **Hotwire** (Rails views with Turbo/Stimulus) — primary for new backoffice UI
- **Frontend (Legacy):** Vue 3 TypeScript Turborepo monorepo in `app/vue3/` — two apps: backoffice (maintenance only) and pos (web + mobile via Capacitor)
- **Mobile:** POS app via Capacitor (iOS/Android)

---

## Key Infrastructure

| Component | Version / Detail |
|-----------|-----------------|
| Ruby on Rails | 8.0 — Gemfile at `gemfiles/rails8_0/Gemfile` |
| Database | MySQL 8.4 — schema format: SQL (not Ruby) |
| Search | Elasticsearch 8.15 — models include `elasticsearch-model` |
| Cache / Queue | Redis 6.2 |
| Background jobs | Sidekiq (workers) + ActiveJob |
| Cron | Clockwork (`config/clockwork.rb`) |
| Auth | Doorkeeper + OpenID Connect (OAuth2) |
| CI | Docker Compose (local), GitHub Actions (CI) |

---

## Environment Setup

All services run via Docker Compose:

```bash
docker compose up app8                        # ES, Redis, MySQL + Rails on :3008
docker compose --profile sidekiq up job8      # Sidekiq workers
cd app/vue3 && npm run dev                    # Vue3 devservers (:1233 backoffice, :5173 pos)
```

---

## Deployment Environments

| Environment | Branch    | Database              |
|-------------|-----------|----------------------|
| Alpha       | `alpha4`  | Copy of production   |
| Beta        | `master4` | Same as production   |
| Production  | —         | Production (tagged)  |

---

## Architecture Principles

1. **Layered backend** — MVC extended with Commands, Services, Queries, Entities, Serializers
2. **Command pattern for API** — all `api/v1/` controllers use `Core::Api::GenericActionBehavior` wired to Commands
3. **Tell don't ask** — models expose computed values; views never compute
4. **Hotwire for new backoffice** — Vue stays for POS and legacy backoffice fixes only
5. **Feature flags** — new pages gated behind `current_site.feature?(:name, :flag)` until QA sign-off

See `backend-patterns.md` and `frontend-patterns.md` for implementation details.
