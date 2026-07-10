# advisor

**A senior engineer on call for your mid-tier model.** The session model executes every turn; a top-tier model is consulted only at hard trigger points — then the ruling is cached so you never pay for the same question twice.

```
┌─────────────────┐        consultation packet        ┌──────────────────┐
│    Executor      │ ────────────────────────────────► │     Advisor      │
│   (mid tier)     │                                    │   (top tier)     │
│  runs every turn │ ◄──────────────────────────────── │    on-demand     │
└───────┬─────────┘        ruling + reasoning          └──────────────────┘
        │  ▲                                                    │
        └──┘ main loop                            rulings cached in ADVISOR.md
```

This is the inverse of the [orchestrator](../orchestrator/) skill:

| | orchestrator | advisor |
|---|---|---|
| Who owns the loop | top-tier model | mid-tier model |
| Cheap models do | bounded work slices | — (the mid-tier model does everything) |
| Top-tier model does | plan, integrate, review | answer consultations, review before ship |
| Wins when | work is wide and parallel | work is routine with occasional hard calls |
| Saves money by | not paying top-tier for grunt work | not paying top-tier for the loop at all |

## The problem this solves

A mid-tier model's worst mistakes are **confident** ones — it doesn't know when it's out of its depth, so "ask for help when unsure" fails precisely when it matters. This skill removes judgment from the decision entirely:

- **Mechanical triggers, checked at the point of action** — touching auth/payment/personal-data code, dangerous code patterns (eval on strings, SQL concatenation, disabled TLS…), schema and interface lock-ins, irreversible commands, a third consecutive failed fix. No trigger asks the model how it feels.
- **A mandatory pre-ship review** — before work is pushed, PR'd, deployed, or declared done, the advisor reviews the diff, with test files always shown in full ("was any test weakened to get to green?").
- **A decision log with teeth** — rulings cached in `ADVISOR.md`, so repeat questions are free. But the log is a *lead, never an authority*: security rulings can never be discharged from cache, and a log the session didn't write is shown to the user before first reuse (a poisoned log in a cloned repo must not be able to disarm the triggers).
- **Advisor identity verification** — every ruling starts with the advisor's model ID, checked against a tier table. If the model override silently failed and a mid-tier model answered, the answer is discarded: a model consulting itself and logging it as top-tier wisdom is worse than no skill.
- **Terminal states** — every consultation ends applied, logged-and-followed, or escalated to the user with both positions stated. Silence never resolves to "proceed."

## Install

```bash
git clone https://github.com/pinjun99/Sildenafil_coding
mkdir -p ~/.claude/skills
cp -r Sildenafil_coding/skills/advisor ~/.claude/skills/     # Claude Code
cp -r Sildenafil_coding/skills/advisor ~/.codex/skills/      # Codex CLI
```

**Reliable activation tip:** skill auto-loading matches the *task*, not your model tier. For sessions you know run mid-tier, add one line to the project's `CLAUDE.md` / `AGENTS.md`:

> This session runs a mid-tier model — follow the `advisor` skill.

## Use

Run your session on a mid-tier model (e.g. Sonnet, GPT-5.6 Terra) and work normally. The skill consults upward only at trigger points. If your session already runs the top tier, the skill stands down by itself — there's no one above to ask.

## Worked example

Executor (Sonnet) is asked to "add login to the app." It's about to write password handling — security trigger fires *before any code is written*. It sends:

```
DECISION NEEDED
How should passwords be stored for a small Next.js + Postgres app (~1k users)?

CONTEXT
Repo path: /work/habit-app
Task: add email+password login
Relevant code: // users table has: id, email, created_at (schema.sql:3-7)
               // no auth code exists yet

OPTIONS CONSIDERED
1. bcrypt via bcryptjs — simple, widely used; unsure about cost factor
2. argon2id via node-argon2 — stronger, but native module complicates deploys

CONSTRAINTS
Deploys to Vercel serverless; no ops team; must not add heavy infra.

PRIOR RULINGS
none (ADVISOR.md does not exist yet)

EVIDENCE
not applicable — decision precedes code

ANSWER FORMAT (advisor must follow)
Line 1: your exact model ID.
Then: RULING / REASONING / RISKS / WHAT WOULD CHANGE THIS RULING.
You advise only — change nothing, execute nothing.
```

Advisor answers (line 1: `claude-fable-5` — verified top-tier ✓): *RULING: bcrypt via bcryptjs, cost 12, plus a unique salt per user (library default) — argon2's native build pain on serverless outweighs its margin here. RISKS: cost 12 needs revisiting if login latency matters… WHAT WOULD CHANGE THIS: >100k users or a compliance requirement naming argon2.*

Executor applies the ruling, then creates `ADVISOR.md`:

```
## 2026-07-11 — Password hashing scheme for email+password login
Ruling: bcryptjs, cost 12, per-user salt (library default).
Reasoning: serverless deploy makes argon2 native builds fragile; bcrypt at
cost 12 is adequate at this scale and zero-ops.
Premises: deploys on serverless; ~1k users; no compliance mandate on KDF.
```

Next week, "add password change" hits the same trigger. The log has an on-point ruling — but this is a **security-category** trigger, so it can't be discharged from cache: the executor consults anyway, with the prior ruling attached. The advisor confirms in one cheap round-trip. Had it been a *design*-category repeat (say, "should this new table use the same soft-delete pattern?" after a logged data-model ruling), the cached ruling would apply free, no consult.

## Economics — honest version

One consultation costs a fraction of a top-tier session (the advisor sees one packet, not your history). The pattern nets positive while consults stay rare — the triggers are tuned for a handful per day of real work, and a built-in guard pauses on the 3rd consult within one task to ask whether you'd be better off just running top-tier. If you consult constantly, you don't need an advisor; you need a better executor. And a diff review can't catch runtime-only failures — the skill says so rather than pretending otherwise.

## License

MIT — see repo [LICENSE](../../LICENSE).
