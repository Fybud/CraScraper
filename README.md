# CraScaper

Influencer discovery and campaign workspace for agencies. Search a local catalog of **qualified** Instagram creators, select profiles, and run campaigns.

## Demo login

- Email: `agency@fybud.com`
- Password: `password123`

## Local

```bash
cp .env.example .env
docker compose up --build -d
```

RabbitMQ and crawlers stay off unless you pass `--profile collect`.

- App (same-origin `/api`): http://localhost:8081
- Frontend only: http://localhost:8080
- API: http://localhost:4001/api/health

Keep `VITE_API_BASE_URL` empty locally so the browser calls `/api` on the same origin.

A crawler that hits HTTP 429 **stops the whole fleet** (containers stay up, but they do not fetch). Logs will say `Rate limited`. Resume after a cooldown:

```sql
UPDATE crawler_control SET paused = false, reason = NULL, paused_at = NULL, updated_at = NOW() WHERE id = 1;
```

Crawlers notice within about a minute. 404s and login walls still skip that one profile only.

## Production (Fybud Deploy)

| Service | Domain |
| --- | --- |
| Frontend | https://crascraper.fybud.com |
| API | https://api.crascraper.fybud.com |

Deploy: root [`docker-compose.deploy.yml`](./docker-compose.deploy.yml) + [`DEPLOY.md`](./DEPLOY.md).
Rules: [`AGENTS.md`](./AGENTS.md).

Push to `main` → Actions builds `fybud/crascraper-*` → Fybud Deploy pulls, allocates
`127.0.0.1:${*_HOST_PORT}`, writes host nginx + Cloudflare DNS. Paste `JWT_SECRET` once in the
Deploy UI (not host ports / `DATABASE_URL`), then approve.

Keep `VITE_API_BASE_URL` empty in the image so the SPA calls same-origin `/api` (Deploy nginx
proxies that path to the API).

Catalog restore (on the VPS after first deploy, if needed):

```bash
chmod +x scripts/restore-catalog.sh
./scripts/restore-catalog.sh crascraper.sql
```

## Collection pipeline

```
Seed CSV
   ↓
Candidate Queue
   ↓
Fetch permitted profile data
   ↓
Is profile usable?
   ├── No → discard / mark rejected
   └── Yes
        ↓
   Store / update creator
        ↓
   Extract permitted public content
        ↓
   Calculate metrics
        ↓
   Classify niche
        ↓
   Creator Catalog
        ↓
   Need more creators?
        │
        ├── Yes → discover additional candidates
        │          from permitted public following/followers
        │          and supplied URLs
        │
        └── No → Done
```

Usable means: public profile accessible, **≥ 1,000 followers**, public content. Metrics stay NULL when public post numbers are missing. Agency search reads Postgres only.
