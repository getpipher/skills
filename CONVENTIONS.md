# Harness-Agnostic Conventions — getpipher skills

Every SKILL.md in this monorepo (and in arsenal-private) must be executable by **any**
coding-agent harness — Claude Code, pi, omp, Cursor, Codex, or unknown future hosts.
The reading agent must never be assumed to run on a specific host.

Rulebook for writing and refining skills. Apply on every edit; violations are bugs.

## 1. Frontmatter

Keep ONLY:

```yaml
---
name: <family>-<skill>
description: <one-line trigger description>
---
```

- DROP `allowed-tools`, `argument-hint`, `user-invocable` — CC/pi slash-command metadata.
- Argument syntax goes in body prose: `Arguments: <repo-names> [--summary]` near the top.

## 2. Invocation contract

- NEVER use `$ARGUMENTS`, `$1`, `$2` or any host-substitution token.
- Refer to invocation args as "the arguments supplied when invoking this skill".
- In bash, use agent-assigned named variables:

```bash
REPOS="<repo names from the invocation arguments>"
for repo in $REPOS; do ...; done
```

- Usage examples: `Invoke with: git-tools-solve 1-5` — plain skill name, no slash prefix.

## 3. Sibling skill references

- Refer by frontmatter name in backticks: `workspace-init-agents`, `quality-roast`.
- NEVER `/git:solve`, `/workspace:init-agents`, `/stats:loc`, `/commit` — plugin-namespaced
  or bare slash syntax is CC-specific.

## 4. Capability probes, not host claims

- BAD: "pi has no sub-agents, so run sequentially." / "pi has no Chrome MCP."
- GOOD: "If your environment provides sub-agents, fan out in parallel; otherwise run the
  batches sequentially in this session."
- GOOD: "If a browser-automation tool is available, use it; otherwise ask the user to
  paste screenshots."

Never assert what a named host lacks or provides.

## 5. Tool vocabulary = capability classes

- BAD: "Use the Task tool", "Use Bash/Read/Write/Glob/Grep", "TodoWrite", "mcp__claude-in-chrome__*"
- GOOD: "spawn a sub-agent (if available)", "run via shell", "your file-read tool",
  "track progress in your response or task list, if your host provides one".

## 6. Paths: env var + host-neutral default

session-handoff's `$HANDOFF_VAULT` (default `~/.local/handoffs`) is the model.

- State/report dirs: `$QA_REPORT_DIR` → default `~/.local/share/qa-reports/`;
  `$LABEL_CACHE_DIR` → default `~/.cache/label-patterns.json`; etc.
- NEVER hardcode `~/.pi/...` or `~/.claude/...` as the only option.
- Exception (arsenal-private only): fallback chains that DISCOVER existing personal state
  ("first existing of ~/.claude/projects/.../memory, ~/.pi/agent/memory/..., else neutral
  default") are migration logic, not harness coupling. Document them as such.
- Exception (by skill subject): a skill whose subject IS a host (claude-code-statusline
  compat-update) may reference that host's paths.

## 7. Text generated into user repos must be host-neutral

Comments/headers a skill writes into files it creates (e.g. AGENTS.md satellite comments)
must not enumerate host paths. Say "extends your global agent context file", not
"`~/.claude/CLAUDE.md | ~/.pi/agent/AGENTS.md`".

## 8. Reasoning modes

- BAD: "mode: think", "ultrathink"
- GOOD: "If an extended-reasoning mode is available in your environment, use it for this step."

## 9. External reviewer bots

Only mention a review bot when configured:
"If `$REVIEWER_BOT` is set (a GitHub/GitLab user or app that reviews PRs), mention
`@$REVIEWER_BOT` in the description / address re-review requests to it."
No hardcoded bot handles.

## 10. Subject-matter allowlist (mentions that are FINE)

- `AGENTS.md` / `CLAUDE.md` as topics ("CLAUDE.md is Claude Code's legacy name for the
  same file") — factual, neutrally framed.
- Third-party products in reference content (Claude Desktop MCP config sections,
  model lists, "runs on Raspberry Pi").
- Sample outputs describing a hypothetical repo's AI-assistant config — genericize to
  "AI assistant configuration" where trivial, else acceptable.

## Rewrite style

- Minimal diff: fix findings, do not restructure, reformat, or restyle untouched content.
- Preserve behavior, section structure, Islamic openers where present.
- Keep frontmatter `name` stable unless a rename is explicitly requested.
