# Authoring routines

Conventions every plugin in this marketplace follows. A *routine* is a plugin that packages a repeatable product or engineering workflow. Keep them small, composable, and safe to automate.

## Principles

1. **Single responsibility.** One routine does one job. If you're tempted to add a second mindset — say, security review to a code simplifier — make a new plugin instead. Mixing mindsets produces mediocre results on both.
2. **Compose via artifacts.** Prefer routines that read and write durable artifacts (a plan doc, a review, a retro file) so they chain: one routine's output is the next one's input. Name and locate those artifacts predictably.
3. **Bounded scope.** Keep each run proportional to its trigger. A cleanup shouldn't become a rewrite; a review shouldn't refactor. If a worthwhile change exceeds the scope, flag it — don't do it.
4. **Auto-fix the obvious, flag the judgment calls.** Apply changes that aren't judgment calls directly (dead code, unused imports, formatting). Anything structural or behavior-affecting: propose it with rationale and before/after when a human is in the loop; flag it only, and change nothing, when running unattended.
5. **Zero-noise, evidence-backed.** Require confidence before asserting a finding, verify it, and attach concrete evidence — the line, the exploit scenario, the metric. High signal beats high volume.
6. **Don't guess — the Confusion Protocol.** On an ambiguous architectural or intent question, stop and ask rather than assume. A wrong guess compounds downstream.
7. **Safe to run unattended.** Routines can be wired to schedules or triggers, so design for it: deterministic scope, no destructive defaults, read-only where possible, and clear output. State plainly whether the routine edits code.

## Craft patterns

Reusable techniques for building reliable routines:

- **Circuit breaker.** Bound retries. If a routine can't make progress after a few attempts (e.g. 3 failed fixes), stop and report rather than thrash. Corollary: no fixes without investigation.
- **Preview by default.** Any routine that changes things should be able to show its plan before acting. Make dry-run the default for unattended runs, and require an explicit go-ahead to mutate.
- **Plan-vs-outcome boomerang.** When a routine estimates something (scores, effort, risk), pair it with a later routine that measures the real outcome and diffs the two, so reality checks the plan.
- **Independent second opinion.** For high-stakes review, verify with a separate pass — ideally a different model — and surface where the two agree versus diverge.
- **Zero-noise with an exclusion list.** Reviews and audits should carry an explicit list of known false positives to suppress, plus a confidence threshold, so output stays high-signal.
- **Diataxis for docs.** If a routine writes documentation, sort it into the four Diataxis modes — reference, how-to, tutorial, explanation — and track coverage so gaps are visible.

## Choosing a component type

A plugin can bundle any mix of these. Pick the lightest one that fits:

- **Skill** (`skills/<name>/SKILL.md`) — a model-invocable routine triggered by its description or run as `/<plugin>:<name>`, able to carry supporting files. Best for methodologies and multi-step workflows, and the default here (e.g. `code-simplifier`, `retro`).
- **Agent** (`agents/<name>.md`) — a specialized worker Claude delegates to in its own context; invoked with `@agent-<plugin>:<name>` or by naming it in a prompt. Best for a focused, tool-restricted job that should run in an isolated context.
- **Command** (`commands/<name>.md`) — a thin, deterministic slash command. Best for a simple, fixed action.
- **Hook** (`hooks/`) — an event trigger; use it to run a routine automatically on a Claude Code event.

## Writing the description

The `description` *is* the trigger. Write it for how people actually ask: include natural-language phrases ("run a retro", "review my changes"), the situation it applies to, and the boundary — what it will and won't do. Claude uses the description to decide when to delegate, so be specific rather than generic.

## Making a routine automatable

State the trigger explicitly:

- **Periodic** — note the cadence (e.g. weekly) and that it's safe headless; wire it to a scheduled task or a cron job that invokes the agent non-interactively.
- **Event-driven** — ship a `hooks/` entry that fires on the relevant Claude Code event.

Design the routine to produce the same result whether a human or a scheduler runs it, and to degrade to flag-only — no structural changes — when unattended.

## Adding a plugin (checklist)

1. `plugins/<name>/.claude-plugin/plugin.json` — `name`, `version`, `description`, `author`.
2. Components under `plugins/<name>/` — any mix of `agents/`, `skills/`, `commands/`, `hooks/`, and an MCP `.mcp.json`.
3. Register it in `.claude-plugin/marketplace.json` with `"source": "./plugins/<name>"`.
4. Add a row to the README table and document any trigger or schedule.
5. Validate: the JSON parses, frontmatter carries `name` + `description`, and the `source` path resolves.
