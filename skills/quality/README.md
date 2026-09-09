# @getpipher/quality

Code quality skills for any coding agent — linting/fixing, TypeScript strict checks, production-readiness audit, multi-persona QA, and a brutal pre-prod code roast.

## Skills

| Skill | Purpose |
|---|---|
| `quality-lint-fix` | Run linter and auto-fix all issues in the project |
| `quality-production-checklist` | Analyze any codebase for production readiness — security, performance, deployment |
| `quality-qa` | Multi-persona QA — dev-engineer code review + end-user UX critique |
| `quality-roast` | Brutal pre-production code audit that roasts lazy shortcuts and red flags |
| `quality-type-check-strict` | Run TypeScript strict checks and create a plan to fix all errors |

## Install

```bash
# Install as a pi package
pi install npm:@getpipher/quality

# Or add the skills to any coding agent via the skills CLI
npx skills add getpipher/quality
```

Works with any coding agent. `quality-qa` degrades gracefully — sequential personas when sub-agents aren't available, user-provided screenshots when no browser tool is.

## License

MIT
