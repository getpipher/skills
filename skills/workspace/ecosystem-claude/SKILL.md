---
name: workspace-ecosystem-claude
description: Create ecosystem-wide context-file architecture (AGENTS.md | CLAUDE.md) for multi-repo organizations. Host-neutral — works for Claude Code, Pi, and any AGENTS.md-aware agent.
argument-hint: "[org-name]"
allowed-tools: ["bash", "read", "write", "edit"]
---

# Ecosystem Context-File Architecture (AGENTS.md)

Bismillah! I'll analyze your organization's repositories and help set up an ecosystem-wide **context-file architecture** — the cross-tool standard where `AGENTS.md` is the primary file (read by Claude Code, Pi, and any `AGENTS.md`-aware agent) and `CLAUDE.md` is Claude Code's legacy name for the same file.

> **Hub / satellite model.** One **ecosystem hub** `AGENTS.md` (usually in your org's main/overview repo) carries org-wide context: mission, repo index, cross-repo standards. Each member repo gets a lightweight **satellite** `AGENTS.md` that links to the hub and holds only repo-specific content. An agent landing in any repo can orient itself: the satellite points to the hub, the hub indexes everything. This mirrors the `/workspace:init-agents` hub/satellite pattern at org scale.

**This command is always safe to run.** It detects first, shows analysis, and asks before any changes.

---

## STEP 0 — Normalize legacy `CLAUDE.md` (host decoupling)

`AGENTS.md` is the cross-tool standard; `CLAUDE.md` is CC's legacy name. If both exist in a repo, it double-loads and drifts. Before and while building the ecosystem, normalize each repo to one file:

- **`CLAUDE.md` exists, no `AGENTS.md`** → `git mv CLAUDE.md AGENTS.md`, then proceed (treat its content as the seed).
- **Both exist** → ask the user which to keep; default: merge into `AGENTS.md`, `git rm CLAUDE.md`.
- **`AGENTS.md` exists** → analyze and build on it.
- **Neither exists** → create from scratch (Step 4+).

When migrating, rephrase any CC-flavored framing host-neutrally ("guidance to Claude Code" → "guidance for AI agents"). This is the same migration the `/workspace:init-agents` skill performs per-repo; run it across every member repo as you wire the ecosystem.

---

## STEP 1: DETECT (Always runs first — completely safe)

### 1.1 Get Repository Info

```bash
# Get organization from git remote
git remote get-url origin 2>/dev/null

# Get current branch
git branch --show-current

# Check which context file exists and count lines (AGENTS.md is primary; CLAUDE.md is legacy)
ls -la AGENTS.md CLAUDE.md 2>/dev/null
wc -l AGENTS.md 2>/dev/null
```

### 1.2 Determine Context-File Type

Read the context file (prefer `AGENTS.md`; fall back to `CLAUDE.md` only if that's all a repo has) and check for these markers:

| Marker Found | Type |
|--------------|------|
| `## ECOSYSTEM OVERVIEW` | Ecosystem Hub |
| `## REPOSITORY INDEX` | Ecosystem Hub |
| `> **Ecosystem Hub:**` | Satellite |
| None of above | Standard (single-repo) |
| No context file | None |

Also flag the **legacy state**: if only `CLAUDE.md` exists (no `AGENTS.md`), recommend the Step 0 migration so the repo joins the cross-tool ecosystem cleanly.

### 1.3 Scan for Related Repositories

Look for related repos:
- Check README.md for mentions of other repos
- Check the existing context file for repo references
- Look for common local paths (same parent directory)
- Parse organization from git remote URL

For each related repo found, check which context file it has and what type.

---

## STEP 2: SHOW ANALYSIS REPORT

**Always show this report before any action:**

```
┌─────────────────────────────────────────────────────────────┐
│ ECOSYSTEM CONTEXT-FILE ANALYSIS                             │
├─────────────────────────────────────────────────────────────┤
│ Organization: [org-name]                                    │
│ Current Repo: [repo-name]                                   │
│ Branch: [branch]                                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ THIS REPOSITORY:                                            │
│   Context file: [AGENTS.md / CLAUDE.md (legacy) / Missing]  │
│   Type: [Ecosystem Hub / Satellite / Standard / None]       │
│   Lines: [N]                                                │
│                                                             │
│ RELATED REPOSITORIES:                                       │
│   [repo-1]: [Hub ✅ / Satellite ✅ / Standard ⚠️ / None ❌]  │
│   [repo-2]: [Hub ✅ / Satellite ✅ / Standard ⚠️ / None ❌]  │
│   ...                                                       │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│ ARCHITECTURE STATUS: [✅ Complete / ⚠️ Partial / ❌ Missing]│
│                                                             │
│ [Details about what exists and what's missing]              │
├─────────────────────────────────────────────────────────────┤
│ RECOMMENDATION:                                             │
│                                                             │
│ [Specific actionable suggestion]                            │
└─────────────────────────────────────────────────────────────┘
```

---

## STEP 3: ASK USER (Interactive decision)

Based on analysis, ask the user which path to take:

### If no context file exists:

```
Question: "What would you like to create?"
Options:
- Ecosystem Hub (this is the main repo for the org)
- Satellite (this links to a hub elsewhere)
- Nothing (just wanted to check status)
```

### If a Standard context file exists:

```
Question: "Upgrade to ecosystem architecture?"
Options:
- Upgrade to Hub (add ecosystem sections, keep existing content)
- Convert to Satellite (lightweight, links to hub)
- Keep as-is (no changes)
```

### If an Ecosystem Hub exists:

```
Question: "Ecosystem Hub exists. What would you like to do?"
Options:
- Update entries (refresh repo info, versions, test counts)
- Nothing (just checking status)
```

### If a Satellite exists:

```
Question: "Satellite exists. What would you like to do?"
Options:
- Update content (refresh info)
- Nothing (just checking status)
```

### If Architecture is Complete:

```
Just report: "Architecture is complete. All repos have proper context files."
No action needed.
```

---

## STEP 4: EXECUTE (Only after user choice)

### 4.1 Commit Strategy

The context file lives in each repo as a regular file. `AGENTS.md` is the standard name; if a legacy `CLAUDE.md` exists, migrate it first (Step 0):

```bash
# Migrate legacy name if needed (AGENTS.md is the cross-tool standard)
if [ -f CLAUDE.md ] && [ ! -f AGENTS.md ]; then
  git mv CLAUDE.md AGENTS.md
fi

git add AGENTS.md
git commit -m "docs: update ecosystem context file"
git push
```

### 4.2 Creating an Ecosystem Hub

Add these sections at the top of `AGENTS.md`, preserving existing content below:

```markdown
<!-- Ecosystem hub context file — org-wide context for AI agents (Claude Code, Pi, any AGENTS.md-aware agent). Host-neutral. -->

# AGENTS.md - [Project] Ecosystem

**Organization:** https://github.com/[org]
**Website:** [url]
**Purpose:** Ecosystem-wide context for AI agents working across all [org] repositories

---

## ECOSYSTEM OVERVIEW

[Brief description]

### Related Repositories

| Repo | Purpose | Tech Stack | Version |
|------|---------|------------|---------|
| `org/repo1` | **Core** - [desc] | [stack] | vX.X.X |
| `org/repo2` | [desc] | [stack] | vX.X.X |

**Organization Mission:** [mission]

---

## CROSS-REPO STANDARDS

### Shared Coding Standards
[formatting, quality gates, git workflow]

### Shared AI Agent Guidelines

**DO:**
- [best practices]

**DON'T:**
- [anti-patterns]

### Licenses
[license info]

---

## REPOSITORY INDEX

### 1. [main-repo] (Core) - **YOU ARE HERE**

**Purpose:** [description]
**Tech Stack:** [tech]
**Key Commands:**
\`\`\`bash
[commands]
\`\`\`
**Key Files:**
- [files]

**Full details:** See "[PROJECT] CORE REPOSITORY" section below

---

### 2. [other-repo]

**Purpose:** [description]
**Tech Stack:** [tech]
**Key Commands:**
\`\`\`bash
[commands]
\`\`\`
**Key Files:**
- [files]

**Context file:** [link to repo's AGENTS.md]

---

## CURRENT FOCUS

See [ROADMAP.md](ROADMAP.md) for detailed tracking.

**Cross-Repo Coordination:**
- [notes]

---

# [PROJECT] CORE REPOSITORY

> **Note:** Sections below are specific to this repository.

[EXISTING CONTENT PRESERVED HERE]
```

### 4.3 Creating a Satellite

```markdown
<!-- Satellite context file — extends the ecosystem hub (link below). Host-neutral; repo-specific only. Do not duplicate hub standards here. -->

# AGENTS.md - [Repo Name]

> **Ecosystem Hub:** See [hub-repo/AGENTS.md](link) for full ecosystem context

**Repository:** https://github.com/[org]/[repo]
**Purpose:** [description]

---

## Quick Reference

**Tech Stack:** [tech]
**Deployment:** [if applicable]

**Key Commands:**
\`\`\`bash
[commands]
\`\`\`

---

## Key Files

| Path | Description |
|------|-------------|
| [path] | [desc] |

---

## Repo-Specific Guidelines

**DO:**
- [guidelines]

**DON'T:**
- [anti-patterns]

---

**Last Updated:** [date]
```

---

## DISCOVERY HELPERS

When gathering info for a repo:

**From package.json / Cargo.toml / pyproject.toml:**
- name, version, description
- scripts/commands

**From README.md:**
- Project purpose
- Related repos

**From directory structure:**
- Entry points (src/index.ts, main.py, cmd/main.go)
- Key directories

**From git:**
- Organization name (from remote URL)
- Current branch

---

## KEY PRINCIPLES

1. **`AGENTS.md` is the standard** — `CLAUDE.md` is CC's legacy name; migrate it (`git mv CLAUDE.md AGENTS.md`) so the file works across all agents, not just CC.
2. **Always safe** — detect first, ask before write
3. **User decides** — never assume, always ask
4. **Hub = full context** — an agent can work anywhere with just the hub
5. **Satellite = lightweight** — only repo-specific info, links to the hub
6. **No status fields** — avoid things that need constant updates
7. **Simple table** — `| Repo | Purpose | Tech Stack | Version |`
8. **Regular files** — the context file lives in each repo, committed to git normally

---

Alhamdulillah! Run this command anytime — it's safe and will guide you through building a clean, cross-tool context-file architecture for your whole organization. InshaAllah!
