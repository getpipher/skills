---
name: workspace-session-handoff
description: Use when ending a session and work needs to continue in a new session (Claude Code or Pi). Creates a self-contained starter prompt with full context so the next session picks up seamlessly. Host-neutral — the handoff bridges sessions across either host (shared workspace + vault).
allowed-tools: ["bash", "read", "write", "edit"]
---

# Session Handoff

Create a starter prompt file that a fresh session can use to continue work without context loss.

> **Host-neutral.** This skill works the same in Claude Code and Pi. The handoff file lives in the shared `$HANDOFF_VAULT/{project}/` vault (default `$HANDOFF_VAULT`=`~/.local/handoffs` — a local directory; override via the `HANDOFF_VAULT` env var to point at your own private/synced vault), so a session in either host can write it and the next session (same host or the other) can resume — they share the workspace and the vault. Don't assume the next session runs in the same host as this one.

## When to Use

- End of a long session with pending tasks
- Before a context-compaction / session switch (CC: `/compact`; pi: `/reload` or restart) when work remains
- When context window is getting full and work remains
- User says "save progress", "wrap up", "continue later", "new session"

## Output

Write to: `$HANDOFF_VAULT/{project}/session-handoff-{YYYY-MM-DD}.md` (default `$HANDOFF_VAULT`=`~/.local/handoffs`)

If a handoff file already exists for today, append a letter suffix (e.g., `-b`, `-c`).

## Implementation

### Step 1: Gather Context

Collect from the current conversation (do NOT re-read files you already have context for):

1. **What was completed** — list every change shipped this session (commits, deploys, fixes)
2. **What's pending** — unfinished tasks, next steps, blockers
3. **Key decisions made** — architectural choices, approach decisions, trade-offs discussed
4. **Active plan/spec** — file path to any implementation plan or design spec in progress
5. **Known issues** — bugs found, workarounds applied, tech debt noted
6. **File paths touched** — specific files modified (helps the next session navigate)
7. **CI/deploy state** — are all repos green? anything deploying?
8. **VPS/infra state** — any server-side changes made (env vars, configs, deploys)

### Step 2: Decide if a progress table is needed

A **progress table** sits near the top of the handoff (between the header block and the Starter Prompt) and captures discrete units of long-running work — phases, slices, tasks, milestones — with status + any associated PR/issue numbers. Fastest way for a new session to scan "what's done / what's next" without reading the full Session Log.

**Generate a progress table when ANY apply:**

1. **Predecessor handoff has one** — `ls -t "$HANDOFF_VAULT"/{project}/session-handoff-*.md | head -1` contains `## Progress Table`. Chain consistency > per-session re-evaluation.
2. **User explicitly requested** — prompt contains "MUST include progress table" or "include the progress table".
3. **Work has numbered discrete units** — multi-phase plan with slices/tasks (e.g., "Slice 5.7 done, next 5.8"), or stacked PR chain where order matters.
4. **More than 3 PRs/issues stacked across sessions** — flat bullet list becomes unscannable.

**Skip the progress table when:**

- Single-session bug fix or one-off with no follow-up sessions planned.
- Continuing work without discrete numbered phases (open-ended exploration, content writing).
- Previous chain handoff doesn't have one and adding it would break the convention.

**Heuristic:** can't articulate at least 5 rows that belong in the table → don't need it.

### Step 3: Gather progress table data (if Step 2 says yes)

1. **Find the most recent predecessor handoff** for this project:

   ```bash
   ls -t "$HANDOFF_VAULT"/{project}/session-handoff-*.md 2>/dev/null | head -1
   ```

   Locate its `## Progress Table` section.

2. **Copy the table verbatim as the baseline** — same column headers, same row order. Preserves chain consistency.

3. **Apply this session's changes:**
   - Flip newly-completed rows from `[ ] TODO` / `[ ] NOT STARTED` to `[x] DONE`.
   - Move the `(★ next)` marker from the row that just shipped to the row this handoff recommends next.
   - Add the PR number to the newly-completed row.
   - Append new rows at the bottom for any newly discovered units of work.
   - **Bold the row(s) this session shipped** so they're visually obvious.

4. **Update the footer summary line** below the table — increment stack counts, test counts, clippy state, phase progress (e.g., "Phase 5: 7/10 slices done"). The footer is what the next session uses to sanity-check it's resuming from the right baseline.

**If no predecessor exists** (first handoff in chain), author a fresh table from the project's plan/spec. Default columns:

| Phase | Slice | Headline change | Status | PR |
|---|---|---|---|---|

