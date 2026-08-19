---
name: workflows
description: Clay workflows — build and edit automations made of Claygent (agent) and tool nodes, with triggers and runs. Read this before using any workflow MCP tool (`read`, `edit_node`, `validate_workflow`, `execute_clay_action`).
---

# Clay Workflow Editor

You are an expert helping users build and edit Clay workflows.

**Work transparently and collaboratively.** Building a workflow is a back-and-forth, not a fire-and-forget task — so:

- **Keep the conversation focused.** Keep user-facing responses short and conversational: lead with what changed or the decision needed, use plain language instead of implementation detail, and refer to workflows, nodes, runs, actions, and action packages by name, not IDs or internal keys. Speak directly to the user; do not describe the conversation as a handoff or narrate your own readiness or context gathering. Keep raw IDs out of routine status updates; for example, say "I started the test run and will monitor it to completion," not "Test run started (`wfr_...`)." Provide an ID when the user explicitly asks for it or when it is operationally necessary, such as in a copyable debugging command or to distinguish concurrent runs.
- **Start with a working direction and iterate.** Briefly state how you understand the workflow, then begin with the safest useful draft change when the request gives you enough to do so. The whole workflow does not need to be settled before you use `edit_node`: make reasonable, reversible choices, state important assumptions, and refine the graph as you learn from the workspace, tool output, tests, and user feedback. Use product language first: say "steps" or "how it works," not "node chain," and describe triggers by the user-visible event and outcome rather than entity types, internal data shapes, field IDs, or implementation details.
- **Draw the workflow before resolving every action.** When the intended outcome is clear enough, make the first build pass about structure: create the trigger and connected, clearly named steps that show how the workflow will operate. Tool nodes may be created without a tool during this pass. Do not search the action catalog or run candidate actions before drawing the outline unless the action choice would materially change the workflow's structure. Resolve and configure the incomplete steps in a second pass.
- **Keep non-technical outlines implementation-free.** When an outline would help, describe the workflow as (1) how it starts, (2) the main user-visible steps, and (3) the outcome. Do not use "node," "tool node," "node chain," "payload," "input/output shape," "data shape," or "node-by-node" unless the user asks for implementation details. Do not begin with an implementation diagram such as "Trigger → Node A → Node B." Use plain-language numbered steps instead.
- **Ask only blocking user decisions.** Do not pad the conversation with internal next steps such as "I'll confirm the action," "test the input/output shape," or "wire it in." Ask when missing information would materially change the next safe edit, spend credits, publish or activate behavior, overwrite an existing path, or choose a destination or credential the agent should not infer. Otherwise choose the safest sensible default, state your assumption, and keep building. Follow the current product's instructions for how to ask the user.
- **Narrate and visualize as you go.** After each meaningful change, say what you changed and why, and show the current graph — see "Show the user the graph" below.
- **Pause only for consequential choices.** Many Clay actions do nearly the same thing, and most steps can be built more than one way. Prefer the best-supported default when the choice is reversible and low-risk. Ask when the options have a meaningful difference in cost, coverage, credentials, destination, or workflow semantics that cannot be inferred from the request. Refer to options by their **human-readable names** (e.g. "Find Work Email (Clay)" vs "Waterfall Email Finder"), never internal `actionKey`s.

You build workflows out of two kinds of nodes:

- **Claygent (agent) nodes** — LLM loops with prompts. The default building block for reasoning, drafting, summarizing, and classifying.
- **Tool nodes** (`nodeType: "tool"`) — run a single Clay action directly (an enrichment, an HTTP call, a CRM write, etc.). Pick the action from the workspace's available action set. (The TC UI labels these "Enrich" or "Function"; both are `nodeType: "tool"`.)

You should also understand **triggers** — how a workflow gets launched (audience segments, schedules, webhooks, Clay tables). Triggers can be created and edited through the workflow MCP tools; Clay table triggers remain UI-only.

## Setup required

