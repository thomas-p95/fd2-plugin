---
description: Dart Frog API development — layers, shared models, routes, DI
globs:
  - "**/routes/**/*.dart"
  - "**/api/**/*.dart"
alwaysApply: false
---

# API development (Dart Frog)

For **format, analyze, pubspec, and verify loop**, follow `${CLAUDE_PLUGIN_ROOT}/rules/dart-quality-verification.md`.

For **feature workflow, GitNexus, FVM**, follow `${CLAUDE_PLUGIN_ROOT}/rules/development-workflow.md`.

- **Skills**  
  `@dart-frog`, `@data`, `@di`, `@clean`, `@dart`. See `${CLAUDE_PLUGIN_ROOT}/skills/<name>/SKILL.md`.

- **Layers (pattern)**  
  **models** (shared package) → no dependency on api/datasource/domain. **datasource** → external access (DB, HTTP clients); depends only on models. **domain** → repository interfaces/implementations; depends on datasource + models. **Routes** → depend on repository interfaces only, not datasource types in handlers.  
  Typical layout: `lib/domain/<context>/*_repository.dart` + `*_repository_impl.dart`; `lib/datasource/<context>/*_datasource.dart` + `*_datasource_impl.dart` under the API package root (adjust paths to your repo).

- **Models package**  
  Follow Freezed + JSON conventions in the shared models package; export via the project’s public barrel (e.g. `package:models/models.dart`). Request/response layout per project. After changes: `fvm dart run build_runner build -d` in the models package root.

- **DI**  
  Central `injection.dart` (or equivalent): `@InjectableInit()`, `configureDependencies()`, GetIt `sl`. `@Injectable(as: Interface)` with **named** constructor params. Repositories depend on datasource interfaces; third-party clients (e.g. DB) registered in composition root or third-party module per project.

- **Routes**  
  Under `routes/`, grouped by resource. **Logging**: `requestLogger()` **once** on the root handler in `main.dart` inside `run()` — **not** duplicated in every segment `_middleware.dart`. Per-resource middleware: chain `provider<XRepository>((context) => sl<XRepository>())`; handlers use `context.read<XRepository>()` — **no** datasources in routes. Return JSON via `Response.json()` with `.toJson()` / list maps. Details: `@dart-frog`.
