# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Everything Claude Code** (npm: `ecc-universal` v1.10.0) is a community-maintained collection of production-ready agents, skills, hooks, commands, rules, and MCP configurations for Claude Code. It provides battle-tested workflows evolved over 10+ months of intensive daily use.

- **Repository**: github.com/affaan-m/everything-claude-code
- **License**: MIT
- **Node.js**: >=18 (CommonJS only, no transpilation)
- **Package manager**: yarn 4.9.2 (use `yarn` for dependency management)
- **CLI bins**: `ecc` (→ scripts/ecc.js), `ecc-install` (→ scripts/install-apply.js)

## Running Tests

```bash
# Run the full test suite (CI validates all components + unit tests)
npm test

# Run individual test files
node tests/run-all.js
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js

# Code coverage (80% threshold required)
npm run coverage

# Linting (ESLint + markdownlint)
npm run lint

# Validate catalog consistency
npm run catalog:check
```

The full `npm test` command runs in this order:
1. Unicode safety check
2. Agent, command, rule, skill, hook, manifest validation
3. Personal path check (no hardcoded `/home/user/` paths)
4. Catalog check
5. Unit tests

**Always run `npm test` before committing.**

## Architecture

```
everything-claude-code/
├── agents/          # 47 specialized subagents (YAML frontmatter + Markdown)
├── skills/          # 181 skill directories (SKILL.md per skill)
├── commands/        # 79 slash commands invoked by users
├── hooks/           # hooks.json event-driven automations
├── rules/           # Language/domain coding guidelines (15+ languages)
├── mcp-configs/     # MCP server configurations (150+ integrations)
├── manifests/       # Install component/module/profile definitions
├── schemas/         # JSON schemas for validation
├── scripts/         # Node.js CLI utilities and hook implementations
│   ├── hooks/       # 27 hook implementation scripts
│   ├── lib/         # Shared utility modules
│   └── ci/          # CI validators (catalog, unicode, per-type validators)
├── tests/           # Test suite mirroring scripts/ structure
│   ├── ci/          # CI validator tests
│   ├── hooks/       # Hook integration tests
│   ├── lib/         # Library unit tests
│   └── integration/ # Integration tests
├── docs/            # Extended documentation (en + i18n: ja-JP, ko-KR, pt-BR, zh-CN, zh-TW, tr)
├── contexts/        # Context files: dev.md, research.md, review.md
├── examples/        # Example CLAUDE.md, user-CLAUDE.md, statusline.json
├── ecc2/            # Rust-based ECC 2.0 alpha (SQLite session store, TUI dashboard)
├── .claude/         # Claude Code harness config (skills/, rules/, commands/, hooks/, team/)
├── .cursor/         # Cursor IDE subset (skills/, hooks/)
└── .agents/         # Codex/OpenAI harness subset
```

## File Formats

### Agents (`agents/*.md`)
YAML frontmatter required:
```markdown
---
name: code-reviewer
description: Expert code review with security and quality checks
tools: Read, Grep, Glob, Bash
model: claude-sonnet-4-6
---
[Agent instructions...]
```

### Skills (`skills/<name>/SKILL.md`)
YAML frontmatter + required sections:
```markdown
---
name: python-patterns
description: Python-specific coding patterns and best practices
origin: curated
---
## When to Activate
## Core Concepts
## Code Examples
## Anti-Patterns
## Best Practices
## Related Skills
```
Max 800 lines per skill.

### Commands (`commands/*.md`)
```markdown
---
description: Short description of what this command does
argument-hint: "[optional argument]"
---
[Command workflow instructions...]
```

### Hooks (`hooks/hooks.json`)
```json
{
  "$schema": "...",
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash|Write",
        "hooks": [
          {
            "type": "command",
            "command": "node \"${CLAUDE_PLUGIN_ROOT}/scripts/hooks/hook-name.js\"",
            "async": false,
            "timeout": 30
          }
        ],
        "description": "Human-readable description",
        "id": "pre:bash:hook-id"
      }
    ]
  }
}
```

## Key Commands (Slash Commands)

| Command | Purpose |
|---------|---------|
| `/tdd` | Test-driven development workflow |
| `/plan` | Implementation planning |
| `/e2e` | Generate and run E2E tests |
| `/code-review` | Quality code review (local or PR) |
| `/build-fix` | Fix build errors |
| `/feature-dev` | Feature development workflow |
| `/evolve` | Feature evolution |
| `/harness-audit` | Audit Claude Code harness configuration |
| `/checkpoint` | Save session checkpoint |
| `/context-budget` | Token usage advisor |
| `/go-build`, `/go-test`, `/go-review` | Go language workflows |
| `/cpp-build`, `/cpp-test`, `/cpp-review` | C++ language workflows |
| `/flutter-build`, `/flutter-test`, `/flutter-review` | Flutter workflows |
| `/hookify` | Configure hooks interactively |
| `/devfleet` | Multi-agent orchestration |
| `/eval` | Evaluation runner |

## Agents

47 specialized agents in `agents/`. Key examples:

| Agent | Model | Purpose |
|-------|-------|---------|
| architect.md | opus | Software architecture decisions |
| code-reviewer.md | sonnet | Security & quality code review |
| tdd-guide.md | sonnet | Test-driven development |
| security-reviewer.md | sonnet | Vulnerability scanning |
| planner.md | sonnet | Implementation planning |
| e2e-runner.md | sonnet | E2E test generation/execution |
| chief-of-staff.md | opus | Delegation and coordination |
| performance-optimizer.md | sonnet | Performance analysis |
| python-reviewer.md, go-reviewer.md, rust-reviewer.md, typescript-reviewer.md, cpp-reviewer.md, java-reviewer.md, kotlin-reviewer.md, flutter-reviewer.md | sonnet | Language-specific review |

