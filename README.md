# project-routines

A personal marketplace of engineering **routines** — repeatable workflows you run on demand or wire up to a schedule. Built for [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) and structured after [`anthropics/claude-plugins-official`](https://github.com/anthropics/claude-plugins-official).

Everything ships in a single `routines` plugin. Both routines are memory-backed agents you invoke with `@agent-routines:<name>` — `code-simplifier` and `retro` (which fans read-only gathering out to subagents).

## Repository layout

```
project-routines/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace manifest (lists the routines plugin)
├── plugins/
│   └── routines/                 # One plugin holding every routine
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── agents/
│           ├── code-simplifier.md  # @agent-routines:code-simplifier (persistent memory)
│           ├── retro.md            # @agent-routines:retro (memory; spawns subagents)
│           └── retro-analyst.md    # read-only subagent the retro agent delegates to
├── docs/
│   └── AUTHORING.md              # Conventions every routine follows
└── README.md
```

The marketplace manifest references the plugin with a relative `source` path (`"./plugins/routines"`). Both routines are **agents** (`agents/<name>.md`) so they get an isolated context, persistent memory across runs, and — in retro's case — the ability to fan work out to subagents. `retro-analyst` is a read-only helper the `retro` agent delegates to; it isn't invoked directly.

## Routines

| Routine | Kind | Invoke | Trigger | What it does |
| --- | --- | --- | --- | --- |
| `code-simplifier` | agent | `@agent-routines:code-simplifier` | on demand · after edits · pre-commit / CI | Tightens recently modified code for clarity and maintainability while preserving behavior — removes AI slop and over-engineering, applies cosmetic fixes directly, flags structural refactors. Remembers the last commit it processed and runs incrementally on what changed since. |
| `retro` | agent | `@agent-routines:retro` | weekly · scheduled | Weekly retrospective over shipping cadence, test health, and progress, ending in 1–3 concrete next-week improvements. Fans read-only gathering out to `retro-analyst` subagents and remembers prior retros to track trends. |

## Installation

Add this repo as a marketplace from inside Claude Code, then install the plugin:

```
/plugin marketplace add tracychen/project-routines
/plugin install routines@project-routines
```

Then hand a task to either agent:

```
@agent-routines:code-simplifier   # or: "use the code-simplifier agent on my recent changes"
@agent-routines:retro             # or: "run a retro for this week"
```

The `add` argument is the repository path (`tracychen/project-routines`); the `@project-routines` suffix is the marketplace `name` from `marketplace.json`. Plugin agents are invoked as `@agent-<plugin>:<name>`.

To develop locally, point the marketplace at this checkout instead:

```
/plugin marketplace add /Users/tracy/Development/projects/project-routines
```

## Updating

Third-party marketplaces don't auto-update by default. To pull newer versions, run `/plugin marketplace update project-routines` then `/reload-plugins` — or enable auto-update in `/plugin` → **Marketplaces**. New versions ship when a plugin's `version` in `plugin.json` is bumped.

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

1. Add it to the `routines` plugin — usually an **agent** (`plugins/routines/agents/<name>.md` → `@agent-routines:<name>`) so it gets isolated context, memory, and subagents; or a **skill** (`plugins/routines/skills/<name>/SKILL.md` → `/routines:<name>`) for a lighter checklist-style routine.
2. If it needs its own dependencies or hooks, add a separate plugin under `plugins/<name>/` instead and register it in `.claude-plugin/marketplace.json` with a `"source": "./plugins/<name>"`.
3. Document how the routine runs (on demand and/or a schedule) in its `description`, and add a row to the table above.

See [`docs/AUTHORING.md`](docs/AUTHORING.md) for the conventions every routine follows.

## License

Released under the [MIT License](LICENSE). © 2026 Tracy Chen