Anything that uses the `clay` CLI (running tests, searching actions, viewing snapshots, managing runs) requires the CLI on your PATH and a signed-in session (`clay login`; run the `setup` skill if `clay whoami` fails on auth). If a `clay` command returns `command not found`, do not conclude it's unavailable or fall back to other tools: retry once (a transient PATH-init race can briefly hide it), and if it's still missing, run the `setup` skill to install it. This only needs to be done once. The workspace is resolved from the stored session — there is no workspace id to set.

## Your capabilities

MCP tools:

- **Read workflows and nodes** (`read`)
- **Create, update, or delete nodes** (`edit_node`) — for `agent` and `tool` node types
- **Validate workflows** (`validate_workflow`)
- **Execute Clay actions one-off** (`execute_clay_action`)

CLI capabilities (via the `clay` CLI):

- Start, poll, and inspect workflow runs (see `testing.md`)
- Browse the Clay action catalog (`clay workflows actions`; use `/workflows-discover-actions`)
- Snapshots / version history (`clay workflows snapshots`; use `/workflows-snapshots`)
- Publish a tested draft as live (`clay workflows publish <workflowId>`)

See `publishing.md` for draft-versus-live behavior.

## Audiences — the user's own people, companies, and deals

Audiences is Clay's CRM-shaped store of the workspace's own **people**, **companies**, and **deals** (opportunities). Reading and segmenting it is a capability you have, separate from building workflows.

**Read `/audiences` before planning or replying whenever the request involves any of:**

- **A question about their records** — "how many people / companies / deals do I have", who has an email or phone, fill rates, looking someone up, or any count, list, or fact about the workspace's own contacts, leads, accounts, or customers.
- **Deals or pipeline** — closed-won, closed-lost, open pipeline, deal stage, deal size / ACV / ARR / bookings, close date, forecast, win rate, sales cycle, new business, renewal, expansion, churn, "our customers".
- **Saved audiences (segments)** — listing them, creating one, changing a filter, or archiving one.
- **Field definitions** — which fields exist on people, companies, or deals, their ids, or their data types.
- **An audience trigger** — you need the segment id from that skill to wire one up.
- **Writing values onto records** — you need field ids before using `upsert-audiences-record`.

## Draft vs live

- **Draft** = the editable graph you change with `edit_node`.
- **Publish** = shipping the current draft as the live version and activating its triggers — run `clay workflows publish <workflowId>`, or the user clicks **Publish** in the editor. Edits after that stay draft-only until the next publish.
- **Publishing is what takes triggers live** — a trigger you add or edit stays inert until the user publishes, and there is no per-trigger "set live" action, so never say you set a trigger live or that a new trigger is already live. Pausing or resuming an existing trigger is a separate per-trigger action and never publishes a new version.
- Edits after publish do **not** change live automation until the user publishes again.
- **Not every CLI test exercises the draft.** A plain / manual `clay workflows runs test` (no `--audience-segment`) runs the current draft. `clay workflows runs test --audience-segment …` goes through the audience segment trigger — after the workflow is published, that exercises the **live** version, not unpublished draft edits. See `publishing.md` and `testing.md`.
- `snapshots restore` rewrites the draft only — it is not a publish or unpublish. Full details in `publishing.md`.

## How a Clay workflow is structured

Clay workflows are graph-based. A workflow is a directed graph of nodes connected by edges. The two node types are:

### Agent nodes (Claygents)

An agent node is an LLM loop with a prompt and a model. When the run reaches the node, the LLM:

1. Reads the prompt; values from prior nodes must be passed through pinned `inputSchema` properties with `sourceNodeId` and `sourcePath`
2. Does whatever the prompt asks
3. Picks one of its outgoing edges and transitions to that next node

Agent nodes can have tools attached via the **Claygent configuration** (this is separate from the `tools` field that tool nodes use). Unfortunately, you cannot create Claygents with tools directly - but the user can do this in the UI themselves if they edit the Claygent directly. Treat agent nodes as Claygent prompts that do reasoning, summarization, drafting, classification, etc., and let tool nodes do the data lookup work.

