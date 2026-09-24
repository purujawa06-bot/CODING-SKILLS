---
name: rules-write-code
description: Mandatory coding standards that MUST be followed whenever writing, editing, generating, or refactoring code in any programming language or task. Always consult and apply this skill for any code-writing task — new features, bug fixes, refactors, scripts, snippets, or reviews — even if the user does not explicitly ask for "clean code" or "English comments". Enforces production-grade clean code, relevant comments/JSDoc, and English-only naming and documentation.
---

# Rules: Write Code

Mandatory ruleset for producing code. Apply these rules to **every** code output, without exception, unless the user gives an explicit conflicting instruction for that specific task.

## 1. Clean, Production-Standard Code

- Write code that is clean, readable, and follows the idiomatic conventions of the language/framework in use.
- Use descriptive, meaningful names for variables, functions, classes, and files (no `x`, `tmp`, `data2`, etc. unless truly trivial/local scope).
- Keep functions small and single-purpose (Single Responsibility Principle).
- Avoid duplication (DRY) — extract shared logic into reusable functions/modules.
- Handle errors and edge cases explicitly; never silently swallow exceptions.
- Follow consistent formatting/style (indentation, spacing, naming convention), matching the existing project style if one is present.
- Avoid dead code, commented-out code blocks, and unnecessary complexity.
- Prefer explicit, predictable logic over clever/obscure one-liners.
- Validate inputs and sanitize outputs where relevant, especially for security-sensitive code (auth, DB queries, file I/O, user input).
- Structure code for maintainability: logical file/folder organization, clear separation of concerns.

## 2. Comments & Documentation

- Add comments only where they add real value: explain **why**, not **what**, when the code isn't self-explanatory.
- For functions, classes, and modules with non-trivial behavior, add proper documentation blocks (JSDoc for JS/TS, docstrings for Python, XML doc comments for C#, etc.) describing:
  - Purpose / summary
  - Parameters (name, type, description)
  - Return value
  - Exceptions/errors thrown (if any)
- Skip comments/docs for trivial, self-explanatory code (e.g., simple getters/setters, one-line utilities).
- Keep comments in sync with the code — remove or update stale/misleading comments.

## 3. Language: English Only

- All code artifacts must be written in English: variable/function/class/file names, comments, docstrings/JSDoc, commit messages, log messages, and error messages.
- Do not use local/regional language (e.g., Indonesian) inside code, comments, or documentation — regardless of what language the user is speaking in the conversation.
- **Exception:** string literals meant as user-facing content in a non-English locale (e.g., UI text for a localized app, translation files) may stay in the target language, since that is product content, not code documentation.

## Application

- Apply this skill automatically to every request involving writing, generating, editing, or reviewing code — do not wait for the user to explicitly ask for "clean code" or "English comments".
- If editing existing code that violates these rules, match the surrounding style for the edit itself, but briefly flag the inconsistency to the user.
- An explicit, task-specific user instruction (e.g., "no comments for this one", "keep this variable name in Indonesian") overrides these defaults for that task only.
