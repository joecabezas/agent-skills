# agent-skills

Portable [Agent Skills](https://agentskills.io) — the open `SKILL.md` format
used by Claude, Cursor, Copilot, Gemini CLI, and 30+ other AI coding agents.
A skill is just a folder with a `SKILL.md` file: name, description, and
instructions an agent loads on demand when a task matches.

## Skills in this repo

| Skill | What it does |
|---|---|
| [`find-products`](skills/find-products/) | Rigorous, opinionated product/vendor comparisons — writes one file per candidate plus a README with a comparison table and a real recommendation. Works for hardware, SaaS tools, service providers, anything you'd shop for. |

## Install

### With `npx skills` (works with most agents)

```bash
npx skills add https://github.com/joecabezas/agent-skills --skill find-products
```

### With Claude Code's native plugin system

```
/plugin marketplace add joecabezas/agent-skills
/plugin install find-products@joecabezas-agent-skills
```

### Manually

Copy `skills/find-products/SKILL.md` into your agent's skills directory
(e.g. `.claude/skills/find-products/SKILL.md` for a Claude Code project,
or wherever your agent looks for skills — see the [client list](https://agentskills.io/clients)
for exact paths per tool).

## Structure

```
agent-skills/
├── .claude-plugin/
│   └── marketplace.json      # Claude Code native plugin discovery
├── skills/
│   └── find-products/
│       └── SKILL.md
└── README.md
```

Each skill is self-contained: everything an agent needs lives in its own
`skills/<name>/` folder as `SKILL.md`, following the
[Agent Skills specification](https://agentskills.io/specification).

## Contributing

New skills welcome. Each one gets its own folder under `skills/` with a
`SKILL.md` (kebab-case `name` matching the folder name, `description` that
covers both what it does and when to use it), and an entry added to
`.claude-plugin/marketplace.json` plus the table above.