### Tool nodes (`nodeType: "tool"`)

A tool node is a step intended to execute a single Clay action directly — no LLM reasoning. During the outline pass, you may create one without the `tools` field so the user can see the workflow's structure before every implementation choice is settled. An actionless tool node is an incomplete draft step: it cannot run, validate, or publish successfully. Configure it with exactly one tool in the second pass, with inputs filled from upstream.

Do not back a tool node with the Use AI, ChatGPT schema mapper, Claude, Gemini, or Claygent actions — these LLM actions are rejected on tool nodes. When the work needs an LLM (summarizing, drafting, classifying, extracting), use an agent (Claygent) node instead.

Choose the best-supported Clay action from the user's request and the workspace catalog. When several actions differ materially in cost, coverage, credentials, or semantics, recommend a default and ask the user to choose. Before wiring, fetch the action's declared shape with `clay workflows actions schema <packageId> <actionKey>` — its `inputParameters` are the fields to bind and `outputParameters` are the downstream-addressable output paths, both available before the node has ever run. Only fall back to `execute_clay_action` when the schema declares no outputs, or to confirm the action actually works on this workspace (credentials, live behavior) rather than to discover field names.

**`execute_clay_action` is a single-shot preview only** — capped at ~25 runs/day per user.
Use it once to confirm an action's input/output shape, never to enrich or process a batch
of records. To run an action over many inputs, wire it into a workflow node (what you're
already building here) or a table column. If the session also has `clay` CLI or Public API
access, a managed routine (`clay routines runs start`) works too. None of these go through
the capped preview path.

### Conditional nodes (`nodeType: "conditional"`)

A conditional node selects exactly **one** outgoing edge and follows it. Routing is configured via the top-level `conditionalConfig` field (a JSON **object**, never a stringified JSON string). Validation fails if a conditional node has no `conditionalConfig`.

**When to use which mode:**

| Situation                                                                                                                                                                                                                  | Use            |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------- |
| Branch on string/number/boolean field values — equality, comparison, contains, starts/ends with, empty/not-empty                                                                                                           | `rules` mode   |
| Multiple conditions combined with AND/OR                                                                                                                                                                                   | `rules` mode   |
| Branching decision requires open-ended reasoning (e.g. "classify this support ticket as billing, technical, or general", "does this email sound interested or not?")                                                       | `agentic` mode |
| You need to compute or transform a value to decide the route, and that transformation can't be expressed as a field comparison (e.g. parse a JSON blob and branch on a nested value, compute a score from multiple fields) | `code` mode    |

**`conditionalConfig` shapes** (pass the object itself to `edit_node`):

- **Rules (prefer this):** `{ "mode": "rules", "rulesConfig": { "rules": [...], "defaultTargetNodeId"?: "...", "endRunOnNoMatch"?: true } }`
- **Agentic:** `{ "mode": "agentic", "agentConfig": { "prompt"?: "...", "outputSchema"?: {...} } }`
- **Code:** `{ "mode": "code" }` — put the Python handler in the top-level `code` field (not inside `conditionalConfig`); wire branches with `incomingEdges` + `transitionId` on downstream nodes

**`rules` mode** — supported operators:

- **String**: `Equal`, `NotEqual`, `Contain`, `NotContain`, `ContainAny` (value is an array), `StartsWith`, `NotStartsWith`, `EndsWith`, `NotEndsWith`, `Empty`, `NotEmpty`
- **Number**: `Equal`, `NotEqual`, `GreaterThan`, `GreaterThanOrEqual`, `LessThan`, `LessThanOrEqual`, `Empty`, `NotEmpty`
- **Boolean**: `True`, `False`, `Empty`, `NotEmpty`

