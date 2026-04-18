<p align="center">
  <img src="https://em-content.zobj.net/source/apple/391/electric-plug_1f50c.png" width="120" />
</p>

<h1 align="center">@thomas/fd2-plugin</h1>

<p align="center">
  <strong>one source of truth for AI rules across every project, every tool, every stack</strong>
</p>

<p align="center">
  <a href="#before--after">Before/After</a> •
  <a href="#install">Install</a> •
  <a href="#usage">Usage</a> •
  <a href="#stacks">Stacks</a> •
  <a href="#shared-content">Content</a> •
  <a href="#supported-tools">Tools</a> •
  <a href="#adding-new-content">Contribute</a> •
  <a href="#ci-integration">CI</a>
</p>

---

Internal plugin that registers the marketplace in Claude Code, enabling native plugin loading by stack. No file copying — skills, agents, commands, and hooks load directly from the plugin source.

## Before / After

<table>
<tr>
<td width="50%">

### 😵 Without plugin

```
project-a/CLAUDE.md           ← hand-written, incomplete
project-a/.cursorrules         ← v3, outdated
project-b/CLAUDE.md           ← missing entirely
project-b/.cursorrules         ← v5, different from A
project-c/.cursor/rules/       ← someone's personal copy
```

Rules drift. New projects start from scratch. Onboarding is "copy from that other repo."

</td>
<td width="50%">

### ✅ With plugin

```bash
npx fd2-plugin init
```

```jsonc
// .claude/settings.json — auto-written
{
  "extraKnownMarketplaces": {
    "fd2": { "source": { "source": "github",
                            "repo": "thomas-p95/fd2-plugin" } }
  },
  "enabledPlugins": { "full-stack@fd2": true }
}
```

Claude Code loads skills, agents, commands, and hooks natively. Always current — no sync needed.

</td>
</tr>
</table>

```
┌──────────────────────────────────────────┐
│  STACKS            ████████ full-stack   │
│  SHARED SKILLS     ████████ 15 skills   │
│  SHARED AGENTS     ████████ 5 agents    │
│  SHARED COMMANDS   ████████ 2 cmds      │
│  SUPPORTED TOOLS   ████████ 2 tools     │
│  FILES TO COPY     ░░░░░░░░ 0           │
└──────────────────────────────────────────┘
```

## Stacks

Content lives in `plugins/<stack>/`. Each stack is an independent plugin with its own rules, skills, agents, commands, and hooks.

| Stack | Status | Contents |
|-------|--------|----------|
| `full-stack` | ✅ Active | Flutter/Dart — clean architecture, BLoC, Fastlane |

## Supported Tools

| Tool | Settings file | What gets registered |
|------|--------------|----------------------|
| **Claude Code** | `.claude/settings.json` | `extraKnownMarketplaces` + `enabledPlugins` |

More coming: Cursor, Windsurf, Copilot, Cline.

## Install

### As a Claude Code plugin (recommended — no CLI needed)

```
/plugin marketplace add thomas-p95/fd2-plugin
/plugin install full-stack@fd2
```

Skills, agents, commands, and hooks load natively. Claude Code reads `plugins/full-stack/` directly from the repo.

### Via CLI — register marketplace in project settings

```bash
# Run directly from GitHub, no install needed
npx github:thomas-p95/fd2-plugin init

# Or install as devDependency
npm install --save-dev github:thomas-p95/fd2-plugin
npx fd2-plugin init

# Or install globally
npm install -g github:thomas-p95/fd2-plugin
fd2-plugin init
```

## Usage

### `init` — Register marketplace in project

```bash
# Both Claude + Cursor (default targets)
npx fd2-plugin init

# Preview only
npx fd2-plugin init --dry-run

# Single tool
npx fd2-plugin init -t claude
```

Output:

```
Initializing AI plugin (stack: full-stack)...
  registered marketplace: fd2 (thomas-p95/fd2-plugin)
  enabled plugin: full-stack@fd2
✓ claude configured

Done! AI configs installed.
```

Writes `.fd2-plugin.json` (tracks stack + configured tools) and updates `.claude/settings.json`.

### `sync` — Re-register (idempotent)

```bash
npx fd2-plugin sync                        # stack from .fd2-plugin.json (default full-stack)
npx fd2-plugin sync --dry-run              # preview only
```

Since plugins load live from source, sync is mainly useful after adding tools or changing `.fd2-plugin.json`.

