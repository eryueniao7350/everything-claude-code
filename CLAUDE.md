# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**everything-claude-code** (`ecc-universal` v1.10.0) is a production-ready plugin for AI coding assistants — a collection of battle-tested agents, skills, hooks, commands, rules, and MCP configurations built over 10+ months of intensive daily use.

It ships as an npm package (`npx ecc <profile>`) and supports multiple harnesses: **Claude Code**, **Cursor**, **Kiro**, **Codex**, **OpenCode**, **CodeBuddy**, **Trae**, and **Gemini**.

## Running Tests

```bash
# Full CI suite (validators + unit/integration tests)
npm test

# Tests only (no CI validators)
node tests/run-all.js

# Individual test files
node tests/lib/utils.test.js
node tests/hooks/hooks.test.js

# Lint (ESLint + markdownlint)
npm run lint

# Coverage (80% threshold enforced)
npm run coverage

# Validate skill catalog
npm run catalog:check
```

## Repository Structure

```
everything-claude-code/
├── agents/              # Specialized subagents (~45 agents)
├── commands/            # Slash commands (~80 commands)
├── skills/              # Workflow knowledge modules (~150 skill dirs)
├── rules/               # Always-on coding guidelines by language
│   ├── common/          # Language-agnostic rules
│   ├── typescript/      # TypeScript-specific rules
│   ├── golang/          # Go-specific rules
│   └── <lang>/          # cpp, csharp, dart, java, kotlin, perl, php, python, rust, swift, web
├── hooks/               # hooks.json — hook registry for Claude Code
├── mcp-configs/         # MCP server configuration presets
├── manifests/           # Install profiles and module definitions
├── schemas/             # JSON schemas for validation
├── scripts/
│   ├── hooks/           # Hook scripts (~30 scripts)
│   ├── lib/             # Shared Node.js utilities
│   └── ci/              # CI validation scripts
├── tests/
│   ├── hooks/           # Hook integration tests
│   ├── lib/             # Library unit tests
│   └── run-all.js       # Test runner
├── docs/                # Design guides and architecture docs
├── examples/            # Example CLAUDE.md templates
├── contexts/            # Context modes (dev, research, review)
├── plugins/             # Plugin metadata
├── .claude/             # Claude Code project settings (rules, commands, skills)
├── .cursor/             # Cursor IDE adapter (hooks, rules, skills)
├── .kiro/               # Kiro IDE adapter (agents, hooks, steering)
├── .codex/              # Codex adapter (agents, config)
├── .opencode/           # OpenCode adapter (commands, plugins)
├── .codebuddy/          # CodeBuddy installer
└── .trae/               # Trae installer
```

## Key Commands (CLI)

```bash
# Install ECC into current project
npx ecc typescript          # TypeScript developer profile
npx ecc python              # Python developer profile
npx ecc full                # Full install (all modules)

# ECC utilities
node scripts/ecc.js         # Main CLI entrypoint
node scripts/status.js      # Show installed components
node scripts/doctor.js      # Diagnose installation issues
node scripts/repair.js      # Fix broken installs
```

## Slash Commands

- `/tdd` - Test-driven development workflow
- `/plan` - Implementation planning
- `/e2e` - Generate and run E2E tests
- `/code-review` - Quality review
- `/build-fix` - Fix build errors
- `/learn` - Extract patterns from sessions
- `/skill-create` - Generate skills from git history
- `/quality-gate` - Run full quality checks
- `/orchestrate` - Multi-agent orchestration
- `/hookify` - Configure and manage hooks
- `/verify` - Verify a change works in the real app
- `/security` - Security review workflow

## Architecture

### Agents (`agents/`)

Markdown files with YAML frontmatter. Required fields: `name`, `description`, `tools`, `model`.

Valid models: `haiku`, `sonnet`, `opus`.

```markdown
---
name: planner
description: Expert planning specialist. Use PROACTIVELY when users request feature implementation.
tools: ["Read", "Grep", "Glob"]
model: opus
---

Your agent instructions here...
```

### Skills (`skills/`)

Each skill is a directory containing `SKILL.md` at root. Optional `examples/` and `references/` subdirs.

```markdown
---
name: tdd-workflow
description: Enforces TDD with 80%+ coverage including unit, integration, and E2E tests.
origin: ECC
---

# Skill Title

## When to Activate
...

## Core Concepts
...

## Code Examples
```

**Skill placement policy** (see `docs/SKILL-PLACEMENT-POLICY.md`):

| Type | Location | Shipped |
|------|----------|---------|
| Curated | `skills/<name>/` (this repo) | Yes |
| Learned | `~/.claude/skills/learned/` | No |
| Imported | `~/.claude/skills/imported/` | No |
| Evolved | `~/.claude/homunculus/evolved/skills/` | No |

Only curated skills in `skills/` are shipped. Learned/imported/evolved skills live in the user's home directory.

### Commands (`commands/`)

Markdown with `description:` frontmatter line required. Most commands delegate to a skill rather than duplicating the playbook.

