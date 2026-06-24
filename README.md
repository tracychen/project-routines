# project-routines

A marketplace of custom [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) plugins, named **`project-routines`**. Structured after [`anthropics/claude-plugins-official`](https://github.com/anthropics/claude-plugins-official).

## Repository layout

```
project-routines/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace manifest listing every plugin
├── plugins/
│   └── code-simplifier/          # One directory per plugin
│       ├── .claude-plugin/
│       │   └── plugin.json       # Plugin manifest (name, version, author)
│       └── agents/
│           └── code-simplifier.md  # Subagent definition
└── README.md
```

The marketplace manifest references first-party plugins (those living in this
repo) with a relative `source` path, e.g. `"./plugins/code-simplifier"`.

## Plugins

| Plugin | Category | Description |
| --- | --- | --- |
| `code-simplifier` | productivity | A subagent that simplifies and refines recently modified code for clarity, consistency, and maintainability while preserving functionality. |

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
/plugin marketplace add /Users/tracy/Development/projects/claude-plugins
```

## Adding a new plugin

1. Create `plugins/<name>/.claude-plugin/plugin.json` with the plugin's `name`,
   `version`, `description`, and `author`.
2. Add the plugin's components — `agents/`, `commands/`, `hooks/`, and/or an
   MCP `.mcp.json` — under `plugins/<name>/`.
3. Register it in `.claude-plugin/marketplace.json` by appending an entry to the
   `plugins` array with a `"source": "./plugins/<name>"`.

## License

Released under the [MIT License](LICENSE). © 2026 Tracy Chen
