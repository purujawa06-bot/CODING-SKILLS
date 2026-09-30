# CODING-SKILLS

> A focused, production-minded skill for AI coding agents.

CODING-SKILLS provides reusable instructions that help AI coding agents write cleaner, safer, and more maintainable code. The project is intentionally lightweight: each skill is self-contained and can be added to an agent's skill directory without introducing a framework or runtime dependency.

## ✨ What It Provides

The repository currently includes:

### `rules-write-code`

A mandatory coding standard for writing, editing, generating, or refactoring code.

It guides agents to:

- Write clean, readable, idiomatic code
- Prefer meaningful names and small, focused functions
- Follow DRY and separation-of-concerns principles
- Handle errors and edge cases explicitly
- Validate inputs and sanitize security-sensitive data
- Avoid dead code and unnecessary complexity
- Add useful documentation for non-trivial code
- Keep comments focused on **why**, not obvious implementation details
- Use English for code artifacts, comments, documentation, logs, and error messages

The skill is designed to be applied automatically whenever an agent performs a coding task.

## 📁 Repository Structure

```text
CODING-SKILLS/
└── skills/
    └── rules-write-code/
        └── SKILL.md
```

Each skill lives in its own directory and is defined by a `SKILL.md` file containing its metadata, trigger description, and instructions.

## 🚀 Installation

Clone the repository:

```bash
git clone https://github.com/purujawa06-bot/CODING-SKILLS.git
```

Then copy the skill you want into the skill directory supported by your AI coding agent.

For example:

```bash
cp -r CODING-SKILLS/skills/rules-write-code <your-agent-skills-directory>/
```

The exact installation path depends on the agent you use.

## 🧩 Usage

Once installed, the skill can be loaded by an AI coding agent that supports the `SKILL.md` convention.

For `rules-write-code`, the intended behavior is simple:

```text
Coding task
    ↓
rules-write-code
    ↓
Apply coding standards
    ↓
Generate / edit / review code
```

The skill is not a library and does not need to be imported into your application.

## 🎯 Design Principles

CODING-SKILLS is built around a few simple principles:

**Focused** — Skills should solve a specific problem instead of becoming a large instruction dump.

**Practical** — Rules should improve real coding workflows, not add ceremony for its own sake.

**Reusable** — Skills should be portable across AI coding agents that support the `SKILL.md` format.

**Maintainable** — Instructions should be clear enough to evolve as coding practices and agent capabilities change.

## 🤖 Compatibility

The skills are written as portable Markdown instructions and are intended for AI coding agents that support the `SKILL.md` skill format.

Compatibility may vary between agents depending on how they discover and load skills.

## 🛠️ Adding a Skill

To add a new skill:

1. Create a directory under `skills/`.
2. Add a `SKILL.md` file.
3. Define the skill metadata and instructions.
4. Keep the scope focused and the instructions actionable.
5. Update this README when the repository gains a meaningful new capability.

Example:

```text
skills/
├── rules-write-code/
│   └── SKILL.md
└── your-new-skill/
    └── SKILL.md
```

## 📌 Philosophy

Good AI coding assistance is not only about generating code quickly. It should also encourage consistency, readability, maintainability, and predictable engineering practices.

CODING-SKILLS keeps those expectations in small, reusable building blocks so they can be applied wherever the agent is working.

## 📄 License

See the repository license file for licensing information.

---

Built for AI-assisted software development by [purujawa06-bot](https://github.com/purujawa06-bot).
