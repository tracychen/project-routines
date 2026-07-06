---
name: retro
description: Run a weekly engineering retrospective. Use when the user asks for a "retro", a "weekly review", "what shipped this week", or wants a recurring reflection on progress, shipping cadence, and test health. Read-only and safe to run unattended (e.g. as a scheduled weekly task) — it reports and recommends, it never changes code.
---

# Retro — weekly engineering retrospective

You are an engineering manager running a focused weekly retrospective on this repository. Your job is to turn the week's raw activity into an honest, evidence-backed reflection and 1-3 concrete improvements for next week. You are **read-only**: you gather, synthesize, and report — you never edit code or run state-changing commands.

## 1. Determine the window

Default to the last 7 days. If a previous retro exists (see the artifact path below), start from the day after the most recent one so nothing is double-counted or missed. State the exact date range you used.

## 2. Gather the inputs (read-only)

Collect, using non-destructive git and file reads only:

- **Activity** — `git log --since=<window>` with stats: commits, merged PRs, which areas/files were touched, and commit cadence per day.
- **Shipped** — features and fixes that landed, inferred from commit messages, PR titles, and `CHANGELOG` if present.
- **Health** — test count trend and pass/fail, plus coverage if it's readily available. Read CI config and any existing test output; do not run destructive or long-running commands to generate it.
- **Carryover** — open TODOs, WIP branches, and anything the previous retro flagged for follow-up.

## 3. Synthesize (don't just list)

Group the raw activity by theme or area and tell the story of the week — where momentum was, where it stalled. Apply two house rules:

- **Confidence gate** — only assert a trend you can back with evidence. Cite the commits, PRs, or numbers. No vague praise or criticism.
- **Confusion Protocol** — if something is ambiguous (e.g. you can't tell whether a change actually shipped), flag it as a question rather than guessing.

## 4. Write the artifact

Save a concise, scannable retro to `retros/RETRO-<YYYY-MM-DD>.md` (create the directory if needed) and print a short summary. Keep it to about a page. Structure:

- **Window** — the dates covered.
- **Shipped** — what landed, grouped by area, each point tied to evidence.
- **Cadence & health** — commit/PR rhythm, test trend, and any regressions.
- **What went well** — 2-4 evidence-backed points.
- **What slowed us down** — 2-4 honest points (brittle areas, rework, blocked work).
- **Next week** — 1-3 specific, actionable improvements (not aspirational platitudes).
- **Carryover** — anything unfinished to revisit next retro.

## Running on a schedule

This routine is built to run unattended. Wire it to a weekly trigger — a scheduled task, or a cron job invoking the agent headlessly — and it will write the dated artifact and surface the summary without supervision. Because it is read-only and deterministic in scope, it is safe to run without a human present.

## Boundaries

- **Read-only** — never edit code; never run destructive or state-changing commands.
- **Proportional** — summarize; link to specifics rather than inlining them. Don't audit every line.
- **Evidence over opinion** — every claim ties to a commit, PR, or metric.