## Hooks

Active hooks in `hooks/hooks.json` (30+ entries):

**PreToolUse hooks:**
- `pre:bash:block-no-verify` — blocks `git --no-verify` / hook-bypass flags
- `pre:bash:auto-tmux-dev` — auto-starts dev servers in tmux
- `pre:bash:commit-quality` — pre-commit linting, message validation, secret detection
- `pre:write:doc-file-warning` — warns about non-standard doc files
- `pre:edit-write:suggest-compact` — suggests compaction at intervals

**PostToolUse hooks:**
- `post:bash:pr-logger` — logs PR URLs after `gh pr create`
- `post:bash:build-analysis` — background build analysis
- `post:edit:format` — Prettier auto-format for JS/TS files
- `post:edit:typecheck` — `tsc --noEmit` for `.ts`/`.tsx` files
- `post:edit:console-warn` — warns about `console.log` usage
- `post:edit-write:quality-gate` — fast quality checks

**Lifecycle hooks:**
- `SessionStart` — loads previous context, detects package manager
- `PreCompact` — saves state before context compaction
- `Stop` — console.log audit, pattern extraction, cost tracking, session summary

**Hook exit codes:**
- `0` — success, continue
- `2` — block tool execution (PreToolUse only)
- non-zero — error logged, does not block

All hook scripts must `exit 0` on non-critical errors and never block tool execution unexpectedly. Use the `run-with-flags.js` wrapper for runtime gating via `ECC_HOOK_PROFILE` and `ECC_DISABLED_HOOKS`.

## Rules

Rules in `rules/` are organized by language/domain:

```
rules/
├── common/     # Hook architecture, testing requirements
├── cpp/        # C++ coding standards
├── csharp/     # C# coding standards
├── dart/       # Dart/Flutter rules
├── golang/     # Go rules
├── java/       # Java rules
├── kotlin/     # Kotlin rules
├── perl/       # Perl rules
├── php/        # PHP rules
├── python/     # Python rules
├── rust/       # Rust rules
├── swift/      # Swift rules
├── typescript/ # TypeScript rules
├── web/        # Web/frontend rules
└── zh/         # Chinese documentation rules
```

## MCP Configurations

`mcp-configs/mcp-servers.json` contains 150+ server configurations. Key integrations:

| Server | Purpose |
|--------|---------|
| github | PR, issue, repo operations |
| context7 | Live documentation lookup |
| sequential-thinking | Chain-of-thought reasoning |
| memory / omega-memory | Persistent memory across sessions |
| playwright / browserbase | Browser automation |
| firecrawl | Web scraping |
| supabase | Database operations |
| vercel / railway / cloudflare | Deployment platforms |
| exa-web-search | Web research |
| fal-ai | AI image/video generation |
| devfleet | Multi-agent orchestration (localhost:18801) |
| token-optimizer | Token reduction via content dedup |
| clickhouse | Analytics queries |

## Installation Profiles

Managed via `manifests/install-profiles.json`:
- **baseline** — minimal rules + hooks
- **standard** — adds agents + skills
- **extended** (full) — all components

```bash
npx ecc typescript          # Install TypeScript support
npx ecc-install typescript  # Compatibility alias
npx ecc list                # List available components
```

## Cross-Harness Support

| Harness | Location | Notes |
|---------|----------|-------|
| Claude Code | `.claude/` + `hooks/hooks.json` | Full support |
| Cursor | `.cursor/` | Subset of skills + hooks |
| Codex (OpenAI) | `.agents/`, `.codex/` | Subset |
| OpenCode | `.opencode/` | TypeScript plugin (built via `npm run build:opencode`) |

## Development Notes

- **CommonJS only** — no ESM (`import`/`export`) unless `.mjs`; no TypeScript in scripts
- **Package manager detection** — npm, pnpm, yarn, bun (configurable via `CLAUDE_PACKAGE_MANAGER` env or project config)
- **File naming** — lowercase with hyphens for all scripts and markdown files (e.g., `python-reviewer.md`, `session-start.js`)
- **Hook scripts** — keep under 200 lines; extract helpers to `scripts/lib/`; blocking hooks must be <200ms with no network calls
- **Skill placement** — curated skills in `skills/`; generated/imported skills under `~/.claude/skills/`. See `docs/SKILL-PLACEMENT-POLICY.md`
- **No personal paths** — never commit hardcoded paths like `/home/username/` or `/Users/name/`
- **No sensitive data** — no API keys, tokens, or secrets in any file

## Contributing

Follow formats in `CONTRIBUTING.md`. PR commit convention:

```
feat(skills): add python async patterns skill
feat(agents): add kubernetes specialist agent
fix(hooks): correct pre-bash matcher regex
docs: update skill development guide
```

Branch naming: `feat/my-contribution`

**New scripts in `scripts/lib/` require a matching test in `tests/lib/`.**
**New hooks require at least one integration test in `tests/hooks/`.**

## Skills

Use the following skills when working on related files:

| File(s) | Skill |
|---------|-------|
| `README.md` | `/readme` |
| `.github/workflows/*.yml` | `/ci-workflow` |
| This repo's JS scripts | `/everything-claude-code` |

When spawning subagents, always pass conventions from the respective skill into the agent's prompt.
