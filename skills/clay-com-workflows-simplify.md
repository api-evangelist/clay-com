---
name: workflows-simplify
description: 'Clay workflows — simplify a workflow: merge redundant nodes, cut unnecessary complexity, and replace LLM nodes with deterministic alternatives where possible.'
---

# Simplifying a workflow

Analyze the current workflow and suggest concrete simplifications to reduce complexity, improve reliability, and lower costs.

## Process

1. **Read the workflow** using `read` (full mode) to get all node details
2. **Analyze each node** against the simplification checklist below
3. **Present findings** as a prioritized list of suggestions with specific changes, alongside a render of the **current graph** (`clay workflows diagram <workflowId>`) so the user can see which nodes each suggestion affects
4. **Apply authorized changes** — edit the workflow only when the user's request authorizes modifications. Once authorized, apply clearly behavior-preserving improvements as you go and ask before changes with a material behavior or quality trade-off
5. **Show the result** — after applying, run `validate_workflow` with `prettier=true` and render the **updated graph** so the simplification is visible, not just described

Narrate throughout and prefer the diagram over raw node JSON — see `workflows/presenting.md`.

## Simplification Checklist

### Replace LLM nodes with code nodes

Regular (LLM) nodes cost an LLM call per execution. Many can be replaced with deterministic code:

- **Data transformation** — extracting fields, reformatting JSON, string manipulation → code node
- **Simple routing** — if the decision can be expressed as rules on data fields → conditional node (rules mode)
- **Calculations** — math, aggregation, counting → code node
- **Template filling** — constructing strings from known fields → code node

**Ask:** "Does this node require creative reasoning, or could a Python function do the same thing?"

### Merge sequential nodes

Two adjacent nodes can often be combined into one if:

- Node A passes all its output to Node B, and Node B doesn't add new tools or branching
- Both nodes use the same model and could be described in a single prompt
- One node just reformats the other's output

**Ask:** "Would combining these prompts into one still produce the same result?"

### Remove unnecessary nodes

- Nodes that just pass data through without transformation
- Conditional nodes with only one possible outcome

### Pin typed inputs deterministically

When a downstream node needs specific typed data from an upstream node:

- Add `outputSchema` to the upstream node
- On the downstream agent node, pin each input via `sourceNodeId`/`sourcePath` inline on the `inputSchema` property (see `workflows/data-passing.md`)
- This preserves exact values across nodes

### Simplify tool usage

- If a node has tools it never uses, remove them (reduces prompt size and cost)
- If a node calls one tool and passes the result, consider making it a code node with `context.call_tool()`

### Use code mode for conditional and map nodes

- Conditional nodes: prefer `rules` or `code` mode over `agentic` mode when the decision logic is expressible programmatically
- Map nodes: prefer `code` mode over `agent` mode when processing is deterministic

## Output Format

Present suggestions as:

1. **What to change** — specific node(s) affected
2. **Why** — what complexity or cost this removes
3. **How** — the concrete edit (new node type, merged prompt, code snippet)

Pair the suggestion list with the current-graph render so each affected node is easy to locate. For analysis or recommendation requests, present the suggestions without editing. If the user asks you to modify the workflow, apply clearly behavior-preserving improvements as you identify them and state your assumptions. Ask only when an edit has a meaningful behavior or quality trade-off. Then run `validate_workflow` with `prettier=true` and show the updated graph so the user can see the before/after difference.
