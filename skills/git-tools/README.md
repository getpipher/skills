# @getpipher/git-tools

Git workflow skills for [pi](https://github.com/earendil-works/pi-coding-agent) — issue creation, intelligent labeling, PR auditing, and issue solving across GitHub and GitLab.

## Skills

| Skill | Purpose |
|---|---|
| `git-tools-commit` | Commit and push current changes to remote |
| `git-tools-create-issue` | Comprehensive issue creation (GitHub/GitLab) with intelligent questioning, labels, type assignment |
| `git-tools-create-pr` | Create a PR (GitHub) or MR (GitLab) from one branch to another |
| `git-tools-get-pr-review` | Retrieve and analyze PR/MR review feedback to improve code quality |
| `git-tools-label-issues` | Intelligently analyze and label unlabeled GitHub issues (any repo) |
| `git-tools-local-sync` | Fetch all remotes, update all branches (dual-remote aware) |
| `git-tools-merge-pr` | Merge a PR/MR by number or URL with configurable merge strategy |
| `git-tools-pr-audit` | Review open PRs, merge sequentially with rebase, handle conflicts (org-aware) |
| `git-tools-solve` | Create conventional branch, tackle issues serially, create PR |
| `git-tools-standup` | Morning standup — PRs, issues, activity digest across GitHub orgs |

## Install

```bash
pi install npm:@getpipher/git-tools
```

Requires [pi](https://github.com/earendil-works/pi-coding-agent). Org-aware skills (`pr-audit`, `solve`, `label-issues`) read your `approvedOrgs`/`owned_orgs` from config — no orgs are assumed by default.

## License

MIT
