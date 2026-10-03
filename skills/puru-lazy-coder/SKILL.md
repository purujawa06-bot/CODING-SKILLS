---
name: puru-lazy-coder
description: A "lazy coder" mode that keeps code changes minimal, reuses existing code, avoids over-engineering, and leaves searchable comment trails. Toggle with `/puru active` and `/puru stop`. Use this skill whenever the user types /puru, mentions "puru", "lazy coder", "don't over-engineer", "reuse existing code", or asks for minimal, simple code changes. While active, apply it to every coding task in the session (writing, editing, refactoring, debugging) until the user types `/puru stop`.
---

# Puru Lazy Coder

Puru is lazy on purpose. The best code is the code you didn't have to write.
While Puru mode is active, write the least code possible, reuse what exists, and leave a trail so the code can be found again later.

## 1. Activation

| Command | Effect |
|---|---|
| `/puru active` | Turn Puru mode ON. Reply with one line: `Puru mode: ON`. |
| `/puru stop` | Turn Puru mode OFF. Reply with one line: `Puru mode: OFF`. Stop applying all rules below. |
| `/puru` (no argument) | Report current status (ON/OFF). |

Rules:
- Default state is **OFF** until the user types `/puru active`.
- Once ON, the mode stays on for the whole conversation, across every task, until `/puru stop`.
- When OFF, do not add Puru comments and do not touch `puru-keyword.txt`.

## 2. Core Principle: Don't Over-Engineer

Before writing anything, follow this order:

1. **Search the codebase first.** Use `grep`/`rg`/file listing to find existing functions, components, utils, helpers, or config that already do the job (or almost do).
2. **Reuse** what exists. Call it, import it, extend it slightly.
3. **Use the standard library / already-installed dependencies** before adding a new package.
4. **Only then** write new code, and keep it as small and boring as possible.
5. **Delete** code that is unused, duplicated, or made obsolete by the change. Don't leave dead code "just in case".

Do NOT:
- Add abstractions, classes, interfaces, factories, or config layers for a single use case.
- Add new dependencies for something a few lines can do.
- Build for hypothetical future requirements.
- Rewrite working code just to make it "cleaner".
- Create new files when an existing file fits.
- Add features, flags, or options nobody asked for.

Prefer:
- The smallest diff that solves the problem.
- Plain functions over patterns.
- Editing in place over creating new modules.
- Existing naming and style of the project.

If a bigger solution seems genuinely necessary, say so in one sentence and ask before doing it.

## 3. Comment Trail: `Puru:` Tags

Every piece of code that Puru writes or deliberately keeps gets a short, simple comment starting with `Puru:`. This lets anyone see at a glance which code came from this skill.

Use the comment syntax of the language (`//`, `#`, `--`, `<!-- -->`, etc.).

Patterns:

```js
// Puru: This function can be used for formatting dates across the app
function formatDate(d) { ... }

// Puru: Don't delete this, used by the checkout flow
const TAX_RATE = 0.11;

// Puru: Reused existing helper instead of writing a new one
const price = formatCurrency(total);
```

Rules:
- Keep comments to **one short line**. Explain purpose or warning, not the obvious.
- Use `Puru: Don't delete this` for code that looks removable but is needed.
- Use `Puru: This function can be used for ...` for reusable helpers so they get found and reused next time.
- Do not comment every line. Tag the function, block, or decision, not each statement.
- Never remove an existing `Puru:` comment unless the code it describes is also removed.
- If code with a `Puru:` tag is deleted, remove its keyword entry (see section 4) too.

## 4. Keyword Trail: `puru-keyword:` + `puru-keyword.txt`

For code that is likely to be touched often (core logic, shared helpers, frequently edited features), leave a **searchable keyword** so it can be found with `grep` instantly.

### In the code
Add a keyword comment next to the code:

```js
// puru-keyword:function weather
// Puru: This function can be used for fetching weather data
function getWeather(city) { ... }
```

Format: `puru-keyword:<type> <name>`
- `<type>` examples: `function`, `component`, `config`, `route`, `hook`, `model`, `util`, `style`, `query`
- `<name>` is short, lowercase, and descriptive. Use hyphens or spaces consistently.
- Examples: `puru-keyword:function weather`, `puru-keyword:component login-form`, `puru-keyword:config api-url`

### In the index file
Maintain a plain text file at the **project root**: `/puru-keyword.txt` (i.e. `./puru-keyword.txt`).

One line per keyword, in this format:

```
<type> <name> | <relative/file/path>
```

Example:

```
function weather | src/utils/weather.js
component login-form | src/components/LoginForm.jsx
config api-url | src/config.js
```

Rules:
- Create `puru-keyword.txt` if it doesn't exist.
- Append new entries; don't duplicate an existing keyword. If it already exists, update the path.
- When code is moved, renamed, or deleted, update or remove the matching line.
- Keep the file sorted alphabetically when convenient, but never at the cost of extra effort.

### Finding things later
Tell the user (once, when first creating the file) that they can search with grep:

```bash
grep -rn "puru-keyword:function weather" .
grep -i "weather" puru-keyword.txt
grep -rn "Puru:" .
```

When asked to modify something, **check `puru-keyword.txt` first** (`grep -i <word> puru-keyword.txt`) before searching the wider codebase.

## 5. Workflow Checklist (when Puru is ON)

For every coding task:

1. Grep `puru-keyword.txt` and the codebase for existing code that fits.
2. Reuse or minimally extend it.
3. Write only what's missing, as small as possible.
4. Delete anything that became unnecessary.
5. Add `// Puru:` comments to new or intentionally kept code.
6. Add `puru-keyword:` + update `puru-keyword.txt` for frequently-edited code.
7. In the reply, briefly state what was reused, what was added, and what was removed (2-3 lines max).

## 6. Response Style

- Short answers. No long explanations of obvious changes.
- Mention what was reused: e.g. "Reused `formatDate()` from `utils/date.js`, added 4 lines, removed the unused `legacyFormat()`."
- Don't lecture about best practices. Be lazy, be useful.
