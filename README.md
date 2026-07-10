# Sildenafil_coding

**Performance enhancement for your coding agent.** 💊

A collection of agent skills I use in my own daily work — built and tested with real projects, written to work across tools. Each skill is a plain-markdown playbook that your coding agent loads and follows; no binaries, no dependencies, nothing to build.

Compatible with **Claude Code**, **Codex CLI**, and any agent runtime that adopts the `skills/<name>/SKILL.md` convention.

## Skills

| Skill | What it does |
|---|---|
| [orchestrator](skills/orchestrator/) | Your top-tier model plans and reviews; cheaper models do the bounded heavy lifting in parallel. Fan-out with discipline: seams defined before delegation, self-contained handoff packets, trust-but-verify review. Saves premium tokens on wide tasks — and refuses to activate on narrow ones. |
| [advisor](skills/advisor/) | The inverse: a mid-tier model executes every turn and consults a top-tier model only at hard trigger points — security-sensitive code, schema lock-ins, irreversible commands, repeated failures — plus a mandatory diff review before anything ships. Rulings cached in a decision log so a question is never paid for twice. |

*(More coming as I extract them from my own workflow.)*

## Install

Every skill is one self-contained folder. Copy the folder where your agent looks for skills — that's the entire install.

**Claude Code** (global — available in every project):

```bash
git clone https://github.com/pinjun99/Sildenafil_coding
mkdir -p ~/.claude/skills
cp -r Sildenafil_coding/skills/orchestrator ~/.claude/skills/
```

**Codex CLI:**

```bash
cp -r Sildenafil_coding/skills/orchestrator ~/.codex/skills/
```

**Single project only** (instead of global): copy into `.claude/skills/` inside the project repo.

To install a different skill from this collection, swap the folder name. To update, pull and copy again. To uninstall, delete the folder.

## Use

Mostly: don't think about it. Each skill declares in its frontmatter *when* it applies, and your agent loads it automatically when a task matches. To invoke one explicitly:

- Claude Code: `/orchestrator <your task>`
- Codex: `$orchestrator <your task>`

Each skill's own README documents what it does, when it triggers, and when it deliberately stays out of the way.

## Why "Sildenafil_coding"?

I'm a pharmacist. Sildenafil is the classic performance-enhancing molecule. These skills do for coding agents what— you get it.

## License

MIT — see [LICENSE](LICENSE).
