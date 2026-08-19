---
name: audiences
description: Clay Audiences — the workspace's own people, companies, and deals (contacts, leads, accounts, customers). Use for any request about their records when no surface is named, including counts, fill rates, lookups ("how many people have a phone?"), saved segments, and field definitions. Also deal and pipeline questions like closed-won, open pipeline, deal stage, and ACV.
---

# Clay Audiences

Audiences is Clay's CRM-shaped store of **people** and **companies** (plus deal
records). An **audience** — also called a saved segment — is a named filter over
one entity type. Nothing is copied into it: it selects records live, so its
membership changes as records change.

**Audiences is the default home for the workspace's own people and companies.**
When a user mentions people, companies, contacts, leads, accounts, or customers
without naming a surface, they mean these records — start here, not in `tables`
(a separate surface, right only when the user names a table) and not in `search`
(net-new prospects that are not in the workspace yet).

Read this before any audiences work. Four supporting references:

- `answering-data-questions.md` — **read this first for any "how many / who has /
  look up X" question.** Covers reading existing fields before paying for an
  enrichment, checking fill rates, and what to do when the data is mostly missing.
- `filters.md` — writing the filter AST that defines an audience. Read it before
  you author or edit a filter.
- `custom_objects.md` — **deals / opportunities.** Read it before anything that
  touches them, including GTM phrasings that mean deals: closed-won, closed-lost,
  open pipeline, deal stage, deal size / ACV / ARR, close date, forecast, win rate,
  renewal, expansion, churn, "our customers". Deals are read-only, and a deal
  query is rooted at **people or companies** — you filter people or companies
  by their deals rather than filtering deals directly.
- The `workflows` skill's `audiences.md` — writing values onto records
  (the `upsert-audiences-record` action) and triggering a workflow off an audience.

**Read with the CLI, write records with the action.** `clay audiences` covers
every Audiences primitive — segments, fields, and reading records — but it has no
command that writes a field value onto a record. That is the
`upsert-audiences-record` action's job.

**Audiences is CLI-only.** No MCP tool serves it — `surfaces_read` and
`surfaces_list` carry no `audience` resource type, and nothing else in the tool
list reads segments, fields, or records. That absence is not a missing
capability: use `clay audiences`, and don't report Audiences as unavailable
because no tool offers it.

## One entity type, three spellings

The same entity types are named differently depending on where you are. Get this
mapping right up front — it is the most common source of wasted round trips.

| CLI `--entity-type` | Filter AST / action `entityType` | Filter `dataPath` root        |
| ------------------- | -------------------------------- | ----------------------------- |
| `people`            | `CONTACT`                        | `contact_entity_field_values` |
| `companies`         | `ACCOUNT`                        | `account_entity_field_values` |
| `deals`             | `CUSTOM` (Opportunity)           | `opportunity`                 |

Everything under `clay audiences records`, plus `fields list`, accepts `deals`;
the audience commands and the other `fields` subcommands take `people` or
`companies` only. CLI output can also carry `entityType: "deals"`. Note that a
_selective_ deal query is written as a `people` or `companies` search whose filter
reaches into deals — for anything deal-shaped, read `custom_objects.md` first.

Workflow **triggers** use the middle spelling: an `audience_segment` trigger's
`segmentId` is the audience id from `clay audiences list`, and its `entityType`
is `CONTACT` / `ACCOUNT`, not `people` / `companies`.

## Audiences (saved segments)

```bash
clay audiences list --entity-type people          # id, name, entityType (no filter); 50/page, pass back .cursor
clay audiences get <audienceId>                   # same, plus the full filter AST
clay audiences create --entity-type people --name "Missing emails" --filter ./filter.json
clay audiences update <audienceId> --name "…" --description "…" --filter ./filter.json
clay audiences archive <audienceId>               # soft delete, idempotent; records untouched
```

- `create` and `update` take `--filter` as inline JSON, **a file path, or `-` for
  stdin**. Use a file or stdin for anything non-trivial — inline JSON in a shell
  triggers an approval prompt and invites quoting mistakes.
- `update` replaces the whole filter; there is no partial merge, and entity type
  is immutable after creation. Omitted flags are left alone.
- `get` returns an id-free filter, so `clay audiences get <id> | jq .filter` pipes
  straight back into `create --filter -` to clone an audience.

## Fields

Field ids are what record payloads and filter ASTs key on, and they are scoped
to an entity type — every subcommand takes `--entity-type`.

