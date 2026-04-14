---
name: karpathy-guidelines
description: Behavioral guidelines to reduce common LLM coding mistakes. Use when writing, reviewing, or refactoring code to avoid overcomplication, make surgical changes, surface assumptions, and define verifiable success criteria. Derived from Andrej Karpathy's observations on LLM coding pitfalls.
origin: https://github.com/forrestchang/andrej-karpathy-skills
license: MIT
---

# Karpathy Guidelines

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## When to Activate

- Writing new features or fixing bugs (avoid wrong assumptions)
- Reviewing or refactoring code (avoid unnecessary changes)
- Planning an implementation (surface tradeoffs before diving in)
- Any non-trivial coding task where overcomplication is a risk

## How It Works

Four principles that directly address the most common LLM coding pitfalls.

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:

| Instead of... | Transform to... |
|--------------|-----------------|
| "Add validation" | "Write tests for invalid inputs, then make them pass" |
| "Fix the bug" | "Write a test that reproduces it, then make it pass" |
| "Refactor X" | "Ensure tests pass before and after" |

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let the LLM loop independently. Weak criteria ("make it work") require constant clarification.

## Examples

### Think Before Coding

```
User: "Add caching to the search endpoint"

Bad: Silently pick Redis, implement it, done.

Good: "Before implementing, a few questions:
  - Should this cache per-user or globally?
  - What's the acceptable staleness window?
  - Is Redis already in the stack, or should I use in-memory?
  I'll wait for your answers before writing any code."
```

### Simplicity First

```
User: "Add a function to format a date"

Bad:
  class DateFormatterFactory {
    static create(strategy = 'default') { ... }
    format(date, options = {}) { ... }
  }

Good:
  function formatDate(date) {
    return date.toISOString().slice(0, 10)
  }
```

### Surgical Changes

```
User: "Fix the off-by-one error in pagination"

Bad: Fix the bug AND rename variables, reformat the block,
     extract a helper, and delete an old TODO comment.

Good: Change only the one line with the off-by-one error.
     Mention the TODO as a side note if relevant.
```

### Goal-Driven Execution

```
User: "Refactor the auth module"

Bad: Dive in, rewrite, hope it works.

Good:
  "Here's my plan:
  1. Run existing auth tests → verify: all pass
  2. Extract token validation to its own function → verify: same tests pass
  3. Extract session logic → verify: same tests pass
  I'll stop after each step and confirm before proceeding."
```

## Success Signals

These guidelines are working when you see:

- **Fewer unnecessary changes in diffs** — only requested changes appear
- **Fewer rewrites due to overcomplication** — code is simple the first time
- **Clarifying questions before implementation** — not after mistakes
- **Clean, minimal PRs** — no drive-by refactoring or "improvements"
