# CraScraper tenant stacks (fybud Deploy)

Image-only Compose. Shared Postgres = `fybud-postgres` on `fybud-net`.

| Folder  | Domains                                            | DB                |
| ------- | -------------------------------------------------- | ----------------- |
| `demo/` | `crascraper.fybud.com`, `api.crascraper.fybud.com` | `crascraper-demo` |

GitHub Actions: `.github/workflows/build-push.yml` → `fybud/crascraper-*`.
