# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**everything-claude-code** (`ecc-universal` v1.10.0) is a production-ready Claude Code plugin — a curated collection of agents, skills, hooks, commands, rules, and MCP configurations evolved through 10+ months of intensive daily use. It supports Claude Code, Cursor, OpenCode, and Codex.

Install via: `npx ecc typescript` (or `npx ecc-install typescript` for compat).

## Repository Structure

```
everything-claude-code/
├── agents/          # 47 specialized subagents (code-reviewer, planner, architect, etc.)
├── skills/          # 183+ workflow modules (tdd-workflow, security-review, etc.)
├── commands/        # 89 slash command definitions (/tdd, /plan, /code-review, etc.)
├── hooks/           # hooks.json (30KB) + README — trigger-based automations
├── rules/           # Language rulesets: common/, cpp/, csharp/, dart/, golang/,
│                    #   java/, kotlin/, perl/, php/, python/, rust/, swift/,
│                    #   typescript/, web/, zh/
├── scripts/         # Node.js utilities for hooks, CI, install, session management
│   ├── hooks/       # 33 hook scripts (pre/post-bash, pre/post-edit, session, etc.)
│   ├── lib/         # Shared utilities: utils.js, package-manager.js, session-manager.js, etc.
│   └── ci/          # Validation scripts run in CI
├── tests/           # Test suite (no Jest — plain Node.js)
│   ├── hooks/       # Hook integration tests (~21 files)
│   ├── lib/         # Unit tests for scripts/lib/ (~26 files)
│   ├── ci/          # CI validator tests
│   └── run-all.js   # Master test orchestrator
├── docs/            # 746+ markdown docs, guides, architecture, localization
├── examples/        # Sample CLAUDE.md files for different project types
├── manifests/       # install-profiles.json, install-components.json
├── mcp-configs/     # mcp-servers.json for external integrations
├── contexts/        # dev.md, research.md, review.md context guides
├── .claude/         # Claude Code config: commands/, rules/, skills/, ecc-tools.json
├── .github/         # CI workflows: ci.yml, release.yml, maintenance.yml, monthly-metrics.yml
└── install.sh / install.ps1  # Bootstrap installers
```

## Running Tests

```bash
# Run all tests (includes CI validators + unit tests)
node tests/run-all.js

# Run the full npm test suite (validators + unicode check + catalog + tests)
npm test

# Run individual test files
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js

# Coverage (80% threshold on lines/functions/branches/statements)
npm run coverage
```

## Key npm Scripts

```bash
npm test              # Full test suite (CI validators + run-all.js)
npm run lint          # ESLint + markdownlint on all .md files
npm run coverage      # c8 coverage with 80% threshold
npm run catalog:check # Validate catalog is in sync
npm run catalog:sync  # Regenerate catalog
npm run harness:audit # Audit the hook harness
npm run build:opencode # Build OpenCode plugin (auto-runs on prepack)
```

## Architecture: Core Components

### Agents (`agents/*.md`)

Specialized subagents for task delegation. Format: Markdown with YAML frontmatter.

```markdown
---
name: code-reviewer
description: Expert code review specialist. Use immediately after writing code.
tools: ["Read", "Grep", "Glob", "Bash"]
model: sonnet
---

Agent body here...
```

Key agents: `code-reviewer`, `planner`, `architect`, `security-reviewer`, `tdd-guide`, `python-reviewer`, `rust-reviewer`, `typescript-reviewer`, language-specific build resolvers.

### Skills (`skills/<name>/SKILL.md`)

Workflow modules loaded based on context. Each skill lives in its own subdirectory.

```markdown
---
name: tdd-workflow
description: Use when writing features, fixing bugs, or refactoring. Enforces TDD with 80%+ coverage.
origin: ECC
---

# Skill Title

## When to Activate
...

## How It Works
...

## Examples
...
```

183+ skills covering: TDD, security-review, deep-research, architecture patterns, language-specific patterns (Python, Go, Rust, TypeScript, etc.), e2e-testing, eval-harness, and more.

### Commands (`commands/*.md`)

Slash command definitions. Many are legacy shims delegating to skills.

```markdown
---
description: Legacy slash-entry shim for the tdd-workflow skill.
---

# /tdd

Apply the `tdd-workflow` skill.
```

Key commands: `/tdd`, `/plan`, `/e2e`, `/code-review`, `/build-fix`, `/learn`, `/skill-create`, `/security-review`, `/verify`.

### Hooks (`hooks/hooks.json` + `scripts/hooks/*.js`)

