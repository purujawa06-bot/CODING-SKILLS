---
name: agent-development-tips
description: Guide to writing correct tool descriptions and system prompts when building AI agents (function calling / tool use). Use this skill whenever the user is building, reviewing, or fixing an agent, defining tools or function schemas, writing or tidying a system prompt, or debugging an agent that picks the wrong tool, fills parameters incorrectly, hallucinates, or ignores instructions, even if the user never says "skill" or "best practices".
---

# Agent Development Tips: Tool Descriptions & System Prompts

An agent is only as good as the information it receives. The model never sees your tool's code. It only reads the **name, description, and parameter schema**. And it only follows behavior that is **clearly written in the system prompt**. Most "the agent is dumb" bugs are actually documentation bugs.

## Workflow

1. List your tools and give each one a single, clear purpose.
2. Write the tool descriptions (Part 1).
3. Write the system prompt (Part 2).
4. Test with 5-10 realistic prompts, including ambiguous ones and out-of-scope ones.
5. Treat failures as clues: which tool was chosen wrongly? Which parameter was filled wrongly? Fix the **text** first before adding code.

---

## Part 1: How to Write Tool Descriptions

### Core principle

Imagine explaining this tool to a smart new colleague who has zero context. The description should answer:

1. **What** does the tool do?
2. **When** should it be used, and when should it **not** be used?
3. **What does each parameter mean**, including format and example values?
4. **What does it return**, including the shape of errors?
5. **What side effects** exist (writes data, sends email, deletes)?

### Recommended structure

```
[One sentence: what the tool does]
[When to use it / when NOT to use it]
[Important notes: limits, side effects, output format]
```

### Practical rules

- **Tool name = verb + object**: `search_orders`, `create_invoice`. Avoid `helper`, `tool1`, `do_stuff`.
- **One tool, one job.** If the description needs "or" many times, split it into several tools.
- **Explicitly differentiate similar tools.** If you have `search_customer` and `get_customer`, state which to pick and when.
- **Give example parameter values** (`"2026-10-01"`, `"ORD-12345"`), not just data types.
- **Mark required vs optional parameters** and explain the defaults.
- **Use `enum`** for parameters with a fixed set of choices. Don't make the model guess free-form strings.
- **Describe what is returned** (which fields, what empty data means) so the agent knows how to read it.
- **Make error messages actionable**: "Date must be YYYY-MM-DD" is more useful than "Invalid input".
- **Warn on dangerous tools**: state that the action is irreversible and when to ask the user for confirmation.
- **Return only relevant data.** Overly long tool output fills the context and confuses the model. Support pagination or filters.

### Example of a good description

```json
{
  "name": "search_orders",
  "description": "Search customer orders by customer ID or date range. Use this when the user asks about order status, history, or lists of orders. Do NOT use it for a single order whose ID is already known; use get_order for that. Returns at most 20 orders, newest first. An empty result means no orders matched (it is not an error).",
  "input_schema": {
    "type": "object",
    "properties": {
      "customer_id": {
        "type": "string",
        "description": "Customer ID, format 'CUS-' followed by digits. Example: 'CUS-00421'."
      },
      "date_from": {
        "type": "string",
        "description": "Start date (inclusive), format YYYY-MM-DD. Example: '2026-09-01'. Optional."
      },
      "status": {
        "type": "string",
        "enum": ["pending", "shipped", "delivered", "cancelled"],
        "description": "Filter by order status. Optional; if omitted, all statuses are returned."
      }
    },
    "required": ["customer_id"]
  }
}
```

### Right vs Wrong Table: Tool Descriptions

