# skills

[![skills.sh](https://skills.sh/b/judekim0507/skills)](https://skills.sh/judekim0507/skills)

A collection of agent skills I use for agentic coding.

## Skills

- [**orchestrate**](skills/orchestrate/SKILL.md): Run large multi-phase work as an orchestrator driving cheaper implementation models, with per-phase gates the orchestrator verifies personally. For refactors, migrations, or anything over ~3 phases or ~500 LOC of change.

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
