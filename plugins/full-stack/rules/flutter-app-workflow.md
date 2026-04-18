---
description: Flutter app development — skills, architecture, DI, models (presentation → domain → data)
globs:
  - lib/**/*.dart
alwaysApply: false
---

# Flutter app development

For **format, analyze, pubspec, and verify loop**, follow `${CLAUDE_PLUGIN_ROOT}/rules/dart-quality-verification.md`.

For **ticket-driven workflow, GitNexus usage, FVM, and subagent delegation**, follow `${CLAUDE_PLUGIN_ROOT}/rules/development-workflow.md` (always applied).

- **Skills (read relevant sections before coding)**  
  Use plugin skills: `@clean`, `@presentation`, `@state`, `@di`, `@dart`, `@flutter`. Paths resolve via `${CLAUDE_PLUGIN_ROOT}/skills/<name>/SKILL.md`.

- **Architecture**  
  Dependency direction: **presentation** (`lib/screens/`, `lib/components/`, cubits under feature folders, views in `view/` where used) → **domain** (if present) → **data** (`lib/data/` or `packages/data/` — repositories, datasources, app-local models). Data must not depend on presentation. **Resources** (themes, assets): `lib/resources/` or project convention. Use barrels for data/features; **do not** barrel cubits — import cubit/state files directly.

- **DI & codegen**  
  `lib/injection.dart` (or project convention): `@InjectableInit()`, `configureDependencies()`, GetIt before `runApp()`. After DI or injectable changes: `fvm dart run build_runner build -d` in the affected package (see `${CLAUDE_PLUGIN_ROOT}/skills/workflow/SKILL.md` for FVM). Repositories: `@Injectable(as: Interface)`; cubits: `@injectable` with **named** constructor params; resolve with `sl<CubitType>()` or `getIt<CubitType>()` in screens per project standard.

- **Models**  
  Shared DTOs in a dedicated models package (Freezed + JSON per project conventions); run codegen in that package after changes. App-local models under `lib/data/models/` (or equivalent) follow the same Freezed/JSON style; run build_runner in the app package after changes.
