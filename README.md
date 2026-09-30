# CODING-SKILLS

> Reusable skills for AI coding agents. Small, focused, production-minded.

**CODING-SKILLS** is a collection of portable `SKILL.md` instructions that help AI coding agents write cleaner, safer, and more maintainable code.

No framework. No runtime. No dependencies. Just copy a skill into your agent's skills directory and it works.

## Skills

| Skill | Purpose |
|-------|---------|
| [`rules-write-code`](skills/rules-write-code/SKILL.md) | Mandatory standard for writing, editing, generating, and reviewing code. Enforces clean idiomatic code, meaningful names, small focused functions, DRY, explicit error handling, input validation, `why`-focused documentation, and English-only code artifacts. Applies automatically to every coding task. |

## Installation

```bash
git clone https://github.com/purujawa06-bot/CODING-SKILLS.git
```

Copy what you need:

```bash
# Claude Code
cp -r CODING-SKILLS/skills/rules-write-code ~/.claude/skills/

# PuruClaw
cp -r CODING-SKILLS/skills/rules-write-code ~/.puru/workspace/skills/

# Generic agent (adjust to your agent's skills dir)
cp -r CODING-SKILLS/skills/rules-write-code <skills-dir>/
```

## Usage

Once installed, agents that support the `SKILL.md` convention load the skill automatically.

For `rules-write-code`:

```text
Coding task → rules-write-code → apply standards → generate / edit / review
```

It is not a library — nothing to import into your app.

## Repository Structure

```text
CODING-SKILLS/
├── skills/
│   └── rules-write-code/
│       └── SKILL.md
├── LICENSE
└── README.md
```

Each skill is self-contained: one directory, one `SKILL.md` with frontmatter (`name`, `description`) plus instructions.

## Design Principles

- **Focused** — one skill, one job. No instruction dumps.
- **Practical** — improves real workflows, no ceremony.
- **Portable** — plain Markdown, works across agents that support `SKILL.md`.
- **Maintainable** — short enough to evolve with practice.

## Contributing

1. Create `skills/<your-skill>/SKILL.md` with `name` + `description` frontmatter.
2. Keep scope tight and instructions actionable.
3. Update this README's Skills table.
4. Open a PR.

## License

MIT — see [LICENSE](LICENSE).

---

Built for AI-assisted development by [purujawa06-bot](https://github.com/purujawa06-bot).