```markdown
---
description: Legacy slash-entry shim for the tdd-workflow skill.
---

# TDD Command
Apply the `tdd-workflow` skill...
```

### Rules (`rules/`)

Markdown files organized by language. No frontmatter required. Always-on guidelines (loaded automatically). `rules/common/` applies to all projects; language subdirs apply when that language is detected.

### Hooks (`hooks/hooks.json` + `scripts/hooks/`)

`hooks.json` is the registry. All hook scripts go through `scripts/hooks/run-with-flags.js` for runtime gating via `ECC_HOOK_PROFILE` and `ECC_DISABLED_HOOKS` env vars.

Hook profiles: `minimal`, `standard`, `strict`.

```json
{
  "matcher": "Bash",
  "hooks": [{
    "type": "command",
    "command": "node \"${CLAUDE_PLUGIN_ROOT}/scripts/hooks/run-with-flags.js\" \"pre:bash:commit-quality\" \"scripts/hooks/pre-bash-commit-quality.js\" \"strict\""
  }],
  "id": "pre:bash:commit-quality"
}
```

Hook development rules:
- All hooks must exit 0 on non-critical errors (never block tool execution)
- Blocking hooks (PreToolUse, Stop): keep under 200ms — no network calls
- Async hooks: mark `"async": true` with `timeout ≤30s`
- Always log to stderr with `[HookName]` prefix

### Install Profiles (`manifests/`)

Profiles compose modules. Available profiles: `core`, `developer`, `security`, `research`, `full`.

- `manifests/install-profiles.json` — profile→module mapping
- `manifests/install-modules.json` — module→file mapping
- `manifests/install-components.json` — individual component registry

### MCP Servers (`.mcp.json`)

Default MCP servers bundled with ECC:

| Server | Purpose |
|--------|---------|
| `github` | GitHub API access |
| `context7` | Library documentation lookup |
| `exa` | Web search |
| `memory` | Persistent memory across sessions |
| `playwright` | Browser automation |
| `sequential-thinking` | Structured reasoning |

Additional presets in `mcp-configs/mcp-servers.json`.

## Development Notes

### Code Style

- CommonJS only — no ESM (`import`/`export`) unless file ends in `.mjs`
- No TypeScript — plain `.js` throughout (except `.opencode/` which uses TypeScript)
- `const` over `let`; never `var`
- Keep hook scripts under 200 lines — extract helpers to `scripts/lib/`
- File naming: **lowercase with hyphens** (e.g. `session-start.js`, `post-edit-format.js`)

### Cross-Platform

Scripts in `scripts/` support Windows, macOS, and Linux via Node.js. Never use bash-only features in hook scripts.

### Package Manager Detection

Detects npm, pnpm, yarn, bun automatically. Override with:
- `CLAUDE_PACKAGE_MANAGER` env var
- `.claude/package-manager.json` project config

## Testing Requirements

- `node tests/run-all.js` before committing
- New scripts in `scripts/lib/` require a matching test in `tests/lib/`
- New hooks require at least one integration test in `tests/hooks/`
- New agents validated by `scripts/ci/validate-agents.js`
- New skills validated by `scripts/ci/validate-skills.js`
- 80% coverage enforced via `npm run coverage`

## CI Validation Scripts (`scripts/ci/`)

| Script | Validates |
|--------|-----------|
| `validate-agents.js` | Agent frontmatter (name, description, tools, model) |
| `validate-commands.js` | Command description frontmatter |
| `validate-skills.js` | Skill SKILL.md format |
| `validate-rules.js` | Rules markdown structure |
| `validate-hooks.js` | Hook JSON configuration |
| `validate-install-manifests.js` | Install manifest JSON |
| `validate-no-personal-paths.js` | No hardcoded user paths |
| `check-unicode-safety.js` | Unicode safety in scripts |
| `catalog.js` | Skill catalog sync |

## Contributing

Follow the formats in CONTRIBUTING.md:
- Agents: Markdown with frontmatter (name, description, tools, model) — valid models: haiku, sonnet, opus
- Skills: `SKILL.md` in a named subdirectory under `skills/`
- Commands: Markdown with `description:` frontmatter
- Hooks: JSON entry in `hooks/hooks.json` + script in `scripts/hooks/`
- Rules: Markdown in `rules/<language>/` or `rules/common/`

Commit style: conventional commits (`feat:`, `fix:`, `docs:`, `test:`, `chore:`).

File naming: lowercase with hyphens (e.g., `python-reviewer.md`, `tdd-workflow/`).

Run before submitting a PR:
```bash
npm test
npm run lint
```

## Skills

Use the following skills when working on related files:

| File(s) | Skill |
|---------|-------|
| `README.md` | `/readme` |
| `.github/workflows/*.yml` | `/ci-workflow` |
| `skills/**` | `/skill-create` |
| `agents/**` | `/code-review` |

When spawning subagents, always pass conventions from the respective skill into the agent's prompt.
