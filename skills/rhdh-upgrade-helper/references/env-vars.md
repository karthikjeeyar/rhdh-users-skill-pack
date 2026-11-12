# Known RHDH Environment Variables

Environment variables that affect RHDH behavior and may change between releases. When `.env` files are found in `--config-path`, parse them and cross-reference with this list.

## File Discovery

Scan `--config-path` for:

- `.env`, `.env.local`, `.env.production` — dotenv files
- `env.sh` — shell export files
- Any file matching `*.env`

Parse `KEY=VALUE` pairs. Ignore comments (`#`) and empty lines. For shell exports, strip the `export` prefix.

## Upgrade-Relevant Environment Variables

### Backstage Config Overrides (`APP_CONFIG_*`)

Environment variables prefixed with `APP_CONFIG_` override Backstage configuration at runtime. These use double-underscore (`__`) as path separators.

| Variable | Affects | Notes |
|----------|---------|-------|
| `APP_CONFIG_app_packageName` | Frontend system selection | In RHDH 1.10, `app-next` selected NFS. Remove this override when targeting RHDH 2.1; the old frontend system and selector are removed. |
| `APP_CONFIG_app_baseUrl` | Application base URL | Must match route/ingress host |
| `APP_CONFIG_backend_baseUrl` | Backend base URL | Must match route/ingress host |
| `APP_CONFIG_backend_database_*` | Database connection | Check for deprecated connection params |

### RHDH-Specific Variables

| Variable | Affects | Version Notes |
|----------|---------|---------------|
| `ENABLE_STANDARD_MODULE_FEDERATION` | New Frontend System | Used with `APP_CONFIG_app_packageName=app-next` in RHDH 1.10. Remove it when targeting RHDH 2.1; NFS is the supported frontend system. |
| `CATALOG_INDEX_IMAGE` | Plugin discovery | Used by init container. Format may change between releases. |
| `CATALOG_ENTITIES_EXTRACT_DIR` | Plugin extension catalog | New in RHDH 1.10 — set to `/extensions`. |
| `MAX_ENTRY_SIZE` | Dynamic plugin install | Affects large plugin packages (e.g., orchestrator). |
| `LOG_LEVEL` | Logging verbosity | No version-specific changes. |
| `BACKEND_SECRET` | Backend auth | Used with `backend.auth.externalAccess`. Check if `type: legacy` is deprecated. |

### Proxy Variables

| Variable | Affects | Notes |
|----------|---------|-------|
| `HTTP_PROXY` | Outbound HTTP connections | Must be set as pod env var, not in app-config |
| `HTTPS_PROXY` | Outbound HTTPS connections | Same |
| `NO_PROXY` | Proxy bypass list | Ensure internal services are excluded |
| `NODE_EXTRA_CA_CERTS` | Custom CA certificates | Path to CA bundle inside container |

## What to Flag

1. **Deprecated variables:** Variables that were removed or renamed between FROM and TO releases.
2. **RHDH 2.1 NFS selectors:** For target 2.1, if either `APP_CONFIG_app_packageName=app-next` or `ENABLE_STANDARD_MODULE_FEDERATION` is present, report that the legacy selector must be removed. Do not flag the pair as a missing-half configuration or claim it is required to enable NFS.
3. **Conflicting overrides:** `APP_CONFIG_*` variables that conflict with values in `app-config.yaml`.
4. **Secrets in values:** Apply `references/secrets-detection.md` to env var VALUES. Flag literal API keys, passwords, or tokens. `${VAR}` references in env files are unusual — they're typically literal values.

## Output

Include discovered environment variables in the report's "Your Environment Summary" table:

| Dimension | Value | Source |
|-----------|-------|--------|
| Legacy NFS selector present | Yes (`APP_CONFIG_app_packageName=app-next`; remove for 2.1) | .env |
| Proxy configured | Yes (`HTTPS_PROXY=...`) | .env |
