# orchestrator

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

The skill is a single folder (this one). Copy it where your agent runtime looks for skills.

**Claude Code** (global — available in every project):

```bash
git clone https://github.com/pinjun99/Sildenafil_coding
mkdir -p ~/.claude/skills
cp -r Sildenafil_coding/skills/orchestrator ~/.claude/skills/
```

**Codex CLI:**

```bash
mkdir -p ~/.codex/skills
cp -r Sildenafil_coding/skills/orchestrator ~/.codex/skills/
```

(For a single Claude Code project instead of global: `.claude/skills/orchestrator/` inside the repo.)

No config changes needed — in Claude Code the skill self-triggers from its frontmatter description when a task looks wide; on other runtimes auto-triggering varies, so invoke it explicitly or pin it (below).

## Use

Mostly: don't think about it. Describe your task normally; when it's wide (a multi-part build, a repo-wide refactor, a bulk audit), the skill loads and the model orchestrates. To force it explicitly:

- Claude Code: `/orchestrator build a habit-tracker app with auth, history and charts`
- Codex: `$orchestrator ...`

**Reliable activation tip:** for projects where you always want fan-out discipline on wide work, add one line to the project's `CLAUDE.md` / `AGENTS.md`: *"For wide multi-slice tasks, follow the `orchestrator` skill."*

When your task is small ("fix this crash"), the skill's own gate tells the model **not** to fan out — that's by design.

## Worked example (one slice of a real fan-out)

Task: "migrate 30 API handlers to the new `ApiError` type." Wide (30 independent files) → gate passes. The orchestrator commits a checkpoint, writes `contracts/api-error.ts` with the exact `ApiError` signature, splits the handlers into 4 batches, and sends each worker a packet like:

```
OBJECTIVE
Migrate the 8 handlers listed below to throw ApiError instead of raw Error.

CONTEXT
Repo path: /work/api
Starting files: contracts/api-error.ts (the signature you must conform to)
Background: error middleware already handles ApiError; handlers predate it.

SCOPE
In scope: src/handlers/{auth,billing,users,webhooks}/*.ts — the 8 files listed.
Out of scope: the middleware, tests outside these handlers, any other handler.

SEAMS
Contract file: contracts/api-error.ts — conform exactly.
Files you own: src/handlers/auth/*.ts, src/handlers/billing/*.ts, ...
Shared files (manifests, lockfiles, barrels, registries): do NOT edit —
declare needs in your report's NEEDS section.

EVIDENCE REQUIRED IN YOUR REPORT
- Line 1: your exact model ID
- Diffs for every change; commands run with real output
- NEEDS (or "none"); failures and uncertainties

VERIFICATION
Command(s): npx vitest run src/handlers/auth src/handlers/billing
Success criteria: all tests pass; zero remaining `throw new Error` in owned files.

STOP CONDITIONS + CONDUCT
(as in the skill template)
```

Workers run in parallel. At vetting, the orchestrator checks each report's model ID, attributes each diff against the checkpoint (`git diff <checkpoint> -- <owned globs>`), applies the collected NEEDS itself, re-runs the full suite once all workers are terminal — and in this example catches that worker 2's diff touched a barrel file outside its globs: ownership violation, slice re-spawned once with a corrected packet.

## When this is the wrong pattern

This skill assumes your session runs a high-tier model and delegates *down*. If you run cheap sessions and want to escalate hard decisions *up* to a stronger model, you want the inverse (executor/advisor) pattern — a cheap model owning the loop and consulting a premium model on-demand. That's this repo's [advisor](../advisor/) skill; this one deliberately stays a pure orchestrator.

## Cost honesty

Fan-out pays only when slices are genuinely independent and bulky. On wide workloads I've seen multi-x cost and wall-clock savings in my own use; on narrow work the overhead makes it *slower and more expensive* — which is why the go/no-go gate is the first section of the skill, why the gate re-runs mid-flight with an abort rule, and why fan-out width is capped by default.

## Credits

Pattern inspired by [BuilderIO's `efficient-fable`](https://github.com/BuilderIO/skills/tree/main/skills/efficient-fable) skill. This is an independent rewrite: cross-platform, with an explicit model-tier policy, seam-first delegation, a concrete packet template, and a self-refusing gate.

## License

MIT