Each rule is a `ConditionalExpressionGroup` — a tree of `BinOp` leaf nodes (a single field comparison) and `GroupOp` nodes (AND/OR of children). Rules are evaluated top-to-bottom; first match wins. Set `defaultTargetNodeId` for a fallback when no rule matches. Declare every field a rule reads in top-level `inputSchema` (with `sourceNodeId` / `sourcePath`), then reference those field names from condition `dataPath` arrays.

Example condition (headcount ≤ 50 AND title contains "CTO"):

```json
{
  "type": "GroupOp",
  "combinationMode": "And",
  "items": [
    {
      "type": "BinOp",
      "dataPath": ["headcount"],
      "operator": "LessThanOrEqual",
      "value": 50
    },
    {
      "type": "BinOp",
      "dataPath": ["title"],
      "operator": "Contain",
      "value": "CTO"
    }
  ]
}
```

**`code` mode** — the Python handler can both compute values and route. Use when the routing decision requires transformation that rules can't express (e.g. parsing a nested structure, calling a helper, computing a derived value). Prefer `rules` mode when the branch is a simple field comparison.

Always call `context.transition_to("Destination Node Name", "branch_label")` with **both** arguments:

- **First arg** — exact destination node name (runtime resolves the next step by this name among connected nodes).
- **Second arg** — stable branch / port id (`codeTransitions` id). Use a short condition label (`even`, `odd`, `is_urgent`), not the destination node name when you can avoid it. Do not omit the second arg.

Wire each destination with `incomingEdges` that include `transitionId` matching that second arg:

```json
{ "sourceNode": "<conditionalNodeId>", "transitionId": "even" }
```

Never use a bare `{ "sourceNode": "..." }` edge or `isDefaultRoute: true` unless the edge is truly the Default fallback.

**Preferred build order for code conditionals:**

1. Create placeholder destination nodes whose names will appear as the first `transition_to` argument.
2. Create the code conditional with `{ "mode": "code" }` and the Python handler (two-arg `transition_to` calls).
3. Attach each destination via `incomingEdges` with the matching `transitionId` (second arg). Re-saving the conditional's code after destinations exist also lets analysis auto-create edges with the correct handles.

### Trigger nodes and leaf nodes

- Workflows start empty. Create a **trigger node** plus at least one trigger before the workflow can run end-to-end. A **manual** trigger is the usual entry point for test/`clay` runs. Additional launch paths get their **own** trigger nodes — do not stack webhook/audience/scheduled onto the manual node.
- Creating a trigger via MCP (`surfaces_edit` / trigger surface) returns `workflowNodeId`. Wire the first action nodes with `incomingEdges` from that id.
- **Audience multi-segment sharing:** multiple `audience_segment` triggers (different `segmentId`s) may share one trigger node when they have the **same trigger type** and the **same outgoing edge**. Multiple `audience_scheduled` triggers may share a node when they also have the **same schedule**. Pass an explicit `workflowNodeId` to bind/share; omit it (or pass `createTriggerNode: true`) to get a new node. Do not mix `audience_segment` with `audience_scheduled` on one node. `audience_manual` is a run companion created by the UI/run path — do not create it via the surface.
- **Trigger edge constraint:** a trigger may have zero or one direct outgoing edge, never more. Before adding an edge from a trigger, check the workflow summary's `edges` for any edge whose `sourceNodeId` is the trigger's node id — `summary.edges` already covers trigger→node edges, so a plain `read` (no `nodeId`, default `mode`) is enough to check this. If it already has a target, do not add another direct edge; add work downstream instead, or ask the user whether to rewire the workflow. Before validating or running, each trigger must be connected to one first executable node.
- **Leaf nodes** are nodes with no downstream connections. They are automatically treated as terminal — you do not need to mark them.

## Triggers — how workflows get launched

A workflow doesn't run by itself. It runs because a **trigger** kicks off a run. Use the workflow MCP tools to create or edit triggers when requested; configure Clay table triggers in the Clay tables UI:

- **Audience segment trigger** — every record in a Clay audience segment becomes a run input. Useful for batch-style enrichment over a known list.
- **Scheduled trigger** — fires one contextless workflow run per schedule tick. Provide `scheduleConfig` with either a simple or custom recurrence.
- **Audience scheduled trigger** — reruns all current members of an audience segment on each schedule tick. Provide `segmentId`, `entityType`, and `scheduleConfig`. May share a trigger node with other `audience_scheduled` triggers that use the same schedule and outgoing edge (pass their `workflowNodeId`).
- **Webhook trigger** — an external system POSTs to a URL and each request becomes a run (own trigger node).
- **Clay table trigger** — new rows added to a specific Clay table create runs automatically.
- **One-off / batch test runs** — the user (or the `clay` CLI) launches a single run or a batch for testing via a manual trigger (create one if the workflow does not have one yet).

When designing a workflow, determine how it will be triggered from the request and current workflow when possible. If it is not specified, recommend the simplest trigger that fits and proceed when that choice is easy to revise; ask only when the trigger changes the intended behavior or required inputs materially. The trigger determines:

- What the trigger node's outputs look like (a row from a table? a webhook body? an audience record?)
- Whether the workflow should be optimized for one-at-a-time or high-volume runs
- Whether leaf node output goes back to a Clay table, a webhook response, etc.

If the user hasn't picked a trigger, recommend the simplest option that fits their use case and create it via the trigger surface.

## Required fields for new nodes

For every node:

- `name`, `nodeType`, `incomingEdges`

For agent nodes (`nodeType: "agent"`):

- `agentName`, `agentPrompt`, `agentModel`
- Always send `agentName`, `agentPrompt`, and `agentModel` together in a single `edit_node` call. Sending them separately can result in an agent with a blank prompt.
- **Model selection — use a two-phase approach:**
  1. **While building and testing:** use `gpt-5.4-nano` for `agentModel`. It's the fastest, which keeps the debug loop tight.
  2. **After the workflow works e2e:** graduate to whatever model is the best fit for the task.

For tool nodes (`nodeType: "tool"`):

