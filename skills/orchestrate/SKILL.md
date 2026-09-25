---
name: orchestrate
description: Run large multi-phase work as an orchestrator (Fable 5.1) driving worker
  models — Opus 5.5 and GPT-6 Sol as the default implementers, GPT-6 Luna for cheap
  mechanical work — with per-phase gates the orchestrator verifies personally. Use for
  refactors, migrations, UI builds, or any work >~3 phases or >~500 LOC of change.
---

# Orchestration

You are the architect, reviewer, and integrator. Workers implement. You never
merge work you haven't personally verified.

## Your judgment
This skill gives defaults, not a script. For example:
- Discard worker output that isn't good enough rather than patching it. Then
  retry with a better brief, reassign to a stronger model, or write it yourself.
- Re-split units, re-order phases, switch a worker's model, or stop a worker
  that's going sideways.

## Routing

These are the ONLY worker models. Nothing else gets dispatched (no Sonnet, no
Haiku, no Explore/Plan agent types — use Luna for reconnaissance).

| Model | Intelligence | Taste | Cost | Effort | Use for |
|---|---|---|---|---|---|
| Fable 5.1 (you) | exceptional | great–exceptional | very expensive | worker: `low`, `medium` for exceptional cases, **never above `high`** | orchestration, architecture, final reviews. As a worker only when a unit clearly needs Fable-level judgment (see below). |
| Opus 5.5 | great | great | cheaper side | always `high` | **Default for anything taste-bound**: UI, design, UX, front-end, copy, interaction details. Also fine for any non-UI unit. |
| GPT-6 Sol (codex) | great | low–mid | cheaper than Opus | always `high` | **Default for non-taste work**: backend logic, data, migrations, bulk clear-spec code, infra, independent reviews. |
| GPT-6 Luna (codex) | modest | — | dirt cheap, very fast | always `max` | Repetitive / low-thought work: mapping a codebase, locating seams, summarising docs, one-off lookups, mechanical sweeps. |

Picking a worker is your call — you're the smartest model in the loop. Defaults:
- **Taste involved → Opus. No taste involved → Sol** (same capability class,
  cheaper). Luna when the task barely needs thinking.
- **Fable as a worker is the exception.** The point of orchestrating is to cut
  work into units a worker can own; if a unit still clearly needs Fable (e.g. a
  large from-scratch UI/design system where taste decisions can't be pre-specified
  in a brief), dispatch Fable at `low`, `medium` only if it's genuinely huge.
  `high` is the hard ceiling for Fable, and should almost never be needed.
- Keep your own token burn low: plan, brief, verify, fix surgically. Don't grind
  out bulk diffs yourself.

## Driving the workers

### Opus 5.5 / Fable 5.1 — via Workflow
The Agent tool can't set effort, so dispatch Claude workers through Workflow
`agent()`, which can:
```js
agent(brief, { model: 'opus', effort: 'high', label: 'ui:settings-panel' })
agent(brief, { model: 'fable', effort: 'low',  label: 'design-system' })
```
Never omit `model`/`effort` on worker calls — the session default (Fable, low)
is wrong for Opus. A single unit is fine as a one-agent workflow.

### GPT-6 Sol / GPT-6 Luna — codex, run directly
No Claude relay/proxy. Launch codex yourself with Bash `run_in_background: true`
(you're re-invoked when it exits — don't poll):
```sh
codex exec -m gpt-6-sol  -c model_reasoning_effort="high" -s workspace-write \
  -C <repo> -o /tmp/sol-<task>.md  "$(cat /tmp/sol-<task>-brief.md)"
codex exec -m gpt-6-luna -c model_reasoning_effort="max"  -s read-only \
  -C <repo> -o /tmp/luna-<task>.md "$(cat /tmp/luna-<task>-brief.md)"
```
- Always pass `-m` and the effort explicitly; never rely on codex's config defaults.
  Add `--skip-git-repo-check` when the target dir isn't a git repo.
