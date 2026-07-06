---
name: workspace-catch-up
description: Show repo mission, tech stack, current state, last 5 commits, and analyze uncommitted files
allowed-tools: ["bash", "read", "write", "edit"]
---

# Catch-Up: Complete Repository Context

Bismillah! Let me show you where we are and what we just did.

## Implementation Steps

Let me gather complete context for you:

1. **Extract Mission & Purpose**
   - Read the project context file — `AGENTS.md` (or `CLAUDE.md`, CC's legacy name) — first, for Repository Overview and Tech Stack
   - Fallback to README.md if no context file exists
   - Parse mission statement (why this repo exists)
   - Extract tech stack with reasons (how we achieve the mission)

2. **Show Current State**
   - Current branch and tracking status
   - Working tree status (clean/dirty)
   - Staged/unstaged file counts

3. **Display Recent Activity (Last 5 Commits)**
   - Commit hash, message, and time
   - Show commit graph for visual context

4. **Analyze Uncommitted Files (Smart Detection)**
   - For each uncommitted file:
     - Detect if it's auto-generated (lock files, build artifacts)
     - Check for WIP indicators (TODO, FIXME, console.log, commented code)
     - Analyze change size and nature (new file vs modified)
     - Categorize as:
       - ✅ **Forgotten Commits**: Complete work ready to commit
       - 🚧 **Work in Progress**: Unfinished work with WIP indicators
       - 📦 **Generated Files**: Auto-generated (lockfiles, builds)
       - ⚠️ **Should be Ignored**: Belongs in .gitignore

5. **Provide Concise Summary**
   - Quick overview of repo mission
   - Current status (clean/dirty)
   - Action items based on uncommitted file analysis

## Extraction Logic

### Mission & Purpose
- Check the context file (AGENTS.md) for these sections:
  - `## Repository Overview`
  - `## Architecture and Structure`
  - `## Technology Stack`
- If not found, check README.md:
  - First paragraph after title
  - Features/Goals section
  - Tech Stack section
- If no docs found: Infer from file structure and suggest creating an `AGENTS.md` (try `/workspace:init-agents`)

### Smart File Analysis Patterns

**Auto-generated files:**
- `*lock.json`, `*.lock`, `Cargo.lock`, `Gemfile.lock`, `poetry.lock`
- `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`
- `dist/`, `build/`, `target/`, `.next/`, `.nuxt/`
- `.DS_Store`, `Thumbs.db`

**WIP Indicators:**
- Comments: `TODO`, `FIXME`, `HACK`, `XXX`, `WIP`, `NOTE`
- Debug code: `console.log`, `print(`, `debugger`, `dd(`
- Commented-out code blocks (>3 lines)
- Incomplete functions (missing return statements, empty blocks)

**Config changes (likely forgotten commits):**
- Small modifications (<20 lines) to `.json`, `.yaml`, `.toml`, `.conf`
- Version bumps in package.json
- Dependency updates without other changes

**Should be ignored:**
- `.env`, `.env.local`, `.env.*.local`
- `.cache`, `tmp/`, `temp/`, `.tmp/`
- IDE files: `.idea/`, `.vscode/`, `*.swp`
- OS files: `.DS_Store`, `Thumbs.db`

## Output Format

```
═══════════════════════════════════════════════════════
📍 CATCH-UP: [repo name]
═══════════════════════════════════════════════════════

🎯 MISSION & PURPOSE
   [2-3 sentence summary from docs]

   Goal: [Primary objective]

   Key Benefit: [Main value proposition]

🛠️  TECH STACK & APPROACH
   • [Tool/Framework]
     Why: [Reason for choosing this]

   • [Tool/Framework]
     Why: [Reason for choosing this]

   [Max 4-5 key technologies]

🌿 CURRENT STATE
   Branch: [branch-name] ([X files modified, Y untracked])
   Last activity: [time since last commit]

📝 RECENT ACTIVITY (Last 5 commits)
   [hash] [message]
   [hash] [message]
   [hash] [message]
   [hash] [message]
   [hash] [message]

⚠️  UNCOMMITTED CHANGES ([X files])

   [If no uncommitted files:]
   ✅ Working tree is clean - no uncommitted changes

   [If uncommitted files exist, categorize:]

   Forgotten Commits ([count]):
   ✅ [file-path] ([+lines/-lines])
      → [Analysis: why it's a forgotten commit]

   Work in Progress ([count]):
   🚧 [file-path] ([+lines/-lines])
      → [Analysis: what's incomplete, found WIP indicators]

   Generated Files ([count]):
   📦 [file-path] ([+lines/-lines])
      → [Type of generated file, e.g., lockfile, build artifact]

   Should be Ignored ([count]):
   ⚠️ [file-path]
      → [Reason it should be in .gitignore]

💡 SUMMARY
   Mission: [One-line mission statement]
   Status: [Clean/Active development/Pending commits]
   Action: [Specific next steps based on analysis]
```

## Usage Examples

```bash
# Get complete context when returning to a project
/catch-up

# Use when:
# - You just opened the project after time away
# - You forgot what you were working on
# - You need to understand the repo quickly
# - You want to check if you forgot to commit something
```

## Error Handling

- If not in a git repository: "Not a git repository. Run this command from a git repo."
- If neither a context file (AGENTS.md) nor README.md exists: Show inference + suggest creating docs
- If no commits exist: "New repository with no commits yet"
- If git commands fail: Show clear error message with suggested fix

---

Alhamdulillah! This command provides complete context restoration in a concise, scannable format.
