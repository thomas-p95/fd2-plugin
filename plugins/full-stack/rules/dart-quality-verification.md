---
description: Dart format, analyze, verify loop, and pubspec conventions per package
globs: "**/*.dart"
alwaysApply: false
---

# Dart quality verification

Use this **per package** in a monorepo or single app. Align with FVM and Melos when the repo uses them (see `${CLAUDE_PLUGIN_ROOT}/rules/development-workflow.md` and `${CLAUDE_PLUGIN_ROOT}/skills/workflow/SKILL.md`).

- **Loop**: understand the request and relevant files → implement the **smallest** change that satisfies it → **verify** before considering work done.
- **Format**: from the **package root** you changed, run `fvm dart format .` (or `fvm dart run melos dart-format` / project script if that is the standard).
- **Analyze**: in that same package root, `fvm dart analyze`; keep **0 issues**. Treat analyzer infos as must-fix unless the project explicitly documents an exception.
- **Pubspec**: keep `dependencies` and `dev_dependencies` blocks alphabetically sorted; keep shared tooling versions (`build_runner`, code generators, analysis options) aligned across packages unless there is a deliberate reason to diverge.
