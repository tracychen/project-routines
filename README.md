# project-routines

A personal marketplace of common **product and engineering routines** — reusable workflows packaged as [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) plugins and skills. Run them on demand, or wire them up to run automatically on a schedule or in response to events.

## Repository layout

```
project-routines/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace manifest listing every plugin
├── plugins/
│   ├── code-simplifier/          # One directory per plugin
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json       # Plugin manifest (name, version, author)
│   │   └── agents/
│   │       └── code-simplifier.md  # Subagent definition
│   └── retro/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── skills/
│           └── retro/
│               └── SKILL.md      # Skill definition
├── docs/
│   └── AUTHORING.md              # Conventions every routine follows
└── README.md
```

A plugin can bundle any mix of components: `agents/` (subagents), `skills/` or `commands/` (invokable slash commands), `hooks/` (event triggers), and an MCP `.mcp.json`. The marketplace manifest references first-party plugins (those living in this repo) with a relative `source` path, e.g. `"./plugins/code-simplifier"`.

## Plugins and skills

| Name | Type | Category | Description |
| --- | --- | --- | --- |
| `code-simplifier` | Plugin (agent) | productivity | Simplifies and refines recently modified code for clarity, consistency, and maintainability while preserving functionality — removing AI-generated slop and over-engineering, deferring to existing project conventions. |
| `retro` | Plugin (skill) | productivity | Weekly engineering retrospective: a read-only, schedulable review of shipping cadence, test health, and progress that ends in 1–3 concrete next-week improvements. |

## Installation

Add this repo as a marketplace from inside Claude Code, then install a plugin:

```
/plugin marketplace add tracychen/project-routines
/plugin install code-simplifier@project-routines
```

The `add` argument is the repository path (`tracychen/project-routines`); the
`@project-routines` suffix is the marketplace `name` from `marketplace.json`,
which is what namespaces every plugin installed from here.

To develop locally, point the marketplace at this checkout instead:

```
/plugin marketplace add /Users/tracy/Development/projects/project-routines
```

## Running and automating

Plugins and skills can be used two ways:

- **On demand** — run a skill or command directly, or let an agent run when its task comes up (for example, `code-simplifier` refines code right after it's written).
- **Automated** — trigger them without you in the loop:
  - **Event-driven**, via plugin **hooks** that fire on Claude Code events (e.g. after an edit, or before a commit).
  - **Periodic**, by running it headlessly from your own scheduler or CI (cron, GitHub Actions) or a scheduled task (e.g. a nightly dependency audit or a daily standup digest).

Design each plugin to be safe to run unattended: a deterministic scope, no destructive defaults, and clear output.

## Adding a plugin

1. Create `plugins/<name>/.claude-plugin/plugin.json` with the plugin's `name`,
   `version`, `description`, and `author`.
2. Add its components under `plugins/<name>/` — any mix of `agents/`, `skills/`,
   `commands/`, `hooks/`, and an MCP `.mcp.json`.
3. Register it in `.claude-plugin/marketplace.json` by appending an entry to the
   `plugins` array with a `"source": "./plugins/<name>"`.
4. If it's meant to run automatically, document its trigger (a hook event or a
   schedule) in the plugin's description so installers know how to wire it up.

See [`docs/AUTHORING.md`](docs/AUTHORING.md) for the conventions every routine in this marketplace follows.

## License

Released under the [MIT License](LICENSE). © 2026 Tracy Chen