`hooks.json` defines trigger-based automations. All hook scripts use the `run-with-flags.js` wrapper for runtime gating via `ECC_HOOK_PROFILE` / `ECC_DISABLED_HOOKS`.

Hook events: `PreToolUse`, `PostToolUse`, `Stop`, `SessionStart`, `PreCompact`.

Key hooks:
- `session-start.js` — bootstrap session state
- `post-edit-format.js` — auto-format after edits
- `post-edit-typecheck.js` — type-check TypeScript after edits
- `pre-bash-commit-quality.js` — quality gate before git commits
- `cost-tracker.js` — track token/cost per session
- `config-protection.js` — block dangerous config mutations

### Rules (`rules/<language>/`)

Each language directory contains: `agents.md`, `code-review.md`, `coding-style.md`, `development-workflow.md`, `git-workflow.md`, `hooks.md`, `patterns.md`, `performance.md`, `security.md`, `testing.md`.

Languages: `common`, `cpp`, `csharp`, `dart`, `golang`, `java`, `kotlin`, `perl`, `php`, `python`, `rust`, `swift`, `typescript`, `web`, `zh`.

## Development Notes

### File Naming

- Scripts: `lowercase-with-hyphens.js` (e.g., `session-start.js`, `post-edit-format.js`)
- Agents/commands/skills/rules: `lowercase-with-hyphens.md`
- Note: `.claude/rules/everything-claude-code-guardrails.md` specifies `camelCase` for file naming — that applies to JS utilities in `scripts/lib/`, not markdown files.

### Code Style (scripts/)

- CommonJS only — `require`/`module.exports`. No ESM unless `.mjs`
- No TypeScript — plain `.js` throughout
- `const` over `let`; never `var`
- Hook scripts: keep under 200 lines — extract helpers to `scripts/lib/`
- All hooks must `exit 0` on non-critical errors — never block tool execution

### Hook Development

- Hooks receive JSON on stdin; use `run-with-flags.js` wrapper to handle parsing/gating
- Blocking hooks (`PreToolUse`, `Stop`): keep under 200ms — no network calls
- Async hooks: mark `"async": true` in `settings.json` with timeout ≤30s
- Log to stderr with `[HookName]` prefix; always exit 0 on parse errors

```js
// Minimal hook script pattern
const { run } = require('./run-with-flags');

run(async (input) => {
  const data = JSON.parse(input);
  // ... hook logic
});
```

### Testing Requirements

- Run `node tests/run-all.js` before committing
- New `scripts/lib/` utilities require a matching test in `tests/lib/`
- New hooks require at least one integration test in `tests/hooks/`
- Run `npx markdownlint-cli '**/*.md' --ignore node_modules` for markdown changes

### Commit Convention

Use conventional commits: `fix:`, `feat:`, `docs:`, `test:`, `refactor:`, `chore:`.

Example: `feat: add kotlin-patterns skill`

## Package Manager

Detection order: `CLAUDE_PACKAGE_MANAGER` env var → project config → auto-detect (npm, pnpm, yarn, bun). This project uses **yarn 4.9.2** (`packageManager` field in `package.json`).

## Multi-IDE Support

The repo ships integration for multiple AI coding tools:

| Directory | Target |
|-----------|--------|
| `.claude/` | Claude Code |
| `.cursor/` | Cursor IDE |
| `.codex/` | OpenAI Codex |
| `.opencode/` | OpenCode |
| `.kiro/` | Kiro |
| `.gemini/` | Gemini CLI |

## Contributing

See `CONTRIBUTING.md` for full details. Quick reference:

- **Agents**: YAML frontmatter with `name`, `description`, `tools`, `model`
- **Skills**: `skills/<name>/SKILL.md` with When to Activate, How It Works, Examples sections
- **Commands**: `description:` frontmatter required; prefer delegating to a skill
- **Hooks**: JSON entry in `hooks/hooks.json` + script in `scripts/hooks/`
- **Rules**: Follow the per-file pattern of existing language directories

Skill placement policy: curated skills go in `skills/`; generated/imported skills go in `~/.claude/skills/`. See `docs/SKILL-PLACEMENT-POLICY.md`.

## Skills (for Claude Code)

Use the following skills when working on specific files:

| File(s) | Skill |
|---------|-------|
| `README.md` | `/readme` |
| `.github/workflows/*.yml` | `/ci-workflow` |
| Any code change | `/code-review` |
| New feature | `/tdd` |
| Security-sensitive code | `/security-review` |

When spawning subagents, always pass conventions from the respective skill into the agent's prompt.
