# Figma MCP Integration Rules

> AI assistant reference for integrating Figma designs into the `everything-claude-code` repository
> via the Model Context Protocol (MCP).
>
> **Important context:** This repository is a *Claude Code plugin collection* — not a
> styled web application. It ships no CSS, no React component library, and no design tokens of its
> own. Design expertise is encoded as reusable *skills* that AI assistants load when working on
> *other* projects. Keep that distinction in mind when applying any section below.

---

## 1. Design Token Definitions

### Current state

No design token files exist in this repository. There is no `tokens.json`, `theme.ts`,
`colors.ts`, or CSS custom-property system.

### Where the token *knowledge* lives

The `design-system` skill (`skills/design-system/SKILL.md`) encodes the workflow for
generating tokens in a target project:

```
Scan CSS/Tailwind/styled-components for existing patterns
→ Extract: colors, typography, spacing, border-radius, shadows, breakpoints
→ Propose a design token set (JSON + CSS custom properties)
→ Output: DESIGN.md + design-tokens.json + design-preview.html
```

### Figma MCP guidance

When pulling tokens **from Figma into a target project** (not this repo):

1. Use the Figma MCP `get_variable_defs` tool to read the published variable collection.
2. Map Figma variable types to their code equivalents:

| Figma variable type | Code format |
|---------------------|-------------|
| Color / alias | CSS custom property `--color-*` or Tailwind `extend.colors` |
| Number (spacing) | `--spacing-*` scale or Tailwind `extend.spacing` |
| String (font family) | `--font-*` or Tailwind `extend.fontFamily` |
| Boolean (feature flag) | JS/TS constant |

3. Activate the `design-system` skill before writing any token file so the AI follows
the canonical extraction workflow.

---

## 2. Component Library

### Current state

No UI component library exists in this repository. The only `.tsx` files are illustrative
examples bundled inside the `remotion-video-creation` skill:

```
skills/remotion-video-creation/rules/assets/
  charts-bar-chart.tsx          # Remotion bar-chart composition example
  text-animations-typewriter.tsx
  text-animations-word-highlight.tsx
```

These are *reference snippets*, not a shipped component library.

### Figma MCP guidance

When implementing a Figma component in a target project:

1. Use `get_design_context` to read the component's properties, variants, and auto-layout.
2. Use `get_screenshot` to visually verify the component before writing code.
3. Activate the `frontend-patterns` skill (`skills/frontend-patterns/SKILL.md`) —
it encodes React, Next.js, and state-management conventions used in target projects.
4. Use `get_code_connect_map` to check whether a Code Connect mapping already exists
before creating a new component.

---

## 3. Frameworks & Libraries

### This repository

| Layer | Technology |
|-------|-----------|
| Runtime | Node.js ≥18 |
| Module system | CommonJS (`require`/`module.exports`) for `scripts/`; ESM for `.opencode/` |
| Language | Plain JavaScript (`scripts/`); TypeScript (``.opencode/``) |
| Test runner | Custom orchestrator — `node tests/run-all.js` |
| Linter | ESLint flat config (`eslint.config.js`) + markdownlint |
| Formatter | Prettier (`.prettierrc`) |
| Build | None for `scripts/`; `tsc` for `.opencode/` (ES2022, strict) |
| Package manager | Yarn 4.9.2 |

No UI framework (React, Vue, Svelte) is used in this repository.

### OpenCode plugin (``.opencode/``)

The only TypeScript package in the repo. Import style: ESM, path aliases via
`tsconfig.json`, strict mode enabled.

```typescript
// .opencode/index.ts — main export
export { ECCHooksPlugin, default } from "./plugins/index.js"
export * from "./tools/index.js"
```

### Target project frameworks supported by ECC skills

| Skill | Target framework |
|-------|-----------------|
| `frontend-patterns` | React, Next.js |
| `nextjs-turbopack` | Next.js + Turbopack |
| `flutter-patterns` | Flutter / Dart |
| `swift-ui-patterns` | SwiftUI |
| `liquid-glass-design` | SwiftUI, UIKit, WidgetKit (iOS 26) |
| `nestjs-patterns` | NestJS |
| `laravel-patterns` | Laravel / PHP |
| `springboot-patterns` | Spring Boot / Java |

---

## 4. Asset Management

### This repository

```
assets/
└── images/
    ├── longform/   # 9 PNG files — README long-form guide screenshots
    └── shortform/  # 25 JPEG/PNG files — README shortform screenshots
```

These are **documentation images only** — referenced in `README.md` and
`the-longform-guide.md`. No optimization pipeline, no CDN configuration.

### Figma MCP guidance for asset export

To export assets from Figma into a target project:

1. Use `get_screenshot` to preview the asset before exporting.
2. Use `upload_assets` to push optimized assets to Figma or retrieve them.
3. Follow the target project's asset conventions (e.g., Next.js `public/`, Vite `src/assets/`).

---

## 5. Icon System

### Current state

No SVG icon system. No icon directory or icon component exists in this repository.

### Figma MCP guidance

When implementing icons from a Figma icon library in a target project:

1. Use `search_design_system` to find icons by name.
2. Use `get_design_context` with the icon component's node ID to read the SVG structure.
3. Name icons by their Figma component name, converted to `kebab-case`:
   - Figma: `Icon / Arrow / Right` → file: `arrow-right.svg`