```bash
clay audiences fields list --entity-type people                       # every field: id, name, dataType, fieldType, hidden
clay audiences fields list --entity-type people --filter id=email     # narrow by id (repeatable, comma-separated)
clay audiences fields list --entity-type people --include-system      # system fields, hidden by default
clay audiences fields create --entity-type people --name "Lead score" --data-type number
clay audiences fields update <fieldId> --entity-type people --hidden true
clay audiences fields delete <fieldId> --entity-type people
clay audiences fields segments <fieldId> --entity-type people         # audiences whose filter references the field
clay audiences fields list --entity-type deals                        # deal fields; list is the one subcommand taking deals
```

- **Run `fields list` once and save it** (`> /tmp/people-fields.json`), then slice
  it with `jq`. Re-running it to grep, head, and parse the same output three
  different ways is pure latency.
- `fields list` returns the workspace record-field catalog (system fields
  excluded) — use those ids for `upsert-audiences-record`.
- **Names are not ids.** Account "company name" is `org_name`. Never guess an id
  from a display name.
- `create` silently uniquifies a taken name (`"Tier (2)"`) — read the returned
  `name` and `id` rather than assuming the one you passed.
- Before `update --data-type` or `delete`, run `fields segments <fieldId>`: a
  delete rewrites saved filters with those clauses **removed**, changing what
  those audiences match. Empty `data` means nothing is affected.
- Default and system fields reject `--name` and `--data-type` changes and cannot
  be deleted; `--hidden`, `--order`, and `--description` still work.

### Default field ids

Present in every workspace, so you can write a filter against these without
listing fields first:

- **people** — `name`, `first_name`, `last_name`, `email`, `linkedin_url`,
  `phone`, `title`, `signal_summary`
- **companies** — `org_name`, `domain`, `headquarters_location`, `linkedin_url`,
  `sfdc_owner_id`, `signal_summary`, `technographics`
- **deals** — see `custom_objects.md`

Anything else is workspace-defined — get its id from `fields list`.

## Records

```bash
clay audiences records search-count --entity-type people --audience-id <id>   # count a scope server-side
clay audiences records search-ids   --entity-type people --audience-id <id>   # matching ids, 50/page + .cursor
clay audiences records get --entity-type people --ids 1,2,3                   # field values, max 100 ids
```

Scope for both search commands (`--audience-id` and `--filter` are mutually
exclusive):

- neither flag → every record of the entity type
- `--audience-id <id>` → a saved audience's records
- `--filter <json|file|->` → an ad-hoc filter, matching exactly what an audience
  built from that filter would hold

Add `--archived` to either to search archived records instead of live ones.

**`search-count` is the workhorse.** It answers "how many" server-side for free
and instantly, and a `NotEmpty` filter on a field turns it into a fill-rate check
— run that before building anything on a field, and before proposing an
enrichment. See `answering-data-questions.md`.

All three commands take `--entity-type deals`, but a deal search accepts **no
scope** — `--audience-id` or a non-empty `--filter` with `deals` is a
`validation_error`. So `--entity-type deals` covers the whole population (total
count, full id enumeration), while a **selective** deal question is a `people` or
`companies` search whose filter reaches into their deals. That root is also what
the user usually wants back — which contacts or accounts the deals belong to. See
`custom_objects.md`.

`search-ids` returns ids only — feed them to `records get --ids` in batches of
100 for field values, keyed by field id (unset fields may be omitted). Records
not found are omitted rather than erroring. To size a scope, use `search-count`,
not a paging loop over `search-ids`.

## Error codes

Every command prints JSON on stdout and a typed error envelope on stderr. Exit
codes: `0` ok, `2` validation, `3` auth, `4` rate-limit, `5` network, `6`
not-found. `3` on an audiences command usually means Audiences is not enabled for
the workspace — that is a workspace-config answer for the user, not something to
retry.

`4` (`rate_limited`) means the workspace spent its request budget for that one
command. The budget is per command and shared by everyone in the workspace, so a
`fields list` loop cannot starve `search-count`. It defaults to 60 calls/minute
and a workspace can be raised above that, so read `details.limit` off the error
rather than assuming the default.

Sleep `details.retryAfter` seconds and carry on. A request-rate rejection costs no
budget and writes nothing, so repeating the identical call — including a `create`
or `update` — once the wait is over is safe.

`records get` carries a second budget: an hourly one charged per record id it
returns, which batching does not reduce. Read its 429s more carefully than the
rest:

- `details.limit` is records/hour (default 100,000), not calls/minute, and the
  message says "Hourly records limit" rather than "Too many requests".
- The rejected call **did** spend a request token — that one is not refunded — so
  the wait it asks for can end in a second 429, this time from the per-minute
  rate. Back off again rather than reading it as a failure.

That budget exists to prevent data exfiltration, so be conservative: before
pulling a large set, check whether a sample of records answers the request.
