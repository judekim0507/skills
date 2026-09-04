---
name: orchestrate
description: Run large multi-phase work as an orchestrator driving cheaper implementation
  models (GPT-5.6 Sol via codex — almost exclusively), with per-phase gates the
  orchestrator verifies personally. Use for refactors, migrations, or any work >~3
  phases or >~500 LOC of change.
---

# Orchestration

You are the architect, reviewer, and integrator. Subagents implement. You never
merge work you haven't personally verified.

## Routing

| Model | Intelligence | Taste | Cost | Use for |
|---|---|---|---|---|
| Fable 5 (you) | ~9–10 | ~9–10 | high | architecture, end-to-end judgment, final reviews, orchestration |
| GPT-5.6 Sol (via codex CLI) | ~8–9 | ~7 | cheap | THE implementer for non-UI work: bulk and clear-spec code, migrations, data analysis, computer use, independent reviews. **Never UI.** |
| GPT-5.6 Luna (via codex CLI, `codex exec -m gpt-5.6-luna -c model_reasoning_effort="xhigh"` — always max effort) | good | — | very cheap, fast | small tasks: exploring, mapping the codebase, locating seams, summarising docs, one-off lookups. Use it instead of Explore/Sonnet subagents for reconnaissance. |
| Opus 5 | — | — | very high | **NEVER as a subagent** (Jude, 2026-09-04: far too expensive and weaker than Sonnet 5). Do not dispatch it for anything. |
| Sonnet 5 | medium | medium | medium | proxy relays / workflow glue only. Never implements. |

Rules of thumb: **Sol does almost all non-UI work; Luna at max effort does the
small stuff (exploration, mapping). UI work is Fable's own — never handed to
Sol or any subagent.** **Reviews** →
Fable, optionally Sol as an independent second perspective. Keep Fable's own
token burn low: it plans, briefs, verifies, and fixes surgically — it does not
grind out bulk diffs.

## Driving Sol through the proxy
Sonnet subagent relays the brief to `codex exec` **word for word** and returns the
exact output. Mechanics that matter:
- Launch codex detached (`nohup … & echo $! > /tmp/<task>.pid`); proxies often
  return early — never depend on them for the wait.
- You own the monitor: wait for the pid to EXIST (race), then to exit. Poll the
  named pid, never a broad `pgrep` (parallel runs collide).
- Unique prompt/log/pid files per task (`/tmp/sol-<phase>-*`).

## Briefs (the quality lever)
A Sol brief contains: exact scope, files to READ first, numbered build steps,
**hard file-ownership boundaries** (may-edit / may-NOT-edit), the gates to run
with expected counts, "no git add/commit", and required OUTPUT sections
(files changed, gate results, what fought you, unresolved). State known traps
explicitly (path-depth changes, lockfile rules, framework quirks).

## Phases and gates
- Sequence phases so `main`/the branch stays green after every commit. Additive +
  flag-gated beats big-bang; delete last, after replacements are proven.
- Freeze shared contracts BEFORE any parallel fan-out.
- Every phase ends with a gate battery **you run yourself, unsandboxed**: agent
  sandboxes lie (blocked listeners, no `ps`, no network, read-only git). "13
  failures, all sandbox" is usually true — verify it, don't trust it.
- Include at least one gate outside the test runner: a real build, a real boot,
  a real Docker image. These catch what green suites structurally cannot
  (missing COPY, lockfile drift, workspace-member gaps).

## Fan-out (parallel work)
- Agent vs Workflow: your judgment — you're capable of picking. Bias: for big
  refactors prefer Workflow (`pipeline()` fan-out is quicker than serial Agent
  calls); a single unit or one-off review is fine as a plain subagent.
- Only with disjoint file domains; each agent writes its OWN files, shared files
  belong to you. Scaffold first (sequentially) so fan-out slots are mechanical.
- Use Workflow `pipeline()` so each unit flows into its review without barriers.
- Expect proxy early-returns: check the actual artifacts (processes, logs, tree)
  before believing "nothing happened" — detached work often completed anyway.

## Review lanes
- Fable (or Sol) reviews each unit **against ground truth** (the untouched originals), not
  against the diff's own claims. Ask for severity + file:line + the divergent
  original line. Have it try to *refute* correctness, statuses, header/order
  semantics, and test quality — green suites hide real defects.
- Cross-cutting findings (middleware, mounts, shared config) are yours to fix at
  the scaffold level; unit findings go back to the unit or get fixed in place.

## Honesty rules
- Stub what you can't run live (no keys, no target platform) and label it
  PENDING with the reason — in code comments and commit messages. Never fake a
  gate.
- "Keep and report" beats improvising when a deletion candidate is still
  referenced.
- Tests get PORTED, never silently dropped; account for the delta.
- Commit messages record who implemented, who reviewed, what was fixed by review,
  and what is deliberately deferred.

## Recovery patterns
- Agent sandbox couldn't touch git → you stage; renames re-detect at commit.
- Hand-edited lockfile → discard, re-resolve from the registry, diff for drift.
- Live behavior differs from repo behavior → suspect a divergent deployed copy
  or cache before suspecting the code; curl the live surface for ground truth.