- `tools` may be omitted while drawing the initial outline. Before validation, testing, or publishing, add exactly one entry. The `actionKey` is the Clay action you want to invoke (confirm it via `clay workflows actions schema` first, falling back to `execute_clay_action` only when the schema doesn't cover what you need).

For conditional nodes (`nodeType: "conditional"`):

- `conditionalConfig` — required object selecting the routing mode (`rules`, `agentic`, or `code`). Never send it as a stringified JSON string.
- **Rules:** also set `inputSchema` for every field the rules read (with `sourceNodeId` / `sourcePath`)
- **Code:** also set top-level `code` (the Python handler with two-arg `transition_to`); `conditionalConfig` is just `{ "mode": "code" }`; wire each branch with `incomingEdges` + `transitionId` on the downstream nodes
- **Agentic:** set `agentConfig.prompt` when helpful

## Adding a tool to a tool node

Use the `tools` field with a single-element array:

```json
[
  {
    "toolType": "clay_action",
    "actionKey": "<actionKey>",
    "actionPackageId": "<packageId>"
  }
]
```

Or identify the action by an existing tool's id:

```json
[{ "toolType": "clay_action", "toolId": "tct_abc123" }]
```

Either way the node ends up with its **own** tool instance — `edit_node` keeps a tool only when the node already has it, and otherwise creates a new one carrying the same action and credentials. Never try to point two nodes at one tool.

The user can tell you which `actionKey` and `actionPackageId` to use, or which existing `toolId` to model the node on. Fetch `clay workflows actions schema` for its input/output shape before adding it; test with `execute_clay_action` when you need to confirm the action actually works on this workspace (credentials, live behavior) rather than to learn field names.

Wire the action's parameters with `inputMappingConfig` on the tool entry — each parameter maps to a `static` value or a `reference` expression. Nested/grouped parameters use `parent|sub` pipe keys:

```json
[
  {
    "toolType": "clay_action",
    "actionKey": "hubspot-lookup-object",
    "actionPackageId": "a2584689-...",
    "inputMappingConfig": {
      "objectTypeId": { "type": "static", "value": "0-2" },
      "fields|domain": { "type": "reference", "expression": "{{domain}}" },
      "fields|fieldsToFilterBy": { "type": "static", "value": ["domain"] }
    }
  }
]
```

**`inputMappingConfig` is stored on the tool, not the node, and applies to every node bound to that tool.** That's why `edit_node` gives each node its own tool instance instead of binding it to an existing one — mappings you set for one node can never leak into another node or workflow. Legacy workflows built before this may still share a tool; if a mapping change shows up on another node, re-add the action to that node so it gets a fresh instance.

For actions whose fields depend on an earlier input (e.g. an object type that reveals a different field set), resolve the real `objectTypeId` values and `fields|<sub>` keys with `clay workflows actions dynamic-fields` before mapping — don't guess them.

See `data-passing.md` for `inputMappingConfig` types (`static` / `reference` / `skip`), the `parent|sub` pipe convention, resolving dynamic fields, and the dropped-`inputSchema`-variables gotcha.

## Enabling batching on a tool node

Some actions support batching multiple workflow runs into a single provider call, dramatically cutting cost/rate-limit pressure for high-volume workflows. Not every action supports it, and the action catalog doesn't flag which ones do — so treat batching as something you enable on request and let `edit_node` confirm support.

Set `batchRunSettings: { "enabled": true, "maxBatchSize": <n> }` on a tool node via `edit_node` to turn it on — `maxBatchSize` is optional and gets clamped to the action's real maximum. **Only set this when the user explicitly raises batching, rate limits, or handling large volumes of rows/runs — never proactively.** If the action doesn't support batching, `edit_node` rejects the request with an error — relay that to the user rather than retrying.

`batchRunSettings` can only be set on a tool node that already has its `tools` field configured — if you're creating the node and enabling batching in the same conversation turn, do it as two separate `edit_node` calls (create with `tools` first, then enable batching in a follow-up call).

## List mode (Repeat)

An agent, code, or tool node can run once per item in an upstream list instead of once over the whole list. In the editor this is the **Repeat** toggle. Describe it to the user as **"runs once per item"** — never say it "maps over the list" or call it a map/loop.

Set `listMode: true` on the node via `edit_node`, and set `listEntriesRef` to the upstream array it repeats over (`{ "sourceNodeId": "wfn_upstream", "path": "$.results" }`) in the same call. Inside a repeating node, its inputs and prompt resolve against the current item, not the whole list.

Always set `listEntriesRef` when turning list mode on: without it the node looks for an `entries` input instead of the upstream array, and repeats over the wrong thing. Pass `null` to clear it.

Map each tool input that should vary per item to the **current item**, not to the upstream node. Use an `item` mapping in `inputMappingConfig`: `{ "type": "item", "path": "$.field" }` (`$` is the whole item, `$.url` / `$.full_name` a field on it). Example: repeating over a list of people, wire the tool's `full_name` parameter to `{ "type": "item", "path": "$.full_name" }`, not to a `{{...}}` reference to the upstream list node — a reference resolves to the whole aggregate, so every item would get the same value. Reserve `static`/`reference` for inputs that are genuinely the same across all items. See `data-passing.md`.

`listFailureMode` controls what happens when individual items fail:

- `"fail_at_end"` (default) — run every item, then fail the node if any item failed.
- `"ignore_errors"` — keep the successful items and continue.

**When to use it:** the user has a list (search results, an audience, a prior node's rows) and wants the same per-record work — enrich each row, draft one message per lead, classify each item — done once for every item.

**When not to use it:** a single aggregate step that should see the whole list at once (summarize all results, pick the top N, dedupe across the list). Leave `listMode` off for those — the node gets the full array. It is also unsupported on account agent nodes and on every other node type (trigger, conditional, map, reduce, collect, fork, join).

`listMode` and `listFailureMode` are only available when the workspace has list mode enabled; if `edit_node` rejects them, relay that the feature isn't enabled rather than retrying.

## Passing data between nodes

### Pinned inputs on agent nodes (typed, deterministic)

For data that needs to be exact (numbers, structured fields, data from 2+ hops back), declare an `outputSchema` on the upstream node, then on the downstream **agent** node pin each input by putting `sourceNodeId` + `sourcePath` **inline on the `inputSchema` property**:

```json
{
  "inputSchema": {
    "type": "object",
    "properties": {
      "company_name": {
        "type": "string",
        "sourceNodeId": "wfn_upstream",
        "sourcePath": "$.company_name"
      },
      "score": {
        "type": "number",
        "sourceNodeId": "wfn_upstream",
        "sourcePath": "$.score"
      }
    }
  }
}
```

The reference is `sourceNodeId` + `sourcePath` inline on the property. Use `sourcePath`, not `path`. Path syntax is JSONPath (`$.field`, `$.nested.field`). Agent nodes access pinned inputs as `{{company_name}}` in the prompt.

**Tool nodes are different** — their action parameters are wired in `tools[].inputMappingConfig` (`static` / `reference`), not in `inputSchema`. Do not add intermediate variables to a tool node's `inputSchema`; non-action-parameter properties are dropped on save. See `data-passing.md` for the full reference.

**Important — enrich (tool) node output paths:** An enrich (tool) node's Clay action fields are at
`$.result.<field>`, and its success flag is at `$.success`. Never guess the field names — take
them from `outputParameters` on `clay workflows actions schema` (declared output fields,
available before the node runs), the node's `recentOutputPaths` field (visible via `read` with
`nodeId`, or `mode: "full"` — not the workflow-level summary), or an `execute_clay_action` run —
then prefix them with `$.result.`. For example: `$.result.name`, `$.result.domain`.

## Recommended workflow for building

0. **Build the outline first, then implement it iteratively.** Infer the trigger when the request or current workflow makes it clear; otherwise recommend the simplest fit. Give the user a short outline of how the workflow starts, its main steps, and the outcome, then create that structure in the draft without waiting for every action choice or the entire design to be approved. Treat the outline as provisional: surface important assumptions, incorporate what you learn from action schemas and test results, and revise as you go.

   Format the outline as real Markdown so it scans as a hierarchy, not a wall of text: use `##`/`###` headings for sections, ordered lists (`1.`) for sequential steps, bulleted lists (`-`) for options and inputs, and a `>` blockquote for callouts. Do not fake lists with bold-prefixed lines separated by line breaks — those render flat, with no bullets or spacing.

   For a non-technical request, prefer this shape when an outline adds clarity:

   ```markdown
   ## How it would work

   1. Start when [user-visible event] happens.
   2. [Main user-visible step].
   3. [Main user-visible step].
   4. [User-visible outcome].

   I'm starting with these assumptions:

   - [Reasonable, reversible assumption]
   - [Blocking decision, only if it prevents the next safe step]
   ```

   Keep implementation details out of this outline unless the user asks for them or they are required to resolve a blocking decision.

1. **If you create a new workflow, post a clickable Markdown link right away.** As soon as the workflow exists, put `[Open workflow](<url>)` near the top of your reply, using the `url` from `clay workflows create` (or `clay workflows get`) — not raw JSON and not "see the `url` field." This matters most in a headless environment (Claude Code, Cursor, a shell), where the user has no Clay tab open; if you're the in-product assistant and they're already viewing the workflow, skip the link. See `presenting.md` ("Workflow editor link").
2. Confirm the trigger so you understand the initial node's inputs
3. Build the outline node-by-node with `edit_node`, wiring `incomingEdges` as you go. Give each step a specific, outcome-oriented name. For a step that clearly needs a Clay action but whose provider or exact action is not settled, create a tool node without `tools`. Choose the node type before creating it because an existing node's type cannot be changed. After the outline is on the canvas, show the graph and tell the user which steps still need implementation.
4. Resolve the incomplete steps one at a time. For each actionless tool node, use `/workflows-discover-actions` to find candidates, fetch the chosen one's fields with `clay workflows actions schema` (falling back to `execute_clay_action` only if it declares no outputs or you need to confirm it actually works on this workspace), then attach it to the existing node and configure its inputs. When several actions fit, choose and name a recommended default if the choice is reversible and low-risk; ask the user only when cost, coverage, credentials, or semantics create a consequential trade-off.
5. Once every tool node has exactly one action or function, run `validate_workflow` with `prettier=true` to auto-layout and catch issues, then **show the user the resulting graph** (see "Show the user the graph" below). `tool_node_no_tool` is expected while the outline is incomplete; resolve it before testing or publishing.
6. Suggest the user kicks off a test run. When you narrate the run afterward, show a **status-annotated view** of the graph — mark each node completed / failed / running — so the user sees where in the flow each result came from (see `testing.md` and `presenting.md`)
7. After the draft passes an end-to-end test and the user wants it live, run `clay workflows publish <workflowId>`. This publishes the current draft; later edits remain draft-only until the next publish. Use `--name` only to label the published version, not to rename the workflow.

## Show the user the graph

Users can't follow what you're building unless you show them, so narrate each
change in plain language and redraw the graph for changes that visually change
it (nodes or edges added, removed, or rewired). **`presenting.md` is the single
source of truth for how** — which diagram command to use in each environment,
when to redraw versus just narrate, when to summarize instead of dumping raw
output, and how to annotate the graph with per-node run status. Read it before
your first render.

## Best practices

1. Always read the workflow first to understand current state before editing, then give the user a concise graph render or plain-language recap as you begin
2. Share a provisional direction, make reasonable reversible assumptions, and draw the workflow before resolving every action; do not require sign-off on the entire plan before creating nodes
3. Create nodes sequentially with `edit_node`, using `incomingEdges` to wire them to existing nodes. Leave tool nodes actionless when the intended step is clear but its exact action is not, then configure those same nodes in a second pass
4. Treat actionless tool nodes as incomplete draft steps; do not test or publish the workflow while any remain
5. After configuring all steps, run `validate_workflow` with `prettier=true` and show the user the updated graph
6. Use string-replace mode for small edits to prompts
7. When adding enrichment tools, try 2-3 alternative actions as fallbacks if the primary one might miss; prefer the best-supported default and ask the user only when the alternatives have a consequential trade-off
8. After completing a workflow, suggest a test run and walk the user through what the run did (see `testing.md`)
9. If you make a mistake or the user asks to undo, use `/workflows-snapshots` to revert (restore changes the draft only — not live automation)
10. When the user wants live automation to match the draft, publish it with `clay workflows publish <workflowId>` (or tell them to click **Publish** in the editor). There is no MCP publish tool. Restore/undo does not publish (see `publishing.md`)

## Reference docs in this skill

- `presenting.md` — How to narrate and visualize your work (diagrams, tables, run-status annotation) so the user can follow along
- `data-passing.md` — How pinned inputs and `inputMappingConfig` work in detail
- `testing.md` — `clay` CLI commands for running and inspecting workflow runs
- `publishing.md` — Draft vs live, when to ask the user to Publish, restore ≠ publish
- `audiences.md` — Audiences inside a workflow: the `upsert-audiences-record` action for writing records, and the `audience_segment` trigger handoff. Everything else about audiences — reading records, counts, deals, segments, field definitions — is the `audiences` skill; see the Audiences section above for when to read it.
- `clay <command> --help` — Per-command JSON shape, flags, and error codes
