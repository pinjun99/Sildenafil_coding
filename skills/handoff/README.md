# handoff

**A plan that defends itself.** A top-tier model writes the plan today; cheaper sessions execute it brief-by-brief — tomorrow, headless, or on a budget model — with no planner in the room. The discipline is in the files: premises that check reality, verification that catches silent drift, and a manifest whose `NEXT:` line always tells you what to do.

```
 TODAY (top tier)                LATER (mid tier, any session, unsupervised)
┌───────────────┐    handoff/   ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌──────────┐
│    Planner    │ ────────────► │ brief 01│─►│ brief 02│─►│ brief 03│─►│  final   │
│ decides all   │   manifest    │ session │  │ session │  │ session │  │  review  │
│ lock-ins now  │   contracts   └────┬────┘  └────┬────┘  └────┬────┘  │(top tier)│
└───────────────┘   briefs           └── STATE.md journal ──┘          └──────────┘
```

This completes the repo's trilogy:

| | [orchestrator](../orchestrator/) | [advisor](../advisor/) | handoff |
|---|---|---|---|
| Pattern | fan-out in **space** | escalate **up** | fan-out in **time** |
| Top tier does | plans + vets, live | answers consults | writes the plan today |
| Cheap tier does | parallel slices, live | owns the loop | executes briefs later |
| Wins when | work is wide *now* | routine work, rare hard calls | work spans sessions/models/days |

## The problem this solves

The planner isn't there when brief 04 turns out to be wrong. Nobody adapts; drift compounds silently. Four mechanisms replace supervision:

- **Premises check reality, not the ledger.** Every brief opens with `CHECK:`/`EXPECT:` command pairs — including *re-running each dependency brief's verification against today's HEAD*. "Brief 02 is marked done" is a claim; a passing check is a fact. Plus: clean-tree gate (a dirty tree means a session died — that's a reported state, not raw material), plan-revision stamp (a re-planned brief can't run stale), and a lock against concurrent sessions.
- **Two-tier verification.** Every brief must pass its own scoped check **and** the plan-wide regression command. Per-brief green with a broken whole is this pattern's quietest failure — so the whole is checked at every step.
- **Terminal states with a paper trail.** Every session ends in exactly one of done-and-verified / blocked / deviation / crashed-predecessor / paused-mid-brief — written to the journal, stamped with the executor's model ID, committed to git, and surfaced as a one-line `NEXT:` the user can't miss. Silence never resolves to done, and a session that hits its usage limit parks its half-done work in a labeled commit for the next session to continue — instead of losing it.
- **The planner gets re-summoned, mechanically.** Deviations and broken premises set `NEXT: re-plan required`; a top-tier session revises the plan, bumps its revision, and classifies every already-done brief as *stands* or *invalidated*. And the last brief is always a **top-tier final review** — the only thing allowed to declare the plan complete, auditing contracts, model IDs, and weakened tests first-hand.

## Install

```bash
git clone https://github.com/pinjun99/Sildenafil_coding
mkdir -p ~/.claude/skills ~/.codex/skills
cp -r Sildenafil_coding/skills/handoff ~/.claude/skills/     # Claude Code
cp -r Sildenafil_coding/skills/handoff ~/.codex/skills/      # Codex CLI
```

## Use

**Kickoff** (run on a top-tier model):

> /handoff plan: build a medication-reminder web app — auth, schedules, notification log. Write the plan to handoff/, don't execute.

**Continue** (any later session, mid-tier is the point):

> /handoff continue

or headless: `claude -p "execute the next brief in handoff/"` — scheduled, unattended execution is this skill's home ground.

The planner's last act adds a pointer line to your project's `CLAUDE.md`/`AGENTS.md`, so future sessions find the plan without you explaining anything.

## Worked example (miniature)

Manifest after two sessions:

```
handoff-plan v1
Goal: medication-reminder app MVP
Planner: claude-fable-5 · PLAN REVISION: 1
Regression: npx vitest run
| # | brief            | deps | status  | commit  |
| 01| contract stubs   | —    | done    | a1f2c3d |
| 02| schedule CRUD    | 01   | done    | b4e5f6a |
| 03| reminder engine  | 02   | pending |         |
| 04| final review     | 03   | pending |         |
NEXT: execute brief 03
```

Session 3 (cold Sonnet) picks up brief 03. Its premises: manifest header + revision 1 ✓, NEXT names 03 ✓, `git status --porcelain` empty ✓, *re-run brief 02's scoped verification against HEAD* ✓. It takes the lock, builds the engine, runs both checks — scoped ✓, regression ✓ — appends its STATE entry (model ID first line, summary lines only), flips its row, sets `NEXT: execute brief 04`, commits `handoff: brief 03 done-and-verified`, and prints the NEXT line.

**The failure path, because that's where the design earns its keep:** suppose the user had hand-edited the schedule code between sessions and broken it. Brief 03's dependency premise — re-running 02's verification — fails *today* even though the ledger says done. The executor doesn't improvise: it lands `blocked-reported`, writes the failing output to STATE, sets `NEXT: re-plan required — brief 02 verification fails at HEAD`, commits, and tells you. One short top-tier session later the plan is revised (revision 2, done-briefs classified, stale briefs re-stamped) and execution resumes.

## Honest economics

Planning costs a full top-tier session before any work exists, and every re-plan costs another plus a human round-trip. The skill refuses accordingly: work that fits one session gets no plan; a plan you can't confidently write to the end gets planned only to the confidence horizon; a second re-plan triggers a "just finish it top-tier?" checkpoint. Where it wins big: execution is deferred, headless, scheduled, or on cheap models — the plan is the only supervisor those sessions get.

## License

MIT — see repo [LICENSE](../../LICENSE).
