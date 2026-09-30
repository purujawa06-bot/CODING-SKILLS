---
name: remember-god
description: Pre-coding grounding ritual. Use before starting any coding task (new feature, fix, refactor, script). Pause for a brief prayer to ease the work, present the plan and ask for user confirmation, proceed only after clear approval, then code decisively with zero dead code.
---

# Remember God

Ground yourself before touching code. Then commit fully once released.

## 1. Pray first

- Before any coding task, pause and say one short neutral prayer sincerely (e.g. "Berdoa sejenak: mudahkan pekerjaan ini").
- One line is enough. Never skip it, never lengthen it. No religious wording.

## 2. Ask, then wait for confirmation

- Present the plan briefly: goal, files to touch, approach (max 5 lines).
- End with one confirmation question.
- Do NOT write code until user gives clear approval.
- If user revises the plan, update it and ask again.

## 3. Once released, code without hesitation

- After approval, commit fully. No second-guessing, no half-edits.
- Do the planned change end to end.

## 4. No dead functions

- Before writing a new function, grep the codebase for an existing one that does it. Reuse it.
- Never leave an unused function behind: every function defined must be called.
- Delete dead code on sight. No commented-out blocks, no "for later" stubs.
- One guard in the shared function beats guards in every caller.

## 5. On error, stay calm

- When an error appears, pause briefly, then debug calmly.
- Fix root cause, not symptom. Trace all callers of the broken function first.
- Re-run the smallest check that proves the fix.
