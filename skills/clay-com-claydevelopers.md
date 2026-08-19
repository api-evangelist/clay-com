---
name: Claydevelopers
description: Use when building agent workflows, backend integrations, or CLI-based automation that needs to search GTM data, run enrichment functions, or execute custom business logic. Clay provides structured access to company and people data, reusable routines (functions and workflows), and batch execution capabilities.
metadata:
    mintlify-proj: claydevelopers
    version: "1.0"
---

# Clay Developer Platform

## Product summary

Clay is a GTM data and automation platform with a developer API and CLI. Use it to search companies and people, run enrichment functions, execute custom business logic, and batch-process records. The primary entry points are the **Clay CLI** (`clay` command) for agent workflows and local scripts, and the **Public API** (REST endpoints) for backend services and integrations. Key file paths: API keys live in `Settings → Account → API keys (beta)` in the Clay UI. CLI authentication is stored in `~/.config/clay/` (or `$CLAY_CONFIG_HOME/clay/`). Routines are identified by `function:t_...` (functions) or workflow names. See https://developers.clay.com for full documentation.

## When to use

Reach for Clay when:
- An agent needs to search for companies or people matching specific criteria (job titles, industries, locations, etc.)
- You need to enrich records with structured data (emails, phone numbers, company info, tech stack, funding)
- You're building a backend job or internal tool that calls reusable Clay logic
- You want to run the same operation across many records (batch execution)
- You're building a workflow in Claude Code, Codex, or Cursor and need to create or edit logic outside the Clay UI
- You need to query existing Clay table data with structured filters

Do not use Clay for: general-purpose web scraping, real-time streaming data, or operations that don't fit the search-enrich-batch pattern.

## Quick reference

### CLI commands

| Command | Purpose |
|---------|---------|
| `clay login` | Authenticate with OAuth (opens browser) |
| `clay login --device` | Authenticate on headless machines (prints link + code) |
| `clay whoami` | Verify authentication is working |
| `clay logout` | Clear stored credentials |
| `clay routines list` | List available routines (functions and workflows) |
| `clay routines runs start <routine_id> --input <json>` | Run a single routine with JSON input |
| `clay routines runs start <routine_id> --bulk rows.jsonl` | Batch-run a routine with JSONL input |
| `clay routines runs get <run_id>` | Fetch results of a run |
| `clay searches create --source-type people --query <query>` | Create a search (advanced mode) |
| `clay searches run <search_id>` | Fetch results from a search |

