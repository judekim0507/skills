# skills

[![skills.sh](https://skills.sh/b/judekim0507/skills)](https://skills.sh/judekim0507/skills)

A collection of agent skills I use for agentic coding.

## Skills

### [orchestrate](skills/orchestrate/SKILL.md)

Run large multi-phase work with one expensive, high-taste model as the orchestrator and cheaper models as the workforce. Built to be driven by **Claude Fable 5**: Fable is too expensive to grind out bulk diffs, but it's an awesome orchestrator — so it plans the phases, writes the briefs, and personally verifies every gate, while **GPT-5.6 Sol** (via the codex CLI) does the bulk implementation and **Claude Opus** handles reviews and user-facing UI work.

The core ideas:

- The orchestrator is architect, reviewer, and integrator — it never merges work it hasn't personally verified.
- Work is routed by taste and cost: user-facing work needs a high-taste model in the loop, bulk clear-spec work goes to Sol, reviews go to Fable or Opus.
- Every phase ends with a gate battery the orchestrator runs itself, outside agent sandboxes, including at least one real-world gate (a real build, a real boot) that green test suites can't fake.
- Parallel fan-out only over disjoint file domains, with contracts frozen first.

Use it for refactors, migrations, or anything over ~3 phases or ~500 LOC of change. Loosely inspired by [Theo](https://t3.gg)'s takes on model routing.

## Install

### As a Claude Code plugin

Installs every skill in this repository together and updates in place. Run these inside Claude Code:

```text
/plugin marketplace add judekim0507/skills
/plugin install workflow@jude
```

### With the skills CLI

Works in Claude Code, Codex and other agents. You can choose which skills to install or install all of them.

```bash
npx skills add judekim0507/skills
```

```bash
npx skills add judekim0507/skills --skill '*'
```

## Use

Invoke the skill by name. As a plugin, skills are namespaced with the `workflow:` prefix:

```text
/workflow:orchestrate
```

Installed with the skills CLI:

```text
/orchestrate
```

## License

[MIT](LICENSE)
