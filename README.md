# Clay

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Clay is an AI-native sales prospecting and GTM data orchestration platform built around Clay Tables — spreadsheet-like workspaces that combine 150+ data providers, AI research agents, and outbound automations. Clay's developer surface is integration-oriented rather than a traditional public REST API: HTTP API enrichment columns, incoming and outgoing webhooks, a Chrome extension, and native connectors to CRMs, sequencers, and data providers. External systems push data into Clay Tables and consume enriched rows via webhooks or exports back to systems of record.

**Areas:** Prospecting · Go-To-Market · Sales · Data Enrichment · Automation · Artificial Intelligence · Webhook · A2A

## Clay on APIs.io

Everything profiled for Clay is published on APIs.io rather than in this repository. Start at the provider page — **[apis.io/providers/clay-com/](https://apis.io/providers/clay-com/)** — or go straight to one of the APIs or properties below.

### APIs (11)

- [Clay HTTP API Enrichment](https://apis.io/apis/clay-com/http-api/) — Generic HTTP request column inside Clay Tables that lets users call any external REST or GraphQL endpoint with row-level variables and write the response…
- [Clay Incoming Webhooks](https://apis.io/apis/clay-com/webhooks-incoming/) — Webhook source that pushes data from any external system into a Clay Table as new rows. Used to trigger Clay workflows from CRMs, signal providers, and…
- [Clay Outgoing Webhooks](https://apis.io/apis/clay-com/webhooks-outgoing/) — Action column that POSTs enriched row data to an external webhook URL, used to deliver Clay-produced data to CRMs, sequencers, Slack, and custom backends.
- [Clay Tables](https://apis.io/apis/clay-com/tables/) — The core spreadsheet-style workspace that combines source rows, enrichment columns, conditional logic, AI agents, and write-back destinations. Tables are…
- [Clay Integrations](https://apis.io/apis/clay-com/integrations/) — Catalog of native connectors and 150+ data providers — Salesforce, HubSpot, Pipedrive, Apollo, Clearbit, ZoomInfo, LinkedIn Sales Navigator, Smartlead,…
- [Clay Chrome Extension](https://apis.io/apis/clay-com/chrome-extension/) — Browser extension for scraping LinkedIn profiles and other web pages directly into Clay Tables, used to bootstrap prospect lists from manual research.
- [Clay MCP Server](https://apis.io/apis/clay-com/mcp/) — Clay's hosted remote Model Context Protocol server, letting an MCP client search companies and people, run enrichment routines and query Clay tables from…
- [Clay Me API](https://apis.io/apis/clay-com/clay-com-me-api/) — Authenticated user and workspace endpoints.
- [Clay Routines API](https://apis.io/apis/clay-com/clay-com-routines-api/) — Routine discovery and execution endpoints.
- [Clay Search API](https://apis.io/apis/clay-com/clay-com-search-api/) — Search creation and pagination endpoints.
- [Clay Tables API](https://apis.io/apis/clay-com/clay-com-tables-api/) — Structured table query endpoints.

### Published properties

Each property below is a page on APIs.io carrying the machine-readable artifact and what it says.

#### AsyncAPI (1)

Event-driven and webhook contracts.

- [Clay Com Webhooks](https://apis.io/asyncapis/clay-com/clay-com-webhooks/)

#### GraphQL (1)

GraphQL schemas.

- [Moved to /providers/clay-com/](https://apis.io/graphqls/clay-com/clay-com-graphql/)

#### MCP servers (1)

Model Context Protocol servers that expose the API to agents.

- [clay](https://apis.io/servers/clay-com/clay/)

#### Agentic access (1)

Which operations an agent can call, which act, and which need a human in the loop.

- [Clay Com Agentic Access](https://apis.io/agentic-access/clay-com/clay-com-agentic-access/)

#### Overlays (1)

OpenAPI overlays, including conversational phrasing for each operation.

- [Clay Com Public Api Overlay](https://apis.io/overlays/clay-com/clay-com-public-api-overlay/)

#### Security (4)

Authentication, trust centre, and vulnerability disclosure.

- [Clay Com Authentication](https://apis.io/security/clay-com/clay-com-authentication/)
- [Clay Com Domain Security](https://apis.io/security/clay-com/clay-com-domain-security/)
- [Clay Com Trust Center](https://apis.io/security/clay-com/clay-com-trust-center/)
- [Clay Com Vulnerability Disclosure](https://apis.io/security/clay-com/clay-com-vulnerability-disclosure/)

#### OAuth scopes (1)

The permissions the API asks for.

- [Clay Com Scopes](https://apis.io/scopes/clay-com/clay-com-scopes/)

#### Plans and pricing (1)

Published plans, tiers and prices.

- [Clay Com Plans Pricing](https://apis.io/plans/clay-com/clay-com-plans-pricing/)

#### Rate limits (1)

Published request limits.

- [Clay Com Rate Limits](https://apis.io/rate-limits/clay-com/clay-com-rate-limits/)

#### FinOps (1)

What the API costs to run, by operation.

- [Clay Com Finops](https://apis.io/finops/clay-com/clay-com-finops/)

## Where to find this API

Links to the company's own pages. This repository publishes no copy of their specifications — the canonical document is theirs and lives at the URLs below.

- [Website](https://www.clay.com)
- [Documentation](https://university.clay.com/docs)
- [Getting started](https://developers.clay.com/quickstart)
- [API reference](https://developers.clay.com/api-reference/me/get-the-authenticated-user)
- [Pricing](https://www.clay.com/pricing)
- [Sign up](https://app.clay.com/signup)
- [Support](https://community.clay.com)
- [Changelog](https://www.clay.com/changelog)
- [Blog](https://www.clay.com/blog)
- [Security](https://trust.clay.com)
- [Compliance](https://www.clay.com/enterprise)
- [Terms of service](https://www.clay.com/terms)
- [Privacy policy](https://www.clay.com/privacy)
- [GitHub](https://github.com/clay-run)
- [Status](https://status.clay.com)

## API Evangelist rating

**Kin Score 64.1** — strong · rubric 0.25.0 · scored 2026-10-23

| Facet | Score |
|---|---|
| Discoverability | 75.0 |
| Contract quality | 57.7 |
| Contract governance | 4.5 |
| Operational transparency | 73.7 |
| Developer ergonomics | 71.4 |
| Access clarity | 75.0 |

**Agent readiness 60.5** — agent-ready (gated down from agent-native)

| Agent-readiness dimension | Result |
|---|---|
| Spec Presence | yes |
| Agentic Access | derived (partial credit) |
| Reversibility Documented | — |
| MCP Server | verified |
| Auth Clarity | served (partial credit) |
| Idempotency | — |
| Error Semantics | verified |
| OpenAPI Examples | — |
| Rate Limit Signal | documented (partial credit) |
| Event Surface Described | yes |
| Agent Skills | yes |
| Well Known Catalog | — |
| Consent Identity | — |
| Agent Card | — |
| Dry Run Mode | — |
| Delegated Identity | served (partial credit) |
| Protected Resource Metadata | verified |
| Dynamic Client Registration | yes |
| Agentic Commerce | — |
| Pagination Ratio | paginated |
| Operation Distinctiveness | distinct |

**Regulatory alignment 38.2** — Horizontal (data, software, accessibility, platform)

The rubric is published at [github.com/api-evangelist/kin-score](https://github.com/api-evangelist/kin-score). The per-check breakdown behind these numbers — which checks were earned, partial or missed — along with score history and how this provider compares to its peers, is available at [apis.io/providers/clay-com/](https://apis.io/providers/clay-com/).

## How this profile is made

API Evangelist profiles a company's public API surface the way a developer would find it: the website, developer portal and documentation, the specifications it publishes, its public repositories, and its pricing, status and changelog pages. From that it derives the properties above — contracts split per API, workflows, schemas, collections, governance rules, plans and rate limits — and scores them against the published [Kin Score rubric](https://github.com/api-evangelist/kin-score).

The artifacts are published on [APIs.io](https://apis.io), not in this repository. This repository is the public record for Clay: the place to say something is wrong, missing or out of date, and to ask for a re-score or a removal.

## Open an issue

- **Something about this listing** — a wrong description, a missing API, a stale artifact or a question about the rating: [open an issue on this repository](https://github.com/api-evangelist/clay-com/issues/new).
- **A new API, or one that is not in the catalog yet** — [submit it on APIs.io](https://apis.io/add/).
- **Anything else** — [the APIs.io inbox](https://github.com/api-search/inbox/issues/new) or [info@apievangelist.com](mailto:info@apievangelist.com).