### API endpoints (base: `https://api.clay.com/public/v0`)

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/me` | GET | Get authenticated user and workspace |
| `/search/filters-mode` | POST | Create a search with structured filters |
| `/search/filters-mode/{search_id}/run` | POST | Fetch next page of filter-mode search results |
| `/search/query-mode` | POST | Create a search with advanced query syntax |
| `/search/query-mode/{search_id}/run` | POST | Fetch next page of query-mode search results |
| `/routines/{routine_id}/run` | POST | Run a routine synchronously (1-100 items) |
| `/routines/{routine_id}/run-batch/upload-url` | POST | Get presigned URL for JSONL upload |
| `/routines/{routine_id}/run-batch/start` | POST | Start a batch run with uploaded file |
| `/routines/run-batch/{batch_run_id}/results` | GET | Poll batch run progress and results |
| `/tables/query` | POST | Query Clay table data with structured filters |

### Authentication

- **CLI**: `clay login` stores OAuth token in `~/.config/clay/config.json`
- **API**: Pass `clay-api-key` header on every request; store key in `CLAY_PUBLIC_API_KEY` env var
- **API key creation**: Settings → Account → API keys (beta) in Clay UI, or ask agent plugin to generate one

### Routine IDs

- Clay-managed functions: `function:t_<id>` (e.g., `function:t_abc123`)
- Custom functions: `function:t_<id>` (same format)
- Workflows (Alpha): `workflow:<name>` or workflow ID from `clay workflows list`

### Input/output formats

- **Single runs**: JSON object with `items` array, each item has `id` and `inputs`
- **Batch runs**: JSONL (newline-delimited JSON), each line is `{"id": "...", "inputs": {...}}`
- **Search results**: Paginated with `data` array and `has_more` boolean (stateful iterator)
- **CLI output**: JSON to stdout on success, error envelope to stderr on failure

## Decision guidance

### When to use CLI vs. API

| Scenario | Use |
|----------|-----|
| Agent workflow in Claude Code, Codex, or Cursor | CLI (handles auth locally, returns JSON) |
| Backend service, queue worker, or cron job | API (no shell dependency, direct HTTP) |
| Local testing or debugging | CLI (faster iteration, no API key management) |
| Product feature or user-facing integration | API (more control, no subprocess overhead) |
| Batch processing with inspection/snapshots | CLI (better debugging output) |

### When to use search mode

| Scenario | Use |
|----------|-----|
| Complex criteria: people at companies with specific firmographics | Advanced search (beta) with query syntax |
| Simple filters: job titles, industries, locations | Structured filters (filters mode) |
| Existing filters-mode searches | Filters mode (backward compatible) |
| Cross-entity logic: "people at companies where..." | Advanced search (beta) |

### When to use routine type

| Scenario | Use |
|----------|-----|
| Clay-provided enrichment (emails, company data, tech stack) | Clay-managed functions |
| Team-built logic already in Clay UI | Custom functions |
| Agent-first building, code inside flow, run inspection | Workflows (Alpha) |
| Avoid 50,000 row batch limit | Workflows (Alpha) |
| Existing reusable logic, just need to call it | Custom functions |

## Workflow

### Typical search-and-enrich flow

1. **Authenticate**: Run `clay login` (CLI) or create API key in Settings (API).
2. **Discover search fields**: Call `/search/filters-mode/fields?source_type=people` (API) or use advanced search reference to understand available criteria.
3. **Create search**: POST to `/search/filters-mode` with `source_type` and `filters` object, or `/search/query-mode` with a query string.
4. **Fetch results**: POST to `/search/filters-mode/{search_id}/run` with `limit` parameter; repeat while `has_more` is true.
5. **Choose function**: Identify a Clay-managed function (e.g., find emails) or custom function that matches your enrichment need.
6. **Run function**: Call `/routines/{routine_id}/run` with search results as input, or use `clay routines runs start` from CLI.
7. **Fetch results**: Poll `/routines/run/{run_id}` until `status` is terminal (e.g., `completed`, `failed`).
8. **Send output**: Pass structured results to your CRM, warehouse, agent, or downstream system.

### Typical batch-run flow

1. **Prepare JSONL**: Create a file with one JSON object per line: `{"id": "row-1", "inputs": {...}}`.
2. **Get upload URL**: POST to `/routines/{routine_id}/run-batch/upload-url` to get presigned URL.
3. **Upload file**: PUT JSONL to the presigned URL with `Content-Type: application/x-ndjson`.
4. **Start batch**: POST to `/routines/{routine_id}/run-batch/start` with returned `file_id`.
5. **Poll progress**: GET `/routines/run-batch/{batch_run_id}/results` until `status` is terminal.
6. **Download results**: Fetch result file URL from final response and download results.

### Typical Workflow (Alpha) flow

1. **Create Workflow**: Use agent plugin or `clay workflows create` to scaffold a new workflow.
2. **Edit graph**: Add trigger (CSV, webhook, table, etc.), then connect nodes (agent, enrich, code).
3. **Validate**: Run `clay workflows validate` to check graph structure.
4. **Test**: Run `clay workflows run --test` to execute a single test item.
5. **Inspect**: Use `clay workflows inspect <run_id>` to see step-by-step execution and failures.
6. **Batch run**: Use `clay workflows runs start --bulk rows.jsonl` for large datasets.
7. **Snapshot**: Restore previous snapshots with `clay workflows snapshots restore` if needed.

## Common gotchas

- **API key exposure**: Never commit API keys to version control, logs, or client-side code. Store in environment variables and pass via `clay-api-key` header only.
- **CLI output parsing**: The CLI is JSON-first; parse stdout with `jq`, not text patterns. Check exit code (`$?`) to detect errors before parsing.
- **Search result limits**: Free plans cap at 50 results per request and 100 per month. Paid plans scale to 1M/year. Hitting a limit returns HTTP 402; check the error message for which limit was exceeded.
- **Batch run row limit**: Functions have a 50,000 row limit per batch. Use Workflows (Alpha) to avoid this limit.
- **Async polling**: Batch runs and async routines return 202 (still running). Poll the result endpoint at a modest interval (not as fast as possible) to avoid rate limiting.
- **Rate limiting**: 429 responses include `Retry-After` header. Back off and retry; the CLI exits with code 4 and includes `retryAfter` in error details.
- **Pagination state**: Search results use stateful iterators (`has_more` boolean), not cursors. Call the endpoint again while `has_more` is true; don't assume you can skip pages.
- **Workflow Alpha**: Workflows are in Alpha; expect the surface to evolve. For stable, reusable logic, prefer custom functions.
- **Function IDs**: Always prefix with `function:` when calling from CLI or API (e.g., `function:t_abc123`). Omitting the prefix causes 404 errors.
- **JSONL format**: Batch input must be strict JSONL (one JSON object per line, no trailing commas, no comments). Invalid JSONL causes upload or parsing errors.
- **Search mode mismatch**: Don't mix advanced search and filters-mode queries. Choose one mode per search; they use different endpoints and syntax.

## Verification checklist

Before submitting work with Clay:

- [ ] API key is stored in an environment variable, not hardcoded.
- [ ] CLI is authenticated: `clay whoami` returns user and workspace.
- [ ] Search query is valid: test with `/search/filters-mode/fields` or query reference before creating search.
- [ ] Routine ID is correct: verify with `clay routines list` and includes `function:` prefix if needed.
- [ ] Input JSON matches routine's expected schema: check routine details in Clay UI or API response.
- [ ] Batch JSONL is valid: each line is a complete JSON object, no trailing commas or comments.
- [ ] Async operations are polled correctly: check `status` field is terminal before reading results.
- [ ] Error handling covers 401/403 (auth), 404 (missing), 429 (rate limit), 5xx (retry).
- [ ] Rate limit headers are respected: back off on 429 using `Retry-After` value.
- [ ] Pagination is complete: continue fetching while `has_more` is true or `cursor` is present.

## Resources

- **Comprehensive page listing**: https://developers.clay.com/llms.txt
- **API Reference**: https://developers.clay.com/api-reference/me/get-the-authenticated-user
- **Routines (functions and workflows)**: https://developers.clay.com/routines
- **Searches (find companies and people)**: https://developers.clay.com/searches

---

> For additional documentation and navigation, see: https://developers.clay.com/llms.txt