4. Prefer inline SVG for small icons; use `<img>` or CSS `background-image` for large ones.

---

## 6. Styling Approach

### This repository

No CSS methodology is in use. Markdown files use no inline styles. JavaScript scripts
produce no HTML output.

The `.prettierrc` enforces consistent formatting across `.js`, `.ts`, `.md`, and `.json` files.

### Design and styling *skills* available

| Skill | What it encodes |
|-------|----------------|
| `design-system` | Full design-token generation + visual audit workflow |
| `frontend-design` | High-quality intentional UI design principles |
| `liquid-glass-design` | iOS 26 Liquid Glass material system |
| `frontend-slides` | Zero-dependency HTML animation presentations |
| `remotion-video-creation` | React-based video composition (29 rules for 3D, audio, charts) |

### Figma MCP guidance for styling

When translating a Figma design to CSS in a target project:

1. Run the `design-system` skill's **Visual Audit** mode first to score existing
consistency (color, typography, spacing, contrast, white space, animation, component
reuse, responsive, iconography, branding).
2. Use `get_variable_defs` to pull the Figma variable collection, then emit CSS custom
properties:

```css
/* Generated from Figma variable collection */
:root {
  --color-primary:     #0055FF;
  --color-surface:     #FFFFFF;
  --spacing-4:         16px;
  --radius-md:         8px;
  --font-sans:         'Inter', system-ui, sans-serif;
}
```

3. For responsive breakpoints, check the Figma frame widths; map to the target project's
breakpoint system (Tailwind `sm/md/lg/xl`, CSS media queries, etc.).

---

## 7. Project Structure

### This repository

```
everything-claude-code/
├── agents/       # 47 subagent markdown files (YAML frontmatter + body)
├── skills/       # 183+ skills, each in skills/<name>/SKILL.md
├── commands/     # 89 command stubs, each commands/<name>.md
├── hooks/        # hooks.json + README
├── rules/        # 15 language rulesets (10 files each)
├── scripts/
│   ├── hooks/    # 33 hook scripts (.js)
│   ├── lib/      # Shared utilities (.js)
│   └── ci/       # CI validators (.js)
├── tests/
│   ├── hooks/    # ~21 integration tests
│   ├── lib/      # ~26 unit tests
│   └── run-all.js
├── docs/         # 746+ markdown guides
├── .claude/      # Claude Code runtime config
├── .opencode/    # TypeScript OpenCode plugin package
├── .cursor/      # Cursor IDE integration
├── .codex/       # OpenAI Codex integration
├── .kiro/        # Kiro integration
├── .gemini/      # Gemini CLI integration
└── assets/images/ # README documentation images
```

### Feature organization pattern

Skills are organized flat under `skills/<name>/SKILL.md`. No nested feature folders.
Each skill is self-contained: one directory, one primary `SKILL.md`, optional supporting
files alongside.

Hooks follow the same flat pattern: `scripts/hooks/<event>-<purpose>.js`.

### Figma MCP guidance for project structure

When the Figma MCP generates new files for a target project using ECC:

1. Place generated design tokens at the project root: `design-tokens.json` + `DESIGN.md`.
2. Place component implementations where the target project's conventions dictate
(e.g., `src/components/`, `lib/ui/`).
3. If adding a new ECC *skill* for a Figma-based design system workflow, place it at
`skills/figma-<name>/SKILL.md` and follow the standard YAML frontmatter:

```markdown
---
name: figma-<name>
description: Use when ...
origin: ECC
---

## When to Activate
## How It Works
## Examples
```

---

## Using Figma MCP Tools in This Codebase

### Recommended tool sequence for design-to-code

```
1. get_metadata          → read Figma file structure and page list
2. search_design_system  → find specific components or tokens by name
3. get_variable_defs     → extract all design variables (colors, spacing, typography)
4. get_design_context    → read a specific frame or component in detail
5. get_screenshot        → visually confirm before writing code
6. get_libraries         → check which published libraries are in scope
```

### Activating relevant ECC skills before using Figma MCP

| Intent | Activate skill first |
|--------|---------------------|
| Generate design tokens | `design-system` |
| Implement UI component | `frontend-patterns` |
| Audit visual consistency | `design-system` (audit mode) |
| iOS/Swift component | `liquid-glass-design` or `swift-ui-patterns` |
| Video composition | `remotion-video-creation` |
| Slides / presentation | `frontend-slides` |

### What Figma MCP cannot do in this repo

- This repo has no Figma Code Connect mappings (`get_code_connect_map` returns empty).
- There are no published component nodes to sync back (`send_code_connect_mappings`
is only relevant for target projects, not this plugin collection).

---

## Quick Reference

| Question | Answer |
|----------|--------|
| Are there design tokens? | No — use `design-system` skill to generate them in a target project |
| Is there a component library? | No — only illustrative Remotion TSX examples |
| What styling framework? | None in this repo; Tailwind / CSS custom properties in target projects |
| Where are icons? | None — pull from Figma via `search_design_system` |
| Where are assets? | `assets/images/` — documentation images only |
| What build system? | Node.js (no build) for scripts; `tsc` for `.opencode/` |
| Is there Storybook? | No |
| Code Connect mappings? | None currently |