| Aspect | ❌ Wrong | ✅ Right |
|---|---|---|
| **Tool name** | `helper`, `tool1`, `process` | `search_orders`, `send_email`, `create_invoice` |
| **Tool description** | `"Gets data"` | `"Gets a customer profile by ID. Use when the user asks about account info. Returns name, email, and subscription status."` |
| **When to use** | Not mentioned at all | `"Use when... Do NOT use for..."` |
| **Similar tools** | `search_x` and `find_x` have nearly identical descriptions | Clearly differentiated: `search_x` for broad search, `find_x` for an already-known ID |
| **Parameters** | `"id": {"type": "string"}` | `"id": {"type": "string", "description": "Order ID, format 'ORD-' + digits. Example: 'ORD-12345'"}` |
| **Limited choices** | `"status": {"type": "string"}` (free-form) | `"status": {"enum": ["pending", "shipped", "delivered"]}` |
| **Optional parameters** | Unclear which are required | Marked `required`, with defaults explained |
| **One tool, many jobs** | `manage_user` (create/delete/update/search via an `action` parameter) | Split: `create_user`, `delete_user`, `update_user`, `search_user` |
| **Tool output** | Returns 5,000 raw rows | Returns relevant fields only, limited, with pagination |
| **Error messages** | `"Error 500"` / `"Invalid input"` | `"Invalid date. Use format YYYY-MM-DD, e.g. 2026-10-01."` |
| **Dangerous tools** | `"Deletes data"` | `"Deletes the order PERMANENTLY; this cannot be undone. Ask the user for confirmation before calling."` |
| **Empty results** | Returns `null` with no explanation | `"No orders found for these criteria."` |
| **Number of tools** | 40 tools at once, many overlapping | Only tools relevant to the agent's task; merge or remove redundant ones |

---

## Part 2: How to Write a Good System Prompt

### Core principle

A system prompt is a **briefing for a new employee**. Explain the context, the goal, and the reasons, not just a list of prohibitions. A model that understands *why* handles unexpected cases far better than one that has only memorized rules.

### Recommended structure

