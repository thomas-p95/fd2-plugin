---
description: Full-stack Flutter/Dart workflow — discovery tools, FVM, quality, branching, subagent delegation
alwaysApply: true
---

# Development Workflow

Default context: **full-stack Flutter/Dart** with Clean Architecture-style layering. Layer paths and tools vary by repo; details live in `${CLAUDE_PLUGIN_ROOT}/skills/workflow/SKILL.md`, `${CLAUDE_PLUGIN_ROOT}/rules/flutter-app-workflow.md`, `${CLAUDE_PLUGIN_ROOT}/rules/dart-quality-verification.md`, and the implementor agents below.

## Codebase understanding (GitNexus)

When this project **uses GitNexus** (indexed repo), prefer it for “where does this live?”, flow tracing, impact before refactors, and graph-backed exploration—**not** only text search or guesswork.

- **MCP:** Prefer GitNexus tools (`query`, `context`, `cypher`, `impact`, `route_map`, and related) when they fit the question.
- **CLI:** `npx gitnexus analyze` refreshes the index; `npx gitnexus status` checks freshness.
- **Skills:** `gitnexus-guide`, `gitnexus-exploring`, `gitnexus-debugging`, `gitnexus-impact-analysis`, `gitnexus-refactoring`, `gitnexus-cli`.

If GitNexus is not configured for the project, use normal repo search, stack traces, and reading source.

---

## Feature work (requirements-driven)

**When to run:** User gives **requirements** (or a clear task). Follow **1 → 4** in order. Do not invent requirements.

1. **Branch**  
   Create a branch per `${CLAUDE_PLUGIN_ROOT}/skills/workflow/SKILL.md` (types, naming, target branch). Example: feature → `feat/short-slug-from-task`.

2. **Implement**  
   Implement against the stated requirements; delegate using the subagent table when the task matches.

3. **Tests — MUST use `/test-writer`**  
   After code changes, invoke the test-writer subagent to add or update tests for what changed. Task not done until tests exist or are updated as appropriate.

4. **Review — MUST use `/code-reviewer`**  
   After implementation and tests, invoke the code-reviewer subagent for format, analyze, project lint rules (e.g. `bloc_lint` / analyzer if configured), and review. Task not done until review passes and required fixes are applied.

---

## Everyday quality (all work)

Follow `${CLAUDE_PLUGIN_ROOT}/skills/workflow/SKILL.md` for branching, CI, and project conventions. Short summary:

- **FVM:** Prefer `fvm` for Flutter/Dart commands when the repo uses FVM (e.g. `fvm flutter run`, `fvm dart run build_runner build -d`).
- **Format / analyze:** Use the project’s standard (`fvm dart run melos dart-format`, `fvm dart format .`, or scripts in `melos.yaml` / CI). Run `fvm dart analyze` (or project equivalent); keep analyzer clean per project policy.
- **UI copy:** Use localization for user-visible strings when the app is set up for it.
- **Architecture:** Follow Clean Architecture boundaries; see `@clean`, `@presentation`, `@data`, `@domain`, `@di`, `@state`.

---

## Subagent delegation

When a task matches a row, **delegate to that subagent**. Use [explicit invocation](https://cursor.com/docs/context/subagents#explicit-invocation): **`/name`** in the prompt (e.g. `/data-implementor add the endpoint`) or natural mention.

| Task | Invoke with | When to use |
|------|-------------|-------------|
| **Data layer (remote + local)** | `/data-implementor` | Endpoints, API contracts, Retrofit (or equivalent), request/response models, local persistence, or work in the project’s **data** package / layer. (`${CLAUDE_PLUGIN_ROOT}/agents/data-implementor.md`) |
| **Domain layer (repositories + use cases)** | `/domain-implementor` | Repository interfaces/implementations, use cases, domain exceptions, orchestration across data sources. (`${CLAUDE_PLUGIN_ROOT}/agents/domain-implementor.md`) |
| **Presentation layer** | `/presentation-implementor` | Screens, cubits, routes, views, widgets in the project’s presentation areas. (`${CLAUDE_PLUGIN_ROOT}/agents/presentation-implementor.md`) |
| **Tests** | `/test-writer` | **MANDATORY** after code changes: add or update unit, widget, or integration tests. (`${CLAUDE_PLUGIN_ROOT}/agents/test-writer.md`) |
| **Code review** | `/code-reviewer` | **MANDATORY** after development and tests: format, analyze, lint, review. (`${CLAUDE_PLUGIN_ROOT}/agents/code-reviewer.md`) |

**Do not ask the user which subagent to use.** Pick from the table from the task shape, or tell them to run **`/name`** if they must trigger manually. Never ask the user to choose between subagents.
