# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **documentation-only repository** — a curated collection of AI agent definitions, skills, commands, and feature templates for Rails development. There is no application code to build, test, or run. All content is Markdown files.

## Directory Structure

```
37signals_agents/     # Agents following DHH/37signals "vanilla Rails" philosophy
agents/               # Standard Rails agents (service objects, SOLID, Minitest)
feature_spec_agents/  # High-level planning agents (specification, planning, review)
skills/               # Reusable knowledge modules (each has SKILL.md + optional reference/)
commands/             # Claude Code slash command definitions
features/             # Feature specification templates (FEATURE_TEMPLATE.md)
```

## Agent File Format

All agent files use YAML frontmatter:

```yaml
---
name: agent_name
description: One-line description used for agent selection
tools: Read, Write, Edit, Bash, Glob, Grep  # optional, restrict available tools
model: sonnet  # optional
---
```

Agent bodies follow a consistent structure:
1. **Role** - What the agent specializes in
2. **Project Knowledge** - Tech stack and directory map (what it reads vs writes)
3. **Commands** - Executable shell commands with specific examples (e.g., `bin/rails test test/models/user_test.rb:25`)
4. **Boundaries** - Three tiers: ✅ Always / ⚠️ Ask first / 🚫 Never

## Two Architectural Philosophies

### 37signals Style (`37signals_agents/`)
- Rich models, no service objects — business logic lives in models
- Everything is CRUD — new resource instead of new action (e.g., `Closure` model instead of `closed: boolean`)
- State as records — `has_one :closure` with `scope :closed, -> { joins(:closure) }`
- Concerns for shared behavior — `include Closeable`, `include Watchable`
- Minitest + fixtures for testing
- Solid Queue/Cache/Cable, SQLite, Rails 8.x

### Standard Rails Style (`agents/`)
- Thin models and controllers delegating to service/query/presenter objects
- Service objects return `Result.new(success:, data:, error:)` via `ApplicationService`
- Query objects in `app/queries/` — chainable with `.then` pattern
- Pundit policies for authorization (deny by default)
- ViewComponent for reusable UI
- Minitest + Fixtures for testing
- PostgreSQL, Rails 7.x/8.1

## Skills Library Structure

Each skill lives at `skills/<skill-name>/SKILL.md` with optional `reference/` subdirectory for detailed docs:

```
skills/
├── rails-architecture/
│   ├── SKILL.md                    # Decision tree: where does code go?
│   └── reference/
│       ├── layer-interactions.md
│       ├── service-patterns.md
│       └── testing-strategy.md
├── hotwire-patterns/
│   ├── SKILL.md
│   └── reference/
│       ├── turbo-frames.md
│       ├── turbo-streams.md
│       └── stimulus.md
```

Skills use frontmatter with `allowed-tools` to restrict what the skill can do.

## Feature Development Workflow

The `features/FEATURE_TEMPLATE.md` defines a full TDD workflow using agents in sequence:

1. `@feature_specification_agent` → write spec (save to `features/`)
2. `@feature_reviewer_agent` → score spec (must be ≥ 7/10)
3. `@feature_planner_agent` → break into incremental PRs (3–10 steps, each < 400 lines)
4. `@tdd_red_agent` → write failing Minitest tests (RED phase)
5. Specialist agents → minimal implementation (GREEN phase)
6. `@tdd_refactoring_agent` → improve structure (REFACTOR phase)
7. `@lint_agent` → RuboCop style fixes
8. `@review_agent` → code quality review
9. `@security_agent` → Brakeman security audit

## Custom Commands (`commands/`)

Commands are Claude Code slash command definitions. Format:

```yaml
---
name: command-name
description: What this command does
tools: [Read]  # optional tool restrictions
color: blue    # optional
---
```

The `refine-specification` command is an interactive Q&A that reads a draft spec and generates a structured specification summary across 5 domains: Scope, Users & Workflows, Data Model, Integration, Non-Functional Requirements.

## Permissions (`settings.local.json`)

Pre-configured allowed/denied Bash commands. Key allowed patterns:
- `bin/rails test:*`, `bin/rails:*`, `bin/rubocop:*`, `bin/brakeman:*`
- `rails generate:*`, `rails db:migrate:*`, `rails db:rollback:*`
- `git status:*`, `git log:*`, `git diff:*` (read-only git)

Denied: `git push --force`, `git reset --hard`, `sudo`

## Target Tech Stacks

| | 37signals Style | Standard Style |
|---|---|---|
| Ruby | 3.3+ | 3.3+ |
| Rails | 8.x (edge) | 7.x / 8.1 |
| DB | SQLite or PostgreSQL | PostgreSQL |
| Testing | Minitest + Fixtures | Minitest + Fixtures |
| Auth | Custom passwordless (~150 LOC) | Rails 8 built-in |
| Jobs | Solid Queue | Solid Queue |
| Frontend | Hotwire | Hotwire + ViewComponent + Tailwind |
| Authorization | — | Pundit |