Use this order (drop sections that don't apply):

1. **Role & context**: who this agent is, who it serves, in what situation.
2. **Goal**: what outcome counts as success.
3. **How to work**: general steps, when to use specific tools, when to ask the user back.
4. **Constraints**: what must not be done, **with the reasons**.
5. **Style & output format**: language, tone, length, formatting.
6. **Edge cases**: missing data, out-of-scope requests, tool failures, ambiguous requests.
7. **Examples** (1-3 input → ideal output pairs) for behavior that is hard to describe in words.

### Template

```
You are a [role] for [company/product]. You help [who] with [main task].

## Goal
[The expected outcome. What makes an answer good?]

## How to work
- Start by [first step, e.g. checking whether the user has already provided an order ID].
- Use `search_orders` when [condition]. Use `get_order` if the ID is already known.
- If information is missing, ask one clarifying question before acting.
- After calling a tool, summarize the result in plain language; don't paste raw data.

## Constraints
- Do not promise refunds. Refund policy is decided by the finance team, so direct the user to [contact].
- Do not make up data. If a tool returns no results, say so plainly.

## Style
- English, friendly and concise (max 3-4 sentences unless detail is requested).

## Edge cases
- Out-of-scope questions: politely explain you can't help with that, then offer what you can help with.
- Tool error: retry once; if it still fails, tell the user and don't guess the answer.
```

### Practical rules

- **Explain the reason**, not just the command. "Keep answers short because users read on mobile" works better than "BE SHORT".
- **Say what to do**, not only what to avoid. "Write in flowing paragraphs" is clearer than "Don't use bullets".
- **Avoid ALL CAPS and "MUST/NEVER" everywhere.** Modern models respond strongly to instructions, and heavy emphasis makes them rigid. Save emphasis for the one or two truly critical rules.
- **Be specific, not abstract.** "Be professional" is vague. "Address the user formally, avoid slang and emoji" is clear.
- **Avoid conflicting instructions** (e.g. "answer as briefly as possible" and "always explain thoroughly").
- **Give examples**, but vary them. A single example will be copied verbatim.
- **Don't rewrite tool descriptions in the system prompt.** Give only *strategy* (when to use which tool, in what order, when to stop). Technical details belong in the tool description.
- **Give permission to say "I don't know"** and define what to do when information is missing, to prevent hallucination.
- **Define when the agent should stop or ask**, especially for irreversible actions.
- **Use clear delimiters** (Markdown headers or XML tags like `<rules>`, `<context>`) so sections are easy to tell apart, especially in long prompts or ones containing variable data.
- **Put long documents/data at the top, and instructions and the question at the end** for better results.
- **Iterate from real failures.** Don't add rules "just in case"; add them only when you see a recurring failure.

### Right vs Wrong Table: System Prompts

| Aspect | ❌ Wrong | ✅ Right |
|---|---|---|
| **Role** | `"You are an AI assistant."` | `"You are a customer support agent for the online store Acme Shop. You help buyers check order status and returns."` |
| **Goal** | None | `"Success = the buyer gets an accurate order status within one or two replies."` |
| **Instruction style** | `"NEVER answer at length!!! MUST BE SHORT!!!"` | `"Answer in 2-3 sentences because users read on mobile."` |
| **Prohibitions** | `"Don't discuss prices."` | `"Don't discuss prices because they change; direct the user to the product page for current pricing."` |
| **Direction of behavior** | `"Don't use bullet points."` | `"Write in short, flowing paragraphs."` |
| **Specificity** | `"Be professional and helpful."` | `"Address the user formally, no emoji, acknowledge the user's problem in the first sentence."` |
| **Tool usage** | `"Use tools if needed."` | `"Call search_orders when the user asks about order history; if an order ID is already given, go straight to get_order."` |
| **Tool duplication** | Copies the entire tool schema and parameters into the prompt | Strategy only: when, in what order, when to stop. Details live in the tool description |
| **Missing data** | Not addressed (agent guesses or invents) | `"If a tool finds no data, say so plainly and offer alternatives. Do not make anything up."` |
| **Risky actions** | Not addressed | `"Before cancelling an order, confirm with the user and wait for their reply."` |
| **Out-of-scope cases** | Not addressed | `"If a request is out of scope, decline politely and say what you can help with."` |
| **Instructions** | Conflicting ("be brief" vs "be as complete as possible") | Consistent, with clear priority ("brief by default; detailed if the user asks") |
| **Examples** | None, or a single example that gets copied verbatim | 2-3 varied examples covering normal and edge cases |
| **Structure** | One long paragraph with no separators | Sections with headers or XML tags: role, goal, how to work, constraints, style |
| **Length** | 50 "just in case" rules | Concise; only rules that came from real failures |

---

## Pre-Release Checklist

**Tools**
- [ ] Names are verb + object and easy to understand
- [ ] Descriptions explain what it does, when to use it, and when not to
- [ ] Every parameter has a description, type, and example value
- [ ] `enum` is used for fixed choices
- [ ] Output is concise and relevant, with empty results handled
- [ ] Error messages explain how to fix the problem
- [ ] Dangerous tools carry a warning and a confirmation step
- [ ] No two tools have nearly identical descriptions

**System prompt**
- [ ] Has a role, context, and goal
- [ ] Constraints come with reasons
- [ ] Instructions are positive (what to do), not only prohibitions
- [ ] Edge cases are covered: empty data, out of scope, tool failure, ambiguity
- [ ] No conflicting instructions
- [ ] Includes examples for hard-to-describe behavior
- [ ] Tested with real prompts, not only ideal ones

## Quick Debugging Patterns

| Symptom | Most likely cause | Fix |
|---|---|---|
| Agent picks the wrong tool | Tool descriptions overlap or don't say "when to use" | Add "use when / don't use for" and differentiate similar tools |
| Agent fills parameters wrongly | No format or example values | Add examples, `enum`, and validation with clear error messages |
| Agent makes up answers | No instruction for missing data | Add a rule: "if there's no data, say so plainly" |
| Agent never uses tools | Tool description too vague, or the prompt doesn't say when to use it | Sharpen the description and add usage strategy to the system prompt |
| Agent is too rigid or refuses too often | Prompt full of capital letters and absolute prohibitions | Lower the intensity, explain the reasons, give alternative paths |
| Agent ignores a specific rule | Rule is buried in a long prompt or conflicts with another rule | Add structure, remove contradictions, raise that rule's priority |
| Agent calls the same tool repeatedly | Tool output doesn't clearly signal "done" or "no results" | Make output explicit and add a stopping condition in the prompt |
