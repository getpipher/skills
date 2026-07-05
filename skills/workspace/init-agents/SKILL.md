---
name: workspace-init-agents
description: Create or improve a project AGENTS.md with intelligent codebase analysis. Host-neutral satellite context file read by Claude Code, Pi, and any AGENTS.md-aware agent.
argument-hint: ""
allowed-tools: ["bash", "read", "write", "edit"]
---

# AGENTS.md Initialization (project satellite)

Bismillah! I'll analyze this codebase and create (or improve) a project `AGENTS.md` — the cross-tool context file that Claude Code, Pi, and any `AGENTS.md`-aware agent read on startup.

> **Hub / satellite model.** This skill generates a **satellite**: a host-neutral file describing *this repo*. It extends — never duplicates — the **global hub** (`~/.claude/CLAUDE.md` / `~/.pi/agent/AGENTS.md`), which already carries persona, dev standards, git conventions, secret rules, etc. A satellite holds **only project-specific** content (overview, architecture, commands, conventions, gotchas). No identity, no generic standards, no host-specific tooling — those belong to the hub.

---

## Step 0 — Migrate legacy `CLAUDE.md` (host decoupling)

`AGENTS.md` is the cross-tool standard (pi prefers it; CC reads it too). `CLAUDE.md` is CC's legacy name. If both exist, the repo double-loads and drifts — normalize to one:

- **`CLAUDE.md` exists, no `AGENTS.md`** → `git mv CLAUDE.md AGENTS.md`, then proceed (treat its content as the seed).
- **Both exist** → ask the user which to keep; default: merge into `AGENTS.md`, `git rm CLAUDE.md`.
- **Neither exists** → create `AGENTS.md` from scratch (Step 1+).
- **`AGENTS.md` exists** → analyze and suggest improvements (don't clobber; preserve accurate content).

When migrating, rephrase any CC-flavored framing host-neutrally: "guidance to Claude Code" → "guidance for AI agents"; drop any persona/standards that belong in the hub.

---

## Discovery Process

### 1. Start with existing documentation
- Read README.md for project overview and setup
- Look for `.cursor/rules/`, `.cursorrules`, `.github/copilot-instructions.md`
- Find contribution guidelines (CONTRIBUTING.md)

### 2. Discover commands and workflows
- Check package.json "scripts" (Node.js)
- Check Makefile targets
- Look for justfile, Taskfile, or similar task runners
- Find language-specific files: Cargo.toml, pyproject.toml, build.gradle, etc.
- Identify CI/CD configs (.github/workflows/, .gitlab-ci.yml) for standard commands

### 3. Understand architecture
- Read entry points (main.ts, index.js, main.py, cmd/main.go, etc.)
- Identify directory structure and its purpose
- Find configuration files and their significance
- Understand data flow and key abstractions
- Note any unconventional patterns or design decisions

---

## What to include (project-specific ONLY)

### Essential Commands
- **Build**: How to compile/bundle the project
- **Development**: How to run locally with hot reload/watch mode
- **Testing**: How to run all tests, a single file, or a specific test
- **Linting/Formatting**: How to check and auto-fix code style
- **Type Checking**: For typed languages (TypeScript, Python with mypy, etc.)
- **Database**: Migrations, seeding, reset commands if applicable
- **Deployment**: Build for production, deployment commands

### Architecture Insights (High-Level Only)
Information that requires reading **multiple files** to understand:
- Overall project structure and *why* it's organized that way
- Key architectural patterns (e.g., "hexagonal architecture with ports/adapters")
- Data flow between major components
- Important design decisions or constraints
- Where different types of work happen (e.g., "Business logic in services/, API routes in controllers/")
- Non-obvious relationships between modules

### Project-Specific Context
- Language/framework versions and constraints
- Important environment variables or configuration
- Authentication/authorization approach if applicable
- External dependencies or services the project integrates with
- Conventions that aren't obvious from code (naming, file placement)

---

## What to exclude

❌ **Anything in the global hub** — persona/identity, dev standards (2-space indent, error handling rules), git/commit conventions, secret-storage rules, a11y/perf mandates. The hub already injects these every session; duplicating them here causes drift.
❌ Generic advice ("Write clean code", "Handle errors gracefully")
❌ Obvious info discoverable by reading one file
❌ Exhaustive file/directory listings
❌ Repetition of README content (reference it instead)
❌ CC-specific framing ("guidance to Claude Code") — the file is host-neutral

---

## Structure

Use this structure (omit sections if not applicable):

```
<!-- Satellite context file — extends the global hub (~/.claude/CLAUDE.md | ~/.pi/agent/AGENTS.md). Host-neutral; project-specific only. Do not duplicate hub standards here. -->

# {Project Name}

> One-line description of what this project does.

## Repository Overview
[1-2 paragraphs: what this project does, primary language/framework]

## Architecture and Structure
[High-level architecture, key design patterns, major components and relationships]

## Development Environment
[Prerequisites, setup steps, environment variables]

## Common Commands
[Grouped: Development, Testing, Building, Database, etc.]

## Technology Stack
[Key dependencies and why, if non-obvious]

## Workflow Guidelines
[Project-specific workflows: deployment, migrations, feature development]

## Key Files and Locations
[Only files that are important but hard to discover]
```

---

## Quality Checklist

Before finalizing, verify:
- [ ] File is named `AGENTS.md` (no legacy `CLAUDE.md` left alongside)
- [ ] Header satellite-comment + host-neutral phrasing (no "Claude Code" / "Pi" framing)
- [ ] No content duplicated from the global hub (standards, persona, conventions)
- [ ] Commands are copy-pasteable and tested
- [ ] Architecture adds value beyond README
- [ ] No generic development advice
- [ ] No repetition of README (reference it)
- [ ] Focused on what makes THIS codebase unique

---

## Important Notes

- **If `AGENTS.md` exists**: analyze and suggest specific improvements — don't overwrite wholesale.
- **Test commands** before documenting (use Bash to verify they work).
- **Be concise**: quality over quantity — focus on what makes this codebase unique.
- **Reference, don't repeat**: if README documents something well, point to it.
- **Required prefix**: every generated file starts with the satellite header-comment + `# {Project Name}`.

---

Alhamdulillah! Let's create documentation that makes future work in this codebase smooth and productive — for any agent, any host. InshaAllah!
