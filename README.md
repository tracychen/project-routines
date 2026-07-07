# project-routines

A personal marketplace of engineering **routines** — repeatable workflows you run on demand or wire up to a schedule. Built for [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) and structured after [`anthropics/claude-plugins-official`](https://github.com/anthropics/claude-plugins-official).

The routines ship as skills inside a single `routines` plugin, so each one is invoked as `/routines:<name>`.

## Repository layout

```
project-routines/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace manifest (lists the routines plugin)
├── plugins/
│   └── routines/                 # One plugin holding every routine
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── skills/
│           ├── code-simplifier/
│           │   └── SKILL.md       # /routines:code-simplifier
│           └── retro/
│               └── SKILL.md       # /routines:retro
├── docs/
│   └── AUTHORING.md              # Conventions every routine follows
└── README.md
```

The marketplace manifest references the plugin with a relative `source` path (`"./plugins/routines"`). Each routine is a skill — a `SKILL.md` with `name` + `description` frontmatter that Claude invokes by description or that you run directly.

## Routines

| Routine | Command | When it runs | What it does |
| --- | --- | --- | --- |
| `code-simplifier` | `/routines:code-simplifier` | on demand · proactively · scheduled sweep | Tightens recently modified code for clarity and maintainability while preserving behavior — removes AI slop and over-engineering, defers to project conventions, applies cosmetic fixes directly and flags structural refactors. |
| `retro` | `/routines:retro` | weekly · scheduled | Read-only weekly retrospective over shipping cadence, test health, and progress, ending in 1–3 concrete next-week improvements. |

## Installation

Add this repo as a marketplace from inside Claude Code, then install the plugin:

```
/plugin marketplace add tracychen/project-routines
/plugin install routines@project-routines
```

Then run a routine by its namespaced command:

```
/routines:code-simplifier
/routines:retro
```

The `add` argument is the repository path (`tracychen/project-routines`); the `@project-routines` suffix is the marketplace `name` from `marketplace.json`. Plugin skills are always invoked as `/<plugin>:<skill>`, which is why the commands read `/routines:…`.

To develop locally, point the marketplace at this checkout instead:

```
/plugin marketplace add /Users/tracy/Development/projects/project-routines
```

## Team setup

To give everyone on a repo these routines automatically, commit a `.claude/settings.json` to *that* repo. When teammates trust the folder, Claude Code prompts them to add the marketplace and install the enabled plugin:

```json
{
  "extraKnownMarketplaces": {
    "project-routines": {
      "source": { "source": "github", "repo": "tracychen/project-routines" }
    }
  },
  "enabledPlugins": {
    "routines@project-routines": true
  }
}
```

`extraKnownMarketplaces` registers the marketplace by source; `enabledPlugins` turns the plugin on.

## Running and automating

Each routine works two ways:

- **On demand** — run its command (`/routines:code-simplifier`, `/routines:retro`), or let Claude invoke it when the task matches its description.
- **Automated** — run it headlessly from your own scheduler or CI (cron, GitHub Actions) or a scheduled task — for example a weekly `retro` or a nightly `code-simplifier` sweep. A routine can also ship a `hooks/` entry to fire on Claude Code events.

Each routine is written to be safe to run unattended: a deterministic scope, no destructive defaults, and clear output.

## Adding a routine

1. Create a skill directory `plugins/routines/skills/<name>/SKILL.md` with `name` and `description` frontmatter and the procedure in the body. It becomes `/routines:<name>`.
2. If it needs its own tools, dependencies, or hooks, add a separate plugin under `plugins/<name>/` instead and register it in `.claude-plugin/marketplace.json` with a `"source": "./plugins/<name>"`.
3. Document how the routine runs (on demand and/or a schedule) in its `description`, and add a row to the table above.

See [`docs/AUTHORING.md`](docs/AUTHORING.md) for the conventions every routine follows.

## License

Released under the [MIT License](LICENSE). © 2026 Tracy Chen
