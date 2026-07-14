<!-- Satellite context file — extends the global hub (~/.claude/CLAUDE.md | ~/.pi/agent/AGENTS.md). Host-neutral; project-specific only. Do not duplicate hub standards here. -->

# skills (getpipher arsenal)

> Monorepo of `@getpipher/*` pi-packages and framework-agnostic Agent Skills. The same `SKILL.md` files power two install paths: [pi](https://pi.dev) packages (full pi resource surface — extensions/skills/prompts/themes) and the open [Agent Skills](https://agentskills.io) standard for any of the 70+ agents via `npx skills add`.

**Org context:** getpipher is the Pi coding-agent ecosystem. No GitLab mirror for getpipher.

## Install

### Primary — pi packages (full pi surface)

Each family is its own npm package under the `@getpipher` scope:

```bash
pi install npm:@getpipher/git-tools
pi install npm:@getpipher/quality
# …see the family table below
```

Consume in `~/.pi/agent/settings.json`:

```json
{ "packages": ["npm:@getpipher/git-tools", "npm:@getpipher/quality"] }
```

### Complementary — `npx skills add` (skills only, any agent)

```bash
npx skills add getpipher/skills                       # all skills
npx skills add getpipher/skills --skill git-tools-create-issue   # a specific skill
npx skills add getpipher/skills -a claude-code -g -y   # to a specific agent
```

### Local pi development

Point at this repo root (loads every family at once) via the local `packages` path in `~/.pi/agent/settings.json`.

## Structure

```
skills/<family>/      # each family is a publishable @getpipher/* package
  skills/<name>/SKILL.md   # on-demand skill files (read by pi when invoked)
```

## Skill families

git-tools · quality · workspace · work · design · solana-dev · vps-deploy · x-api · browser — see each package's README for the full skill list.

## Notes

- Skills are read on demand by pi — no build step for the skill files themselves.
- Each family package is published to npm under the `@getpipher` scope.