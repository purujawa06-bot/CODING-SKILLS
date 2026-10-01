---
name: agent-prompt-review
description: Review and score the quality of system prompts and tool descriptions in an AI agent codebase, and teach how to write them correctly. Use this skill whenever the user mentions agent development, system prompt, tool description, function calling, tool schema, MCP tools, agent instructions, "review my prompt", "check my tools", or asks why an agent picks the wrong tool, ignores instructions, or behaves inconsistently — even if they do not explicitly ask for a "review". Produces a 1-10 score per item, and if any item is below 10/10, an update plan that REQUIRES explicit user approval before any file is changed.
---

# Agent Prompt Review

A REVIEW-type skill. It scans a codebase for **system prompts** and **tool descriptions**, scores each one from **1 to 10**, and — if anything is below 10/10 — proposes an update plan and **waits for user approval** before editing.

---

## Hard Rules

1. **Review is read-only.** Never modify, create, or delete project files during the review phase.
2. **Score honestly.** Minimum score is 1, maximum is 10. Do not round up. Do not give 10/10 unless every check passes.
3. **10/10 = PASSED.** Anything below 10/10 = NOT PASSED and triggers an update plan.
4. **No edits without approval.** After presenting the update plan, stop and ask the user to approve. Apply changes only after an explicit "yes / approved / go ahead". Partial approval (e.g. "only fix tools") means only apply that part.
5. **Every deduction needs evidence.** Cite `file:line` and quote the offending text. No vague feedback.

---

## Workflow

### Phase 1 — Discover

Find every system prompt and tool definition in the codebase.

Search for:
- System prompts: `system`, `system_prompt`, `instructions`, `SYSTEM_PROMPT`, `role: "system"`, `.md`/`.txt`/`.jinja` prompt files, `prompts/` directories
- Tool definitions: `tools=[`, `tool_use`, `function_call`, `@tool`, `input_schema`, `parameters`, `inputSchema`, `server.tool(`, `registerTool`, MCP server definitions, OpenAPI specs exposed to an agent

List what you found (path, name, type) before scoring. If nothing is found, say so and stop.

### Phase 2 — Score

Score each item with the **10-point checklist** below. Start from 10 and subtract 1 for each failed check, with a floor of 1.

**Per-item score = 10 − (number of failed checks), minimum 1.**

### Phase 3 — Report