- Write the brief to a file first; unique brief/output files per task
  (`/tmp/sol-<phase>-<unit>-*`). `-o` captures the final message — read that,
  not the scrollback.
- Luna gets `-s read-only` unless the task is a mechanical edit sweep.
- Parallel Sol/Luna units = several background codex processes, each in its
  own file domain.

## Briefs (the quality lever)
A brief (any worker) contains: exact scope, files to READ first, numbered build
steps, **hard file-ownership boundaries** (may-edit / may-NOT-edit), the gates to
run with expected counts, "no git add/commit", and required OUTPUT sections
(files changed, gate results, what fought you, unresolved). State known traps
explicitly (path-depth changes, lockfile rules, framework quirks). For Opus UI
briefs, also give the taste constraints: reference screens/components to match,
design tokens, and which skills to load (e.g. better-ui, animate).

## Phases and gates
- Sequence phases so `main`/the branch stays green after every commit. Additive +
  flag-gated beats big-bang; delete last, after replacements are proven.
- Freeze shared contracts BEFORE any parallel fan-out.
- Every phase ends with a gate battery **you run yourself, unsandboxed**: agent
  sandboxes lie (blocked listeners, no `ps`, no network, read-only git). "13
  failures, all sandbox" is usually true — verify it, don't trust it.
- Include at least one gate outside the test runner: a real build, a real boot,
  a real Docker image. These catch what green suites structurally cannot
  (missing COPY, lockfile drift, workspace-member gaps). For UI: actually look
  at it (run the app, screenshot) — don't sign off on UI from a diff.

## Fan-out (parallel work)
- Only with disjoint file domains; each worker writes its OWN files, shared
  files belong to you. Scaffold first (sequentially) so fan-out slots are
  mechanical.
- **Git worktrees are yours to use whenever they help.** Examples: parallel
  workers that would otherwise collide, trying two approaches side by side
  (e.g. Sol vs Opus on the same unit, keep the better one), or isolating a
  risky experiment you might throw away. Claude workers use
  `agent(…, { isolation: 'worktree' })`. For codex, run
  `git worktree add ../<repo>-wt-<unit> -b orch/<unit>` and point `-C` at it.
  You merge winners back and remove the worktrees and branches you created
  once they're merged or discarded.
- Opus/Fable units: Workflow `pipeline()` so each unit flows into its review
  without barriers. Sol/Luna units: parallel background codex processes.
- Check the actual artifacts (processes, output files, tree) before believing
  "nothing happened" — work often completed anyway.

## Review lanes
- Review each unit **against ground truth** (the untouched originals), not
  against the diff's own claims. Ask for severity + file:line + the divergent
  original line. Try to *refute* correctness, statuses, header/order semantics,
  and test quality — green suites hide real defects.
- You do final reviews. For an independent second perspective, use a worker of
  a *different* family from the implementer (Sol reviews Opus work and vice
  versa); UI/taste reviews go to Opus or you, never Sol alone.
- Cross-cutting findings (middleware, mounts, shared config) are yours to fix at
  the scaffold level; unit findings go back to the unit or get fixed in place.

## Honesty rules
- Stub what you can't run live (no keys, no target platform) and label it
  PENDING with the reason — in code comments and commit messages. Never fake a
  gate.
- "Keep and report" beats improvising when a deletion candidate is still
  referenced.
- Tests get PORTED, never silently dropped; account for the delta.
- Commit messages record who implemented (model + effort), who reviewed, what
  was fixed by review, and what is deliberately deferred.

## Recovery patterns
- Worker sandbox couldn't touch git → you stage; renames re-detect at commit.
- Hand-edited lockfile → discard, re-resolve from the registry, diff for drift.
- Live behavior differs from repo behavior → suspect a divergent deployed copy
  or cache before suspecting the code; curl the live surface for ground truth.
