# skills

Monorepo of **`@getpipher/*`** pi-packages and **framework-agnostic Agent Skills**. The same `SKILL.md` files power two install paths: [pi](https://pi.dev) packages for the full pi resource surface (extensions / skills / prompts / themes), and the open [Agent Skills](https://agentskills.io) standard for any of the 70+ agents via [`npx skills add`](https://github.com/vercel-labs/skills).

## Install

### Primary — pi packages (full pi surface)

Each family is its own npm package under the `@getpipher` scope:

```bash
pi install npm:@getpipher/git-tools
pi install npm:@getpipher/quality
# …see table below
```

Consume in `~/.pi/agent/settings.json`:

```json
{ "packages": ["npm:@getpipher/git-tools", "npm:@getpipher/quality"] }
```

### Complementary — `npx skills add` (skills only, any agent)

The skill *subset* is also reachable by any agent on the open Agent Skills standard (Claude Code, Codex, Cursor, OpenCode, pi, Windsurf, Goose, Copilot, Replit, Zed, …):

```bash
# all skills
npx skills add getpipher/skills

# a specific skill
npx skills add getpipher/skills --skill git-tools-create-issue

# to a specific agent
npx skills add getpipher/skills -a claude-code -g -y
```

For local pi development, point at this repo root (loads every family at once):

```json
{ "packages": ["/absolute/path/to/skills"] }
```

## Published packages

| Package | Family | Skills | Purpose |
|---|---|---:|---|
| [`@getpipher/git-tools`](skills/git-tools) | `git-tools` | 10 | Issue creation, labeling, PR audit, issue solving (GitHub & GitLab) |
| [`@getpipher/quality`](skills/quality) | `quality` | 5 | Lint/fix, TS strict, production checklist, multi-persona QA, roast |
| [`@getpipher/workspace`](skills/workspace) | `workspace` | 6 | Catch-up, AGENTS.md init, ecosystem context-files, doc org, handoff |
| [`@getpipher/work`](skills/work) | `work` | 1 | Lines-of-code counter (tokei) |
| [`@getpipher/design`](skills/design) | `design` | 1 | UI/UX analysis → AI-friendly prompts |
| [`@getpipher/solana-dev`](skills/solana-dev) | `solana-dev` | 1 | End-to-end Solana dev playbook |
| [`@getpipher/vps-deploy`](skills/vps-deploy) | `vps-deploy` | 1 | Docker/nginx/SSL/SSH/CI-CD for shared VPS |
| [`@getpipher/x-api`](skills/x-api) | `x-api` | 1 | X/Twitter API v2 reference + cost guard |

> `solana-defi/` (37 per-protocol SDK references) is deferred to a later release.

## Repository layout

```
skills/<family>/            # each family is a publishable @getpipher/<family> pi-package
  package.json              # pi manifest: { "skills": ["."] }  (also consumed by `npx skills add`)
  README.md
  LICENSE
  <skill>/SKILL.md          # Agent Skills standard (agentskills.io)
```

## License

MIT — shared as sadaqah jariyah.