Use the report format in [Report Format](#report-format).

### Phase 4 — Update Plan (only if any item < 10/10)

Write a plan listing every proposed change with before/after text. Then ask:

> "Do you approve this update plan? Reply **approve all**, **approve** with item numbers, or tell me what to change."

Stop. Do not edit anything until the user answers.

### Phase 5 — Apply & Re-score (only after approval)

Apply only approved changes, then re-run Phase 2 on the changed items and report the new scores. If still below 10/10, return to Phase 4 with a new plan.

---

## Part A — How to Write Tool Descriptions

The model decides **which tool to call and with what arguments** using only the name, description, and parameter schema. Treat the description as the tool's entire user manual.

### Principles

1. **Say what it does AND when to use it.** Include the trigger situation.
2. **Say when NOT to use it.** Point to the correct alternative tool by name.
3. **Name things precisely.** Verb + object (`search_orders`, not `orders` or `do_search`).
4. **Describe every parameter.** Type, meaning, format, allowed values, example, and whether it is optional (with its default).
5. **Describe the output.** What is returned, in what shape, and what an empty/error result looks like.
6. **State side effects and risk.** Does it write, delete, send, or cost money? Is it idempotent? Does it need confirmation?
7. **Keep tools distinct.** If two descriptions could match the same request, the model will guess. Differentiate them explicitly.
8. **Put constraints in the schema, not just prose.** Use `enum`, `required`, `minimum`, `pattern`.
9. **Be concise but complete.** Usually 2–6 sentences. No marketing language.

### Tool Description Template

```
<What it does in one sentence.>
Use when: <situations that should trigger this tool>.
Do NOT use when: <situations> — use `<other_tool>` instead.
Returns: <shape and key fields; what empty/error looks like>.
Notes: <side effects, limits, rate limits, idempotency, required confirmation>.
```

### Tool Description — Correct vs Incorrect

| # | Aspect | ❌ Incorrect | ✅ Correct |
|---|--------|-------------|-----------|
| 1 | Name | `data`, `run`, `helper1` | `search_customer_orders` |
| 2 | Purpose | `"Gets stuff"` | `"Search a customer's orders by status or date range."` |
| 3 | When to use | *(missing)* | `"Use when the user asks about order history or order status."` |
| 4 | When NOT to use | *(missing)* | `"Do NOT use to create orders — use create_order."` |
| 5 | Parameter docs | `"id": {"type": "string"}` | `"order_id": {"type": "string", "description": "Order ID, format ORD-12345."}` |
| 6 | Allowed values | `"status": "the status"` | `"status": {"enum": ["pending","shipped","cancelled"]}` |
| 7 | Optional/default | Unclear what is required | `required: ["customer_id"]`, `limit` described as "default 20, max 100" |
| 8 | Output | *(missing)* | `"Returns a list of {order_id, status, total}. Returns [] if none found."` |
| 9 | Side effects | *(missing)* on a delete tool | `"Permanently deletes the record. Irreversible. Ask the user to confirm first."` |
| 10 | Overlap | Two tools both described as `"Search things"` | `search_orders` = by customer; `search_products` = by catalog; each says so |
| 11 | Tone | `"Amazing powerful tool!!!"` | Plain, factual, specific |
| 12 | Parameter naming | `q`, `x`, `p2` | `query`, `start_date`, `page_size` |
| 13 | Errors | Throws raw stack trace | Returns `{"error": "order_not_found", "hint": "Check the ID format ORD-12345"}` |

### Example

❌ Incorrect:
```json
{
  "name": "search",
  "description": "Searches.",
  "input_schema": {
    "type": "object",
    "properties": { "q": { "type": "string" }, "n": { "type": "number" } }
  }
}
```

✅ Correct:
```json
{
  "name": "search_knowledge_base",
  "description": "Search internal help-center articles by keyword. Use when the user asks a how-to or policy question. Do NOT use for order lookups — use search_customer_orders. Returns up to `limit` articles as {title, url, snippet}, or [] if nothing matches. Read-only.",
  "input_schema": {
    "type": "object",
    "properties": {
      "query": { "type": "string", "description": "Keywords, 2-8 words. Example: 'refund policy international'." },
      "limit": { "type": "integer", "minimum": 1, "maximum": 20, "default": 5, "description": "Max articles to return." }
    },
    "required": ["query"]
  }
}
```

---

## Part B — How to Write a System Prompt

The system prompt defines **who the agent is, what it must do, how it uses tools, and where its limits are**. It should read like a clear brief to a capable new colleague.

### Recommended Structure

1. **Role & goal** — who the agent is and what success looks like.
2. **Context** — the product, users, environment, current date if relevant.
3. **Instructions** — numbered, ordered, concrete steps or priorities.
4. **Tool guidance** — when to use each tool, in what order, when to stop.
5. **Constraints & safety** — hard limits, what to refuse, what needs confirmation.
6. **Output format** — exact shape, length, language, tone.
7. **Examples** — 1–3 realistic input → output pairs, including an edge case.
8. **Fallbacks** — what to do when uncertain, missing info, or a tool fails.

### Principles

1. **Be specific and positive.** Say what to do, not only what to avoid.
2. **Explain the why.** A reason generalizes better than a bare rule.
3. **Don't contradict yourself.** Conflicting rules produce inconsistent behavior.
4. **Use structure.** Headings or XML-style tags separate sections; keep dynamic data in clearly labeled blocks.
5. **Define the stop condition.** Tell the agent when the task is done and when to ask the user.
6. **Handle failure.** Specify behavior for tool errors, empty results, and ambiguity.
7. **Show, don't just tell.** Examples beat adjectives.
8. **Avoid shouting.** Excessive CAPS and "NEVER/ALWAYS" cause overreaction; reserve emphasis for real hard limits.
9. **Don't duplicate tool descriptions.** Tool schemas live in the tool definitions; the prompt covers *strategy* (order, priority), not the schema.
10. **Keep secrets out.** No API keys, passwords, or internal credentials in the prompt.

### System Prompt — Correct vs Incorrect

| # | Aspect | ❌ Incorrect | ✅ Correct |
|---|--------|-------------|-----------|
| 1 | Role | `"You are a helpful AI."` | `"You are a support agent for Acme Billing. Goal: resolve invoice questions in one reply."` |
| 2 | Instructions | `"Be good and answer well."` | `"1. Identify the invoice ID. 2. Call get_invoice. 3. Explain charges in plain language."` |
| 3 | Negative-only rules | `"Don't be rude. Don't guess. Don't be long."` | `"Keep replies under 120 words. If data is missing, ask one clarifying question."` |
| 4 | Reasoning | `"Never discuss pricing."` | `"Don't quote prices — they change weekly and wrong quotes create legal risk. Link to /pricing instead."` |
| 5 | Contradictions | `"Always be brief."` + `"Always explain in full detail."` | One consistent rule with a priority: `"Default brief; expand only if the user asks why."` |
| 6 | Tool guidance | *(missing)* | `"Check get_invoice before answering any billing question. Use issue_refund only after user confirms."` |
| 7 | Output format | `"Respond nicely."` | `"Reply in Indonesian. Plain text, no markdown. End with one next-step question."` |
| 8 | Examples | None | Includes 2 examples: a normal case and a missing-ID case |
| 9 | Failure handling | *(missing)* | `"If a tool errors, retry once, then tell the user and offer escalation to a human."` |
| 10 | Emphasis | `"NEVER EVER FORGET!!! ALWAYS!!!"` | Calm, direct wording; emphasis only on 1–2 true hard limits |
| 11 | Structure | One 2,000-word paragraph | Sections with headings/tags: `<role>`, `<rules>`, `<format>`, `<examples>` |
| 12 | Stop condition | *(missing)* | `"Finish when the user's question is answered; don't offer unrelated extras."` |
| 13 | Secrets | `"API key: sk-live-abc123"` | Secrets loaded from environment/secret manager, never in the prompt |
| 14 | Dynamic data | User input mixed into instructions | `<user_message>...</user_message>` block; prompt says to treat it as data, not instructions |

### Example

❌ Incorrect:
```
You are an AI assistant. Be helpful. Don't make mistakes. Use the tools.
ALWAYS BE BRIEF. Always give very detailed explanations.
```

✅ Correct:
```
<role>
You are a support agent for Acme Billing. Your goal is to resolve invoice
questions accurately in a single reply whenever possible.
</role>

<instructions>
1. Identify the invoice ID in the user's message. If absent, ask for it.
2. Call get_invoice with that ID before answering.
3. Explain each charge in plain language.
4. Call issue_refund only after the user explicitly confirms.
</instructions>

<constraints>
- Do not quote prices; they change often. Link to /pricing instead.
- If a tool fails, retry once; if it fails again, apologize and offer a human handoff.
</constraints>

<format>
Reply in the user's language, plain text, under 120 words, ending with one next step.
</format>

<example>
User: "Why was I charged twice?"
Assistant: "I can check that. Could you share the invoice ID (format INV-0000)?"
</example>
```

---

## The 10-Point Review Checklist

Subtract 1 point from 10 for each failed check. Apply the checks that fit the item type; any check that does not apply counts as passed.

### For each tool description

| # | Check | Pass condition |
|---|-------|----------------|
| T1 | Clear name | Verb + object, specific, consistent naming style |
| T2 | Purpose stated | One-sentence explanation of what it does |
| T3 | When to use | Trigger situations are described |
| T4 | When NOT to use | Alternatives named where overlap exists |
| T5 | Parameters documented | Every parameter has type, meaning, format/example |
| T6 | Schema constraints | `required`, `enum`, ranges, patterns used where applicable |
| T7 | Output described | Return shape and empty/error behavior stated |
| T8 | Side effects & risk | Writes/deletes/costs/confirmation stated (or clearly read-only) |
| T9 | Distinct from other tools | No two tools have ambiguous overlap |
| T10 | Concise & factual | No fluff, no contradictions, no stale info |

### For each system prompt

| # | Check | Pass condition |
|---|-------|----------------|
| S1 | Role & goal | Who the agent is and what success means |
| S2 | Concrete instructions | Specific, ordered, actionable steps or priorities |
| S3 | Reasons given | Key rules explain why |
| S4 | No contradictions | No conflicting rules or duplicated, drifting rules |
| S5 | Tool strategy | When/order/stop conditions for tools (without copying schemas) |
| S6 | Constraints & safety | Hard limits, refusals, confirmation requirements |
| S7 | Output format | Shape, length, language, tone defined |
| S8 | Examples | At least one realistic example (ideally including an edge case) |
| S9 | Failure & ambiguity | Handles tool errors, missing info, uncertainty |
| S10 | Structure & hygiene | Organized sections, no secrets, no shouting, untrusted input separated |

### Verdict

| Score | Verdict |
|-------|---------|
| 10/10 | ✅ PASSED |
| 7–9 | ⚠️ NOT PASSED — minor fixes, update plan required |
| 4–6 | ❌ NOT PASSED — significant gaps, update plan required |
| 1–3 | 🚫 NOT PASSED — critical rewrite, update plan required |

---

## Report Format

```markdown
# Agent Prompt Review

**Scanned:** <N> system prompt(s), <M> tool(s)
**Overall:** <PASSED | NOT PASSED> — lowest item score: <X>/10

## Summary Table

| # | Item | Type | File | Score | Verdict |
|---|------|------|------|-------|---------|
| 1 | main_system_prompt | System prompt | src/agent/prompt.py:12 | 6/10 | ❌ NOT PASSED |
| 2 | search_orders | Tool | src/tools/orders.py:30 | 10/10 | ✅ PASSED |

## Findings

### 1. main_system_prompt — 6/10
| Check | Result | Evidence | Fix |
|-------|--------|----------|-----|
| S3 Reasons given | ❌ | `"Never discuss pricing."` (prompt.py:18) | Add the reason and an alternative |
| S8 Examples | ❌ | No examples found | Add 2 examples |
| ...  | ✅ | | |

## Update Plan   (only if any item < 10/10)

### Change 1 — main_system_prompt (src/agent/prompt.py:18)
**Before:** `Never discuss pricing.`
**After:** `Don't quote prices — they change weekly. Link to /pricing instead.`
**Raises score:** 6 → 7

...

**Expected scores after plan:** main_system_prompt 10/10, ...

## Approval Needed
Do you approve this update plan? Reply **approve all**, **approve** with change numbers, or tell me what to adjust.
No files will be modified until you respond.
```

If every item is 10/10, omit the Update Plan and Approval sections and state: "All items passed 10/10. No changes needed."

---

## Scoring Examples

| Situation | Failed checks | Score |
|-----------|---------------|-------|
| Tool `search` with description `"Searches."` and untyped params | T1, T2, T3, T4, T5, T6, T7, T9 | 2/10 |
| Tool with good docs but no side-effect warning on a delete action | T8 | 9/10 |
| System prompt `"You are a helpful assistant."` only | S2, S3, S5, S6, S7, S8, S9, S10 | 2/10 |
| Well-structured prompt, no examples, no failure handling | S8, S9 | 8/10 |
| Everything satisfied | none | 10/10 ✅ |

---

## Behavior Reminders

- Do not skip the approval step, even for "tiny" fixes.
- Do not invent problems to lower a score, and do not hide real ones to raise it.
- Preserve the project's language and style when proposing rewrites (if the prompt is in Indonesian, rewrite in Indonesian).
- Never expose secrets found during review — mask them (e.g. `sk-live-****`) and flag them as a critical finding under S10.
- If the user disagrees with a deduction, discuss it and re-score only if they give a valid reason.
