# project-routines

A personal marketplace of engineering **routines** — repeatable workflows you run on demand or wire up to a schedule. Built for [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) and structured after [`anthropics/claude-plugins-official`](https://github.com/anthropics/claude-plugins-official).

Everything ships in a single `routines` plugin. `retro` is a skill you run as `/routines:retro`; `code-simplifier` is a memory-backed agent you invoke with `@agent-routines:code-simplifier`.

## Repository layout

```
project-routines/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace manifest (lists the routines plugin)
├── plugins/
│   └── routines/                 # One plugin holding every routine
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── agents/
│       │   └── code-simplifier.md  # @agent-routines:code-simplifier (persistent memory)
│       └── skills/
│           └── retro/
│               └── SKILL.md         # /routines:retro
├── docs/
│   └── AUTHORING.md              # Conventions every routine follows
└── README.md
```

The marketplace manifest references the plugin with a relative `source` path (`"./plugins/routines"`). A routine can be a **skill** (a `SKILL.md` run as `/routines:<name>`) or an **agent** (an `agents/<name>.md` Claude delegates to). `code-simplifier` is an agent so it can keep persistent memory of what it has already processed.

## Routines

| Routine | Kind | Invoke | Trigger | What it does |
| --- | --- | --- | --- | --- |
| `code-simplifier` | agent | `@agent-routines:code-simplifier` | on demand · after edits · pre-commit / CI | Tightens recently modified code for clarity and maintainability while preserving behavior — removes AI slop and over-engineering, applies cosmetic fixes directly, flags structural refactors. Remembers the last commit it processed and runs incrementally on what changed since. |
| `retro` | skill | `/routines:retro` | weekly · scheduled | Read-only weekly retrospective over shipping cadence, test health, and progress, ending in 1–3 concrete next-week improvements. |

## Installation

Add this repo as a marketplace from inside Claude Code, then install the plugin:

```
/plugin marketplace add tracychen/project-routines
/plugin install routines@project-routines
```

Then run the retro skill by its namespaced command, or hand a task to the code-simplifier agent:

```
/routines:retro
@agent-routines:code-simplifier   # or: "use the code-simplifier agent on my recent changes"
```

The `add` argument is the repository path (`tracychen/project-routines`); the `@project-routines` suffix is the marketplace `name` from `marketplace.json`. Plugin skills are invoked as `/<plugin>:<skill>` and plugin agents as `@agent-<plugin>:<name>`.

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

Routines differ by what triggers them, so they automate differently:

- **Time-based** (e.g. `retro`) — its natural trigger is a clock. Automate it with a scheduled task or cron running it headlessly, weekly.
- **Event-based** (e.g. `code-simplifier`) — its natural trigger is "code just changed," not a time. Run it on demand, let it fire proactively in a session, or wire it into a pre-commit hook or CI step on each PR. Because it remembers the last commit it processed, each run is incremental — you don't pick a cadence.

Either way, each routine is written to be safe to run unattended: a deterministic scope, no destructive defaults, and clear output.

## Adding a routine

1. Add it to the `routines` plugin as either a **skill** (`plugins/routines/skills/<name>/SKILL.md` → `/routines:<name>`) or an **agent** (`plugins/routines/agents/<name>.md` → `@agent-routines:<name>`; use an agent when it needs isolated context or persistent memory).
2. If it needs its own dependencies or hooks, add a separate plugin under `plugins/<name>/` instead and register it in `.claude-plugin/marketplace.json` with a `"source": "./plugins/<name>"`.
3. Document how the routine runs (on demand and/or a schedule) in its `description`, and add a row to the table above.

See [`docs/AUTHORING.md`](docs/AUTHORING.md) for the conventions every routine follows.

## License

Released under the [MIT License](LICENSE). © 2026 Tracy Chen
