# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Squad** is a programmable multi-agent runtime for GitHub Copilot, built on `@github/copilot-sdk`. It lets teams define persistent AI development teams that live in `.squad/`, where each member is a specialist (Lead, Frontend, Backend, Tester, etc.) with isolated context, shared decisions, and codebase knowledge.

- **Runtime:** Node.js ≥20, TypeScript strict mode, ESM-only
- **Packages:** `@bradygaster/squad-sdk` (core library) and `@bradygaster/squad-cli` (CLI)
- **Distribution:** `npm install -g @bradygaster/squad-cli` / `npm install @bradygaster/squad-sdk`

## Commands

```bash
# Build
npm run build            # builds both packages (sdk then cli)

# Type checking (lint)
npm run lint             # tsc --noEmit for both packages
npm run lint:docs        # markdownlint + cspell on docs and README

# Tests
npm test                 # vitest run (all test/**/*.test.ts)
npm run test:watch       # vitest in watch mode

# Run a single test file
npx vitest run test/path/to/file.test.ts

# Local dev with linked packages
npm run dev:link         # build + npm link both packages globally
npm run dev:unlink       # undo the links

# Docs site (Astro)
npm run docs:dev         # dev server
npm run docs:build       # production build

# Squad config
npm run sync-templates   # sync template files from squad-sdk to root
```

> `npm run build` runs `prebuild` first, which bumps the build version and syncs templates.

## Repository Structure

```
packages/
  squad-sdk/src/          # Core SDK — zero side effects, modular
  squad-cli/src/          # CLI entry point and interactive shell
test/                     # All tests (Vitest), mapped via vitest.config.ts aliases
test-fixtures/            # Fixture data for tests
templates/                # Markdown template files for squad init
.squad/                   # Team state (agents, decisions, charters) — live team data
squad.config.ts           # SDK-first team definition (source of truth for .squad/)
docs/                     # Astro documentation site
samples/                  # Working usage examples
```

## Architecture

### Two-Package Monorepo

**`squad-sdk`** is the core library. Entry point: `packages/squad-sdk/src/index.ts`. Key module groups:

| Module | Responsibility |
|--------|---------------|
| `adapter/` | Bridge to `@github/copilot-sdk` |
| `agents/` | Agent lifecycle and onboarding |
| `casting/` | Character/persona assignment engine |
| `coordinator/` | Routes messages to the right agent using response tiers |
| `config/` | Config loading and schema validation |
| `hooks/` | Security hooks, PII filters, governance |
| `marketplace/` | Plugin discovery and management |
| `runtime/` | Streaming, OTEL telemetry, cost tracking, i18n, benchmarks |
| `tools/` | Tool definitions and execution |
| `builders/` (`build/`) | SDK-first builders: `defineTeam`, `defineAgent`, `defineRouting`, etc. |
| `upstream/` | Remote squad discovery and cross-squad delegation |
| `skills/` | Skill plugin system |
| `sharing/` | Export/import portability |
| `ralph/` | Automated issue triage |

**`squad-cli`** dispatches commands and provides an interactive REPL. Entry point: `packages/squad-cli/src/cli-entry.ts`. The interactive shell (`cli/shell/`) is built with Ink (React terminal UI).

### Configuration

`squad.config.ts` (SDK-first) is the authoritative team definition. It uses `defineSquad`, `defineTeam`, `defineAgent`, `defineRouting`, and `defineCasting` builders from `@bradygaster/squad-sdk`. Running `squad build` regenerates the `.squad/*.md` files from it.

### Testing

All tests live in `test/` and are run from the repo root via `vitest`. The `vitest.config.ts` aliases `@bradygaster/squad-sdk` to force workspace resolution (avoids duplicate installs under `squad-cli/node_modules`). Coverage reports go to `./coverage`.

## Copilot Agent Workflow

When working on issues autonomously (see `.github/copilot-instructions.md`):

1. Read `.squad/team.md` for roles and capability profile before starting.
2. Read `.squad/routing.md` for work routing rules.
3. If an issue has a `squad:{member}` label, read `.squad/agents/{member}/charter.md` and work in that member's voice.
4. Use branch naming: `squad/{issue-number}-{kebab-case-slug}`
5. Write decisions to `.squad/decisions/inbox/copilot-{brief-slug}.md` — the Scribe merges them.
6. PRs must reference the issue (`Closes #N`) and note the squad member if applicable.