### `list` — Browse available content

```bash
npx fd2-plugin list
npx fd2-plugin list -c skills
```

```
[full-stack]

RULES
  dart-frog-api-workflow   — Dart Frog API — layers, shared models, routes, DI
  dart-quality-verification — Format, analyze, verify loop, pubspec conventions per package
  development-workflow     — Development workflow (FVM, git, quality) and subagent delegation
  flutter-app-workflow   — Flutter app — skills, architecture, DI, models (presentation → data)

SKILLS
  bash                     — Bash syntax, error handling, security, ShellCheck compliance
  clean                    — Layer separation, dependency rules, repositories, use cases
  dart                     — Naming, syntax, null safety, async, modern language features
  dart-frog                — Dart Frog routes, middleware, HTTP handlers, context DI
  data                     — Retrofit API, remote/local data sources, models, DI (packages/data)
  di                       — get_it + injectable: annotations, modules, test overrides
  workflow                 — Git branching, feature development, CI/CD, code quality
  domain                   — Repository interfaces, use cases, domain exceptions (packages/domain)
  flutter                  — Widget architecture, composition rules, performance patterns
  git                      — Commits, branches, merges, rebases, conflict resolution, recovery
  presentation             — Screens, cubits, routes, barrel files (lib/screens/)
  pubspec                  — pubspec.yaml dependency management, always use latest pub.dev versions
  ruby                     — Fastlane lane/helper design, env vars, shell safety, CI patterns
  state                    — Cubit/BLoC with flutter_bloc, @freezed states, DataLoadStatus
  test                     — Unit, widget, integration tests with bloc_test and mocktail

AGENTS
  code-reviewer            — Proactive Dart/Flutter + Clean Architecture code reviewer
  data-implementor         — Retrofit APIs, data sources, models, DI (packages/data)
  domain-implementor       — Repository interfaces, use cases, DI wiring (packages/domain)
  presentation-implementor — Screens, cubits, routes, views (lib/screens/)
  test-writer              — Write/update test suites after feature changes

COMMANDS
  help                     — List all available commands and skills in this plugin
  specs                    — Accept ticket ID + requirements, run full 5-step dev workflow
```

## Shared Content

All shared content lives under `plugins/<stack>/` as Markdown with YAML frontmatter.

### Structure

```
plugins/
  full-stack/               # Flutter/Dart stack
    .claude-plugin/
      plugin.json           # Claude Code plugin manifest
    rules/                  # Coding standards (.md)
    skills/                 # Skill folders (each with SKILL.md)
    agents/                 # Agent definitions (.md)
    commands/               # Command templates (.md)
    hooks/
      hooks.json            # PostToolUse dart-format + SessionStart
```

### File Format

```markdown
---
title: Human-readable title
description: One-line description (shown in `list`)
globs: "**/*.dart"        # Optional: file scope (Cursor)
alwaysApply: true          # Optional: auto-apply (Cursor)
---

Content here. Markdown supported.
```

### How It Works

```
npx fd2-plugin init
        │
        └── .claude/settings.json ← extraKnownMarketplaces + enabledPlugins

Claude Code loads plugins natively:
  plugins/full-stack/{skills,agents,commands,hooks}/  ← loaded directly from repo
```

| Command | What it does |
|---------|-------------|
| `init` | Write `.fd2-plugin.json` + register marketplace in tool settings |
| `sync` | Re-register (idempotent — safe to run anytime) |
| `list` | Display available rules/skills/agents/commands |

## Adding New Content

1. Create `.md` in the right dir under `plugins/<stack>/` (`rules/`, `skills/`, `agents/`, `commands/`)
2. Add frontmatter: `title` + `description` (minimum)
3. Write content
4. `npx fd2-plugin list` → verify it shows
5. Commit + push → all projects pick it up immediately (no re-sync needed)

## Adding a New Stack

1. Create `plugins/<stack>/` with `rules/`, `skills/`, `agents/`, `commands/` subdirs
2. Add `plugins/<stack>/.claude-plugin/plugin.json`
3. Register in `.claude-plugin/marketplace.json`
4. Add skills, agents, rules, commands

## Notes

- `.fd2-plugin.json` tracks configured tools and active stack — commit this file
- No generated AI config files — Claude Code and Cursor read plugin content directly from the repo source

## License

Internal use only.
