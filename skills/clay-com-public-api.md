---
name: public-api
description: 'Clay Public API — HTTP access for building services, apps, and integrations: GTM database search, structured table queries, and async routine and batch runs.'
---

# The Clay Public API

Beyond the CLI, Clay exposes a Public API you can develop against directly over HTTP.
Reach for it when building a service, app, or integration — not for one-off agent tasks
in a shell (use the `cli` skill for those).

## What it offers

- **Search** — find people or companies in Clay's GTM database.
  - **Advanced queries (beta)** are the default and support filters-mode criteria, cross-entity filters, and nested Boolean logic.
  - Use **structured filters** (filters mode) if the user prefers its older structure or has existing filters-mode searches.
    See the `search` skill for search behavior and the CLI equivalent. Prefer the CLI for one-off searches.
- **Tables** — structured queries against Clay tables.
- **Routines / batches** — async routine and batch runs.

## Auth

The public API needs its **own** key — the credential from `clay login` is **not**
scoped for it. Issue a dedicated public-API key with the CLI:

```bash
clay api-keys create --name "<name>"   # → { ..., "apiKey": "<secret>" }
```

CLI-created keys are always scoped to the public API. The `apiKey` secret is returned
**only once**, at creation — store it immediately; it can't be retrieved later. Send it
as a Bearer token against `https://api.clay.com/public/v0`. Manage existing keys with
`clay api-keys list | update | delete`.

## Reference

Full developer documentation — Public API reference, CLI reference, concepts, and the
OpenAPI spec — lives at:

- <https://developers.clay.com/llms.txt>

Fetch that first to get exact endpoints, request/response shapes, pagination, and rate
limits before writing integration code.