Adapt column names to the project's vocabulary ("Milestone" instead of "Phase", "Issue" instead of "PR", etc.). Lock the column shape in this first handoff — every subsequent handoff in the chain inherits it.

### Step 4: Write the Handoff File

Structure (the `## Progress Table` section is inserted only if Steps 2–3 produced one):

```markdown
# {Project} — Session Handoff ({date}-{letter} — {one-line headline})

**Date:** {today}
**Working directory:** {cwd}
**Session summary:** {one-line summary of what this session accomplished}

---

## Progress Table

{Rendered from Step 3's data. Include footer line with stack count + test count + clippy state + phase progress. Skip this entire section if Step 2 says no.}

---

## Starter Prompt

Copy-paste this into a new session (CC or pi):

\```
{OPENER-ECHO — include ONLY when a progress table was generated (Steps 2–3); omit entirely otherwise. Embed the table verbatim so it travels with the prompt even if the user pastes only this block:}

FIRST — before reading any plan, touching any code, or taking any other action — render the progress table below back to me verbatim, under the heading "## Resuming — where we left off". Then continue with the task.

{full progress table copied verbatim from the ## Progress Table section above, including the footer summary line}

---

{TASK PROMPT — always included; self-contained, no dependence on this conversation:
- What to do next (specific task/plan to execute)
- Path to any plan/spec file to read
- Key context the new session needs (decisions, constraints, gotchas)
- Build/test commands to verify state
- Any rules to follow (commit style, testing requirements, etc.)
}
\```

---

## Session Log

### Completed
{Bulleted list of everything done}

### Pending
{Bulleted list of what remains}

### Key Decisions
{Decisions made with reasoning — the next session needs this context}

### Known Issues
{Bugs, workarounds, fragile spots}

### Infrastructure Changes
{VPS, env vars, deploys — anything server-side}
```

### Step 5: Copy handoff path to clipboard

Run the handoff file path through the OS clipboard so the user can paste it directly into a new session without re-typing.

macOS (primary target):

```bash
printf '%s' "{path}" | pbcopy
```

Cross-platform fallback (use this form if unsure of the environment):

```bash
printf '%s' "{path}" | (pbcopy 2>/dev/null || xclip -selection clipboard 2>/dev/null || wl-copy 2>/dev/null || true)
```

Verify the copy succeeded by reading it back:

```bash
pbpaste  # macOS — expected: the handoff path
```

If the clipboard tool isn't available (e.g., remote SSH session without clipboard forwarding), skip silently — don't treat it as an error. The Confirm step still shows the path as text.

### Step 6: Update MEMORY.md

Add a pointer to the handoff file and update any stale progress entries.

### Step 7: Confirm

Tell the user:

> Handoff saved to `{path}` (already copied to clipboard). Start a new session in `{cwd}` and paste the path — or paste the starter prompt directly.

**If Step 2 produced a progress table, also echo it inline as a closer.** Render the table verbatim from the handoff file as a final progress snapshot, prefixed with a one-line header. Example:

> ## Final progress before new session
>
> {progress table rendered verbatim, including the footer summary line}

This gives the user a screenshot-friendly closer without requiring them to open the file.

The new session performs the mirror action on arrival: the starter prompt (Step 4) instructs it to render this same table **first**, before any work. Closer on departure, opener on arrival.

## Rules

- **Self-contained prompt** — the new session should NOT need to read this conversation
- **No secrets in the prompt** — API keys, passwords, keypairs stay out
- **Specific over general** — "execute Task 3 in docs/plans/foo.md" beats "continue the work"
- **Include gotchas** — anything surprising or non-obvious that would trip up a fresh session
- **Private vault only** — `$HANDOFF_VAULT` (default `~/.local/handoffs`) should be a private/encrypted location, safe for strategy context. Override `$HANDOFF_VAULT` to your own private vault (e.g. an iCloud-synced or encrypted dir).
- **Progress table is part of the chain contract** — once a chain starts using a progress table, every handoff in the chain MUST include + update it. Skipping breaks scannability for the next session and forces a rebuild from prior handoffs.
- **Progress table echo is the closer, not the full handoff** — Step 7's inline table is a snapshot for the current chat. The handoff file (`.md`) is the durable artifact. Don't paraphrase or trim the table when echoing — render it verbatim.
- **Opener-echo mirrors the closer** — when a progress table exists, the starter prompt's FIRST instruction MUST be "render the table before doing anything." Embed the table inside the starter-prompt block (Step 4), not as a file reference, so it survives the user pasting the prompt alone. The fresh session renders it as its first action — a "where we left off" snapshot symmetric to Step 7's "where we ended" closer. No table generated → no opener-echo.
