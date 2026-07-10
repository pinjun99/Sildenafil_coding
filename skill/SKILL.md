---
name: orchestrator
description: Use when a coding or research task is wide — several independent slices that could run in parallel — and the session is running a high-tier model. Plan with the top-tier model, define the seams, fan out bounded work to cheaper worker agents via self-contained handoff packets, then own integration, verification, and final review. Skip for tiny fixes, tightly coupled edits, or single-thread debugging.
---

# Efficient Orchestrator

Spend top-tier tokens only where judgment lives: decomposition, tradeoffs, conflict resolution, integration, review. Delegate every bounded, parallelizable slice to cheaper workers. This skill is runtime-agnostic — it works in Claude Code, Codex CLI, or any agent runtime with subagents (see Runtime adapters at the end).

## 1. Go / no-go gate — run this before anything else

Ask: **"Could three people do this at the same time without talking to each other?"**

- **Yes** → orchestrate. Continue with this skill.
- **No** → do the work yourself in this session. Declining to fan out is a correct use of this skill, not a failure of it.

Never orchestrate:

- Tiny or single-file fixes — the briefing costs more than the work
- Tightly coupled edits, where each step depends on the previous step's outcome
- Judgment-sensitive debugging — one detective following one trail
- Work whose main difficulty is deciding *what* to do rather than doing it

Size is not the test; **width** is. A large refactor touching 40 independent files is wide (orchestrate). A hard algorithm in one file is narrow (keep it), no matter how big.

## 2. Model hierarchy

The highest tier available owns the session and orchestrates. Workers run one to two tiers below. Never delegate judgment downward.

| Role | Tier rule | Anthropic example | OpenAI example |
|---|---|---|---|
| Orchestrator | Highest tier available | Fable 5 (or Opus) | GPT-5.6 Sol |
| Code / test workers | One tier down; mid-tier floor | Opus or Sonnet | GPT-5.6 Terra |
| Scan / summarize workers | Lowest tier the risk allows | Sonnet | GPT-5.6 Luna |

Rules, not pins:

- These are policies. Choose each worker's tier per slice, based on how much judgment the slice carries — and **promote a worker's tier** when a slice turns out to need more thinking than expected.
- Code that will be kept has a **mid-tier floor**: never hand production edits to the weakest tier.
- Model names age; the tier logic doesn't. Substitute your provider's current lineup.
- If the session itself is running a mid-tier model, this skill still applies — orchestrate with what you have and delegate down from there. But do not run the orchestrator role on the weakest tier.

## 3. What the orchestrator never delegates

- Decomposing ambiguous work into clean parallel slices
- Architecture, product, security, and safety tradeoffs
- Defining the seams (section 4, step 2)
- Reading conflicting worker reports and deciding what matters
- Integration across slices and final review of the combined result

## 4. Delegation sequence

1. **Spot the token risk.** Large repos, long logs, bulk docs, repetitive edits, multi-file test runs — anything where reading or grinding would burn premium tokens without needing premium judgment.
2. **Define the seams first.** Before any fan-out, write down the contracts between slices: interfaces, shared types, naming, data shapes, and file ownership (which worker may write which files, with no overlaps). Every packet carries the seams that touch it. Slices that fit at merge time are designed, not hoped for.
3. **Slice into independent packets.** Each slice must be completable without talking to another worker. If two slices need a conversation, merge them or move the shared decision up into the seams.
4. **Write a handoff packet per slice** (section 5). Assume the worker knows nothing: no conversation history, no implied context.
5. **Fan out.** Launch workers in parallel; run long workers in the background. Match each worker's tier to its slice (section 2).
6. **Vet, integrate, verify** (section 7). The orchestrator owns the merged result.

## 5. Handoff packet template

Every delegated task uses this. Fill every line; "N/A" is an answer, silence is not.

```
OBJECTIVE
One sentence: what done looks like.

CONTEXT
Repo path: <path>
Starting files: <files worth reading first, with why>
Background the worker cannot infer: <1-3 sentences>

SCOPE
In scope: <exact work>
Out of scope: <the tempting adjacent work this worker must NOT do>

SEAMS
Interfaces/contracts you must conform to: <types, endpoints, naming, data shapes>
Files you own: <globs/paths — write nowhere else>

EVIDENCE REQUIRED IN YOUR REPORT
- Files touched or found, with line references
- Commands you ran and their real output
- Diffs for every change
- Failures and anything you are uncertain about — uncertainty is a required
  section of the report, not an embarrassment to hide

VERIFICATION
Command(s): <exact command>
Success criteria: <what output means pass>

STOP CONDITIONS
Stop and report immediately — do not improvise — if:
- The seams above conflict with what you find in the code
- Verification fails twice for the same reason
- Completing the objective requires touching out-of-scope files
```

The verification command must cover the *whole* scope — a check that tests only part of the scope lets the untested part fail silently.

## 6. Worker conduct rules

Paste this block verbatim at the end of every packet:

```
CONDUCT
- Do exactly the scope; no drive-by fixes, no scope creep.
- When unsure, report the uncertainty instead of guessing.
- Never claim success without running the verification command and
  quoting its real output.
- A useful failure report beats a fake success report.
```

## 7. Vetting — reports are leads, not facts

Workers ran cold and cheap; their reports are testimony, not evidence. Before relying on any of it:

- Re-open the key files a report points at; spot-check its line references
- Diff-review every change against the packet's scope and seams
- **Re-run builds and tests yourself** at the top level. "Worker said tests pass" is not a passing test.
- Where two reports conflict, resolve it yourself — never ask one worker to referee another

Nothing merges, ships, or gets reported to the user as done until the orchestrator has verified it first-hand.

## 8. Phase slicing — by feature, never by layer

When work spans multiple phases or sessions, every phase must end with something runnable.

- **By layer (wrong):** phase 1 = all data, phase 2 = all logic, phase 3 = all UI. Nothing runs until the end; early mistakes surface last.
- **By feature (right):** phase 1 = the smallest working end-to-end version, each later phase = one feature added to a running system. You can stop after any phase and still have a working thing.

## 9. Scenarios

| Work | Workers do | Orchestrator keeps |
|---|---|---|
| Research | Scan repos/docs, inventory usages, summarize with file:line refs | Judging which evidence matters; the conclusion |
| Coding | Bounded edits inside owned files, conforming to seams | Seam design, integration, diff review |
| Testing | Run targeted suites, reduce logs to failures + context | Diagnostic weight; what a failure means |
| Debugging | Reproduce, bisect, cluster occurrences | The diagnosis itself |

## 10. Runtime adapters

- **Claude Code:** spawn workers with the Agent (subagent) tool, one per packet, setting the model per spawn to the tier chosen in section 2. Launch independent workers in the same message so they run in parallel; use background mode for long workers. The subagent's final report returns to the orchestrator — vet it per section 7.
- **Codex CLI:** run each packet as a `codex exec` subprocess with a lower-tier model flag (e.g. `codex exec -m <cheaper-model> "<packet>"`), in the background where possible; collect stdout as the report.
- **Any other agent runtime:** use its native subagent/task mechanism with per-task model selection. If none exists, execute the packets sequentially, each in a fresh context — the packets are self-contained by construction, so they lose nothing but parallelism.

## Cost expectations — honest version

Fan-out saves money only when slices are genuinely independent and bulky; savings of 2–5x on cost and 2–4x on wall-clock time are realistic for wide workloads, and zero or negative for narrow ones. That is why section 1 comes first.
