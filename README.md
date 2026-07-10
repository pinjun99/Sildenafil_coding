# efficient-orchestrator

A cross-platform agent skill that makes your **top-tier model orchestrate** and your **cheaper models do the heavy lifting** — with the discipline that makes fan-out actually work: seams defined before delegation, self-contained handoff packets, and trust-but-verify review of everything workers report.

Works in **Claude Code**, **Codex CLI**, and any agent runtime that adopts the `SKILL.md` convention.

```
                        ┌──────────────┐
                   ┌──► │  Worker 1    │  scan / summarize
                   │    │  (lower tier)│
┌──────────────┐   │    └──────────────┘
│ Orchestrator │───┤    ┌──────────────┐
│  (top tier)  │   ├──► │  Worker 2    │  bounded code edits
│ plan · seams │   │    │  (mid tier)  │
│ vet · verify │   │    └──────────────┘
└──────────────┘   │    ┌──────────────┐
                   └──► │  Worker 3    │  targeted tests
                        │  (mid tier)  │
                        └──────────────┘
```

## Why

Running a premium model end-to-end burns premium tokens on work that needs no premium judgment: scanning repos, summarizing docs, reducing logs, grinding through repetitive edits, running tests. The orchestrator pattern splits the bill — the top tier keeps the judgment (decomposition, architecture tradeoffs, integration, review) and everything bounded fans out to cheaper workers running in parallel.

The tier ladder is provider-agnostic — the same rule everywhere: **highest tier orchestrates, lower tiers work, judgment never gets delegated down.**

| Role | Anthropic (example) | OpenAI (example) |
|---|---|---|
| Orchestrator | Fable 5 / Opus | GPT-5.6 Sol |
| Code & test workers | Opus / Sonnet | GPT-5.6 Terra |
| Scan & summarize workers | Sonnet | GPT-5.6 Luna |

Model names age; the ladder doesn't. Substitute your provider's current lineup.

## What makes this version different

Compared to typical delegation prompts, this skill adds the parts that usually go wrong:

- **A go/no-go gate.** Fan-out has negative value on narrow work. The skill's first instruction is a test ("could three people do this simultaneously without talking to each other?") and an explicit refusal list — tiny fixes, coupled edits, single-thread debugging stay with the orchestrator.
- **An explicit model-tier policy.** Not "use cheaper agents" hand-waving: a tier table with a mid-tier floor for production code, and permission to promote a worker's tier when a slice carries more judgment than expected.
- **Seams before fan-out.** The classic fan-out failure is pieces that don't fit at merge time. The orchestrator writes down interfaces, shared contracts, and per-worker file ownership *before* any worker launches.
- **A literal handoff-packet template.** Workers start with zero context. The packet is copy-paste-ready — objective, scope, seams, required evidence, verification command, stop conditions — plus conduct rules pasted verbatim into every packet.
- **Leads, not facts.** Worker reports get vetted: line references spot-checked, diffs reviewed, and builds/tests re-run by the orchestrator before anything counts as done.
- **Phase slicing by feature, never by layer.** Multi-phase work must end every phase with something runnable.

## Install

The skill is a single folder. Copy it where your agent runtime looks for skills.

**Claude Code** (global — available in every project):

```bash
git clone https://github.com/<you>/efficient-orchestrator
mkdir -p ~/.claude/skills
cp -r efficient-orchestrator/skill ~/.claude/skills/efficient-orchestrator
```

**Codex CLI:**

```bash
cp -r efficient-orchestrator/skill ~/.codex/skills/efficient-orchestrator
```

(For a single project instead of global: `.claude/skills/efficient-orchestrator/` inside the repo.)

No config changes, no instruction-file edits — the skill self-triggers from its frontmatter description when a task looks wide, and you can always invoke it explicitly.

## Use

Mostly: don't think about it. Describe your task normally; when it's wide (a multi-part build, a repo-wide refactor, a bulk audit), the skill loads and the model orchestrates. To force it explicitly:

- Claude Code: `/efficient-orchestrator build a habit-tracker app with auth, history and charts`
- Codex: `$efficient-orchestrator ...`

When your task is small ("fix this crash"), the skill's own gate tells the model **not** to fan out — that's by design.

## When this is the wrong pattern

This skill assumes your session runs a high-tier model and delegates *down*. If you run cheap sessions and want to escalate hard decisions *up* to a stronger model, you want the inverse (executor/advisor) pattern — a cheap model owning the loop and consulting a premium model on-demand. That is a different skill; this one deliberately stays a pure orchestrator.

## Cost honesty

Fan-out pays only when slices are genuinely independent and bulky. On wide workloads, 2–5x cost savings and 2–4x wall-clock speedups are realistic; on narrow work the overhead makes it *slower and more expensive* — which is why the go/no-go gate is the first section of the skill.

## Credits

Pattern inspired by [BuilderIO's `efficient-fable`](https://github.com/BuilderIO/skills/tree/main/skills/efficient-fable) skill. This is an independent rewrite: cross-platform, with an explicit model-tier policy, seam-first delegation, a concrete packet template, and a self-refusing gate.

## License

MIT
