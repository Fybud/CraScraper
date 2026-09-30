# CraScraper tenant stacks (FiberAI Deploy)

Image-only Compose. Shared Postgres = `fiberai-postgres` on `fiberai-net`.

| Folder | Domains | DB |
|---|---|---|
| `demo/` | `crascraper.fybud.com`, `api.crascraper.fybud.com` | `crascraper-demo` |

GitHub Actions: `.github/workflows/build-push.yml` → `fiberai/crascraper-*`.
