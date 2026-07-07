---
name: code-simplifier
description: Run a code-simplification pass over recently modified code — tighten it for clarity, consistency, and maintainability while preserving exact behavior. Use when asked to "simplify", "clean up", "tidy", "refactor for readability", or "remove AI slop" from code just written, or as a scheduled cleanup sweep after a change lands. Applies cosmetic fixes directly and flags structural refactors; never changes public APIs or behavior without approval.
allowed-tools: Read, Edit, Grep, Glob, Bash
---

# code-simplifier — a code-cleanup routine

Run this as a bounded pass over recently modified code: tighten it for clarity, consistency, and maintainability while preserving exact functionality. It is a repeatable procedure with a defined scope and a reported outcome — safe to run on demand, proactively right after code is written, or on a schedule as a cleanup sweep. Prioritize readable, explicit code over clever, compact solutions.

## When to run

- **On demand** — when asked to simplify, clean up, or refactor recently changed code for readability.
- **Proactively** — right after code is written.
- **Periodically** — as an unattended sweep over the latest change set (degrade to flag-only, per step 5, when no human is present).

## The pass

**1. Analyze before acting.** First understand what the code does, identify its public interfaces, and map its current behavior. Never assume — verify your understanding by reading the surrounding code and conventions before changing anything.

**2. Preserve behavior.** Your refactorings must maintain:

- All public method signatures and return types
- External API contracts
- Side effects and their ordering
- Error-handling behavior
- Performance characteristics (unless explicitly improving them)

**3. Apply project standards.** Follow the conventions already established in the codebase and in CLAUDE.md — naming, file organization, error-handling patterns, import style, and language idioms. Match the surrounding code rather than imposing a personal style. (For example, a TypeScript/React project might call for ES modules with sorted imports, the `function` keyword over arrow functions, explicit return types on top-level functions, and explicit Props types on components — but always defer to what the project actually does.)

**4. Simplification techniques**, applied in order of priority:

- **Remove AI code slop** — eliminate AI-generated patterns that don't match the codebase:
  - Extra comments a human wouldn't add or that are inconsistent with the file (keep genuinely useful doc comments)
  - Defensive checks or try/catch blocks that are abnormal for that area, especially on trusted or already-validated code paths
  - Casts to `any` used to sidestep type issues
  - Any other style that is inconsistent with the file
- **Reduce complexity** — flatten nested conditionals, extract complex expressions, use early returns
- **Eliminate redundancy** — remove duplicate code, consolidate similar logic, apply DRY
- **Improve naming** — use descriptive, consistent names that reveal intent
- **Extract methods** — break large functions into smaller, focused ones
- **Simplify data structures** — use appropriate collections and types
- **Remove dead code** — eliminate unreachable or unused code
- **Clarify logic flow** — make the happy path obvious and handle edge cases clearly
- **Add a clarifying comment where one is missing** — the complement to slop removal: after stripping noise comments, add a single concise line where a non-obvious decision, workaround, or performance choice would otherwise puzzle the next reader. Cap this at a few; you are capturing intent, not writing documentation.

**5. Classify every change: cosmetic or structural.** Decide which kind each edit is before applying it, because they carry different risk:

- **Cosmetic fixes** — dead-code and unused-import removal, naming, control-flow tightening, visibility reduction, slop removal. These are not judgment calls; apply them directly.
- **Structural refactors** — consolidating or merging functions, changing abstractions or data flow, splitting modules. These are the exception, not the default. The bar is: *would a senior engineer say the current state is clearly wrong, not merely imperfect?* When that bar is met and a human is in the loop, propose the change first (describe it, show before/after) and get approval before applying. When running unattended — a proactive or automated pass — do not perform structural refactors; flag them instead.

**6. Maintain balance — avoid over-simplification.** Do not sacrifice clarity for brevity. Specifically avoid:

- Nested ternary operators — prefer switch statements or if/else chains for multiple conditions
- Dense one-liners that are hard to debug or extend
- Overly clever solutions that obscure intent
- Combining too many concerns into a single function or component
- Removing helpful abstractions that improve organization

Explicit code is often better than maximally compact code. Optimize for the next developer, not the fewest lines.

**7. Quality checks.** For each refactoring:

- Verify the change preserves behavior
- Ensure tests still pass (note if any need updates)
- Confirm complexity genuinely decreased
- Confirm the result is more readable than before

**8. Scope and budget.** Focus only on code that was recently modified or touched in the current session unless explicitly told to review a broader scope. Keep your footprint proportional to that change: a simplification pass should tighten what was written, not balloon into a rewrite — if a worthwhile improvement would substantially exceed the size of the original change, flag it rather than doing it. Flag out-of-scope issues you notice in untouched code instead of acting on them.

## Boundaries

- Never change public APIs without explicit permission
- Maintain backward compatibility and all documented behavior
- Don't introduce new dependencies without discussion
- Respect existing code style and conventions
- Keep performance neutral or better

## When to seek clarification

Pause and ask when you encounter ambiguous behavior that lacks tests, a potential bug that refactoring would expose, a public API change that would substantially simplify the code, a performance trade-off, or an architectural decision that affects the approach.

## Output

For anything beyond trivial in-place cleanups, report concisely so your restraint is visible:

- **Applied** — the cosmetic simplifications you made, and why each improves the code (with before/after for anything significant)
- **Flagged** — structural refactors, out-of-scope issues, or risks you did *not* act on, each with a one-line rationale
- **Left alone** — anything you deliberately preserved (helpful abstractions, intentional patterns) that might look simplifiable but isn't
- Any caveats, assumptions, or tests that need updating

Keep it tight — a short, scannable summary, not a narrative. Your goal is code that developers will thank you for — a joy to read, understand, and modify — while preserving its complete functionality.
