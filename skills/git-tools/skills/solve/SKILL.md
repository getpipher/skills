---
name: git-tools-solve
description: Issue solver — creates conventional branch (feat/fix/chore), tackles issues serially, creates PR
argument-hint: "[issues] [--org] [--pr] [--continue] [--github|--gitlab]"
allowed-tools: ["bash", "read", "write", "edit"]
---

# Solve - Conventional Branch Workflow

Bismillah! I'll create a conventional branch, tackle issues serially, and create a PR at session end.

Arguments provided: $ARGUMENTS

---

## Branch Naming Convention

Branch names are derived from the **first issue's labels and title**:

| Label | Prefix | Example |
|-------|--------|---------|
| `bug` | `fix/` | `fix/stealth-address-validation` |
| `docs`, `documentation` | `docs/` | `docs/update-sdk-readme` |
| `chore`, `maintenance` | `chore/` | `chore/upgrade-dependencies` |
| `refactor` | `refactor/` | `refactor/auth-module` |
| _(default)_ | `feat/` | `feat/add-rate-limiting` |

```
base (main) ──┬── feat/add-rate-limiting → PR (closes #101, #104)
              ├── fix/stealth-validation → PR (closes #107)
              └── chore/upgrade-deps → PR (closes #112)
```

---

## Arguments

```bash
/git:solve                     # Interactive: shows available issues
/git:solve 101,104,107         # Pre-assigned issues
/git:solve --org               # Show issues across ALL org repos
/git:solve your-app#55,101      # Cross-repo issues (your-app#55 + current#101)
/git:solve --continue           # Resume current branch session
/git:solve --pr                 # Create PR for current branch (skip work)
/git:solve --gitlab             # Use GitLab instead of GitHub
```

**Parsing:**
```bash
# Extract issues (comma-separated, may include repo#issue format)
ISSUES=$(echo "$ARGUMENTS" | grep -oE '[a-zA-Z0-9_-]*#?[0-9]+(,[a-zA-Z0-9_-]*#?[0-9]+)*' | head -1)

# Flags
CONTINUE_MODE=false && [[ "$ARGUMENTS" == *"--continue"* ]] && CONTINUE_MODE=true
PR_ONLY=false && [[ "$ARGUMENTS" == *"--pr"* ]] && PR_ONLY=true
GITLAB=false && [[ "$ARGUMENTS" == *"--gitlab"* ]] && GITLAB=true
ORG_MODE=false && [[ "$ARGUMENTS" == *"--org"* ]] && ORG_MODE=true

CLI="gh" && [[ "$GITLAB" == true ]] && CLI="glab"

# Detect org from current repo
ORG_NAME=$(echo "$(git remote get-url origin 2>/dev/null)" | sed -E 's#(git@|https://)([^:/]+)[:/]([^/]+)/.*#\3#')
```

---

## Phase 0: Pre-Flight

**Validate environment and detect context:**

```bash
# Verify git repo
if ! git rev-parse --is-inside-work-tree &>/dev/null; then
    echo "ERROR: Not in a git repository"
    exit 1
fi

# Verify CLI authentication
if ! $CLI auth status &>/dev/null; then
    echo "ERROR: $CLI is not authenticated. Run '$CLI auth login' first."
    exit 1
fi

# Detect base branch (main > master > dev > develop)
BASE_BRANCH=""
for branch in main master dev develop; do
    if git show-ref --verify --quiet refs/remotes/origin/$branch 2>/dev/null; then
        BASE_BRANCH=$branch
        break
    fi
done

# Fetch latest
git fetch origin --prune

# Get repo info
REPO_NAME=$(basename "$(git rev-parse --show-toplevel)")
REPO_ROOT=$(git rev-parse --show-toplevel)
```

**Display pre-flight summary:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PRE-FLIGHT COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Repo:      {REPO_NAME}
Base:      origin/{BASE_BRANCH}
Platform:  {github|gitlab}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Phase 1: Issue Selection

**If --pr flag, skip to PR creation:**
```bash
if [[ "$PR_ONLY" == true ]]; then
    # Jump to Phase 6: PR Creation
fi
```

**If --continue flag, resume current branch:**
```bash
if [[ "$CONTINUE_MODE" == true ]]; then
    WORK_BRANCH=$(git branch --show-current)
    if [[ "$WORK_BRANCH" =~ ^(feat|fix|chore|docs|refactor)/ ]]; then
        echo "Resuming session on branch: $WORK_BRANCH"
        # Skip to Phase 3: Work Loop
    else
        echo "ERROR: Current branch '$WORK_BRANCH' is not a conventional branch"
        exit 1
    fi
fi
```

**If issues not provided, show available issues (WIP filtered):**

```bash
if [[ -z "$ISSUES" ]]; then
    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "AVAILABLE ISSUES (excluding WIP)"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

    if [[ "$ORG_MODE" == true ]]; then
        # Show issues across ALL org repos
        echo "Organization: $ORG_NAME"
        echo ""

        ORG_REPOS=$(gh repo list "$ORG_NAME" --json name --limit 100 --jq '.[].name' | grep -v '^\.github$')

        for repo in $ORG_REPOS; do
            issues=$(gh issue list --repo "$ORG_NAME/$repo" --state open \
                --json number,title,labels,assignees --limit 20 2>/dev/null || echo "[]")

            # Filter out WIP issues (including has-pr)
            available=$(echo "$issues" | jq '
                [.[] | select(
                    (.assignees | length == 0) and
                    ([.labels[]?.name // "" | ascii_downcase] |
                        all(. != "wip" and . != "in progress" and . != "in-progress" and . != "doing" and . != "has-pr"))
                )]
            ')

            avail_count=$(echo "$available" | jq 'length')

            if [[ "$avail_count" -gt 0 ]]; then
                echo "--- $repo ($avail_count available) ---"
                echo "$available" | jq -r '.[] | "  #\(.number) \(.title[:50])"'
                echo ""
            fi
        done
    else
        # Current repo only
        echo "Repo: $REPO_NAME"
        echo ""

        issues=$(gh issue list --state open --json number,title,labels,assignees --limit 50)

        # Display all issues with WIP status
        echo "$issues" | jq -r '
            .[] |
            (
                if .assignees | length > 0 then "⏳ [assigned]  "
                elif [.labels[]?.name // "" | ascii_downcase] | any(. == "has-pr") then "⏳ [has-pr]    "
                elif [.labels[]?.name // "" | ascii_downcase] | any(. == "wip" or . == "in progress" or . == "in-progress" or . == "doing") then "⏳ [WIP]       "
                else "✓  [available] "
                end
            ) as $status |
            "  #\(.number | tostring | ("    " + .)[-4:])  \($status) \(.title[:45])"
        '
    fi

    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "Legend: ✓ available | ⏳ WIP (assigned/in-progress/has-pr)"
    echo ""
    echo "Cross-repo format: repo#issue (e.g., your-app#55,101)"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

    # Ask the user which issues to tackle, e.g. "Which issues? 101,104,107 or your-app#55,101"
fi

# Parse issues into array (supports cross-repo format)
parse_issue_refs() {
    local input="$1"
    local default_repo="$2"

    IFS=',' read -ra parts <<< "$input"

    for part in "${parts[@]}"; do
        part=$(echo "$part" | tr -d ' ')
        if [[ "$part" == *"#"* ]]; then
            echo "${part%#*}:${part#*#}"
        else
            echo "$default_repo:$part"
        fi
    done
}

ISSUE_REFS=$(parse_issue_refs "$ISSUES" "$REPO_NAME")
```

---

## Phase 2: Branch Setup

**Derive branch name from first issue:**

```bash
# Fetch first issue details for branch naming
FIRST_ISSUE=$(echo "$ISSUE_REFS" | head -1 | cut -d: -f2)
FIRST_ISSUE_DATA=$($CLI issue view $FIRST_ISSUE --json title,labels)
FIRST_ISSUE_TITLE=$(echo "$FIRST_ISSUE_DATA" | jq -r '.title')
FIRST_ISSUE_LABELS=$(echo "$FIRST_ISSUE_DATA" | jq -r '.labels | map(.name) | join(", ")')

# Determine branch prefix from labels
if [[ "$FIRST_ISSUE_LABELS" == *"bug"* ]]; then
    BRANCH_PREFIX="fix"
elif [[ "$FIRST_ISSUE_LABELS" == *"docs"* ]] || [[ "$FIRST_ISSUE_LABELS" == *"documentation"* ]]; then
    BRANCH_PREFIX="docs"
elif [[ "$FIRST_ISSUE_LABELS" == *"chore"* ]] || [[ "$FIRST_ISSUE_LABELS" == *"maintenance"* ]]; then
    BRANCH_PREFIX="chore"
elif [[ "$FIRST_ISSUE_LABELS" == *"refactor"* ]]; then
    BRANCH_PREFIX="refactor"
else
    BRANCH_PREFIX="feat"
fi

# Slugify title for branch name
BRANCH_SLUG=$(echo "$FIRST_ISSUE_TITLE" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//' | cut -c1-50)

WORK_BRANCH="${BRANCH_PREFIX}/${BRANCH_SLUG}"

# Ensure we're on base branch, up to date
git checkout "$BASE_BRANCH"
git pull origin "$BASE_BRANCH"

# Create and switch to work branch
git checkout -b "$WORK_BRANCH"
```

**Display:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
BRANCH READY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Branch:    {WORK_BRANCH}
Base:      origin/{BASE_BRANCH}
Issues:    {ISSUE_REFS}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Phase 3: Work Loop (Per Issue, Serial)

**Install dependencies first (if needed):**
```bash
# Detect and run install
if [[ -f "pnpm-lock.yaml" ]]; then
    pnpm install
elif [[ -f "yarn.lock" ]]; then
    yarn install
elif [[ -f "package.json" ]]; then
    npm install
fi
```

**For each issue in ISSUE_REFS:**

### 3a. Fetch Issue Details

```bash
ISSUE_NUM=$current_issue
ISSUE_DATA=$($CLI issue view $ISSUE_NUM --json number,title,body,labels,state)
ISSUE_TITLE=$(echo "$ISSUE_DATA" | jq -r '.title')
ISSUE_BODY=$(echo "$ISSUE_DATA" | jq -r '.body')
ISSUE_LABELS=$(echo "$ISSUE_DATA" | jq -r '.labels | map(.name) | join(", ")')
```

**Display:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ISSUE #$ISSUE_NUM
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Title:  $ISSUE_TITLE
Labels: $ISSUE_LABELS

$ISSUE_BODY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 3b. Implement Solution

- Analyze issue requirements
- Explore relevant code
- Plan implementation (outline steps inline)
- Write code following project conventions
- NO PR yet — just commit to work branch

### 3c. Test

```bash
# Detect and run tests
if [[ -f "pnpm-lock.yaml" ]]; then
    pnpm test --run
elif [[ -f "package.json" ]]; then
    npm test
elif [[ -f "Cargo.toml" ]]; then
    cargo test
elif [[ -f "pyproject.toml" ]]; then
    pytest
fi
```

### 3d. Commit

**Determine commit type from labels:**
```bash
if [[ "$ISSUE_LABELS" == *"bug"* ]]; then
    COMMIT_TYPE="fix"
elif [[ "$ISSUE_LABELS" == *"docs"* ]] || [[ "$ISSUE_LABELS" == *"documentation"* ]]; then
    COMMIT_TYPE="docs"
elif [[ "$ISSUE_LABELS" == *"chore"* ]] || [[ "$ISSUE_LABELS" == *"maintenance"* ]]; then
    COMMIT_TYPE="chore"
else
    COMMIT_TYPE="feat"
fi
```

**Create commit:**
```bash
git add .
git commit -m "$COMMIT_TYPE: $ISSUE_TITLE (closes #$ISSUE_NUM)"
```

### 3e. Push (Incremental)

```bash
git push -u origin "$WORK_BRANCH"
```

### 3f. Update Progress

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ISSUE #$ISSUE_NUM COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Commit:    $COMMIT_HASH
Remaining: X issues
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Ask: Continue to next issue?**
- [Continue] → Move to next issue
- [Skip] → Skip current, move to next
- [Stop] → End work phase, go to PR phase

---

## Phase 4: Session Summary

**After all issues or when stopped:**

```bash
# Get all commits since base
COMMITS=$(git log "origin/$BASE_BRANCH..$WORK_BRANCH" --oneline)
COMMIT_COUNT=$(echo "$COMMITS" | wc -l | tr -d ' ')
ISSUES_CLOSED=$(echo "$COMMITS" | grep -oE '#[0-9]+' | sort -u | tr '\n' ' ')
```

**Display:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SESSION SUMMARY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Branch:      $WORK_BRANCH
Commits:     $COMMIT_COUNT
Issues:      $ISSUES_CLOSED

Commits:
$COMMITS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Phase 5: PR Decision

**Ask user:**
- [Create PR now] → Create PR to base branch
- [Later] → End session, can run `/git:solve --pr` later
- [Continue working] → Add more issues

---

## Phase 6: PR Creation

```bash
# Ensure pushed
git push origin "$WORK_BRANCH"

# Get issues for title
ISSUES=$(git log "origin/$BASE_BRANCH..$WORK_BRANCH" --oneline | grep -oE '#[0-9]+' | sort -u | tr '\n' ' ')
COMMIT_LOG=$(git log "origin/$BASE_BRANCH..$WORK_BRANCH" --oneline)

# Build PR title from branch prefix + issues
PR_TITLE="${BRANCH_PREFIX}: ${FIRST_ISSUE_TITLE} (${ISSUES})"

# Create PR
$CLI pr create \
    --base "$BASE_BRANCH" \
    --head "$WORK_BRANCH" \
    --title "$PR_TITLE" \
    --body "$(cat <<EOF
## Summary

${ISSUES:-No issue references found}

## Commits

\`\`\`
$COMMIT_LOG
\`\`\`

---
Created via \`/git:solve\`
EOF
)"

PR_URL=$($CLI pr view --json url -q '.url')

# Mark issues with has-pr label
ISSUE_NUMS=$(echo "$COMMITS" | grep -oE '#[0-9]+' | tr -d '#' | sort -u)

for issue_num in $ISSUE_NUMS; do
    $CLI issue comment $issue_num --body "Being addressed in $PR_URL — will auto-close on merge"
    $CLI issue edit $issue_num --add-label "has-pr" 2>/dev/null || true
done
```

---

## Phase 7: Done

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SOLVE COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Branch:    $WORK_BRANCH
Issues:    $ISSUES
PR:        $PR_URL
Target:    $BASE_BRANCH

Ready for /git:pr-audit to review and merge.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Organization-Wide Examples

```bash
# Show issues across all org repos (with WIP filtering)
/git:solve --org
# Lists available issues from your-repo, your-website, your-app, etc.

# Work on issues from multiple repos
/git:solve your-app#55,your-website#30,101
# Works on:
#   - your-app issue #55
#   - your-website issue #30
#   - your-org/your-repo (current repo) issue #101

# Cross-repo audit after solving
/git:pr-audit --org
# Reviews PRs across ALL org repos
```

**WIP Filtering in Action:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
AVAILABLE ISSUES (excluding WIP)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

--- your-repo (5 available) ---
  #101 Add rate limiting to API endpoints
  #104 Fix stealth address validation
  #107 Update SDK documentation

--- your-app (2 available) ---
  #55 Implement dark mode toggle
  #58 Add wallet connection retry logic

--- your-website (1 available) ---
  #30 Update homepage hero section

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Legend: ✓ available | ⏳ WIP (assigned/in-progress/has-pr)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Edge Cases

**Issue already has a PR:**
- Check for existing PRs referencing the issue
- Warn if found
- Ask: Skip or work anyway?

**Tests failing on unrelated code:**
- Document in commit message
- Continue with warning
- Note in PR description

**Branch already exists:**
- If remote branch with same name exists, ask: resume or create new?
- Resume: `git checkout $WORK_BRANCH && git pull`
- New: append issue number suffix (e.g., `feat/add-rate-limiting-101`)

---

## Recovery

**Resume interrupted session:**
```bash
/git:solve --continue
```

**View branch status:**
```bash
git fetch origin
git log origin/$BASE_BRANCH..$WORK_BRANCH --oneline
```

**Sync branch with base:**
```bash
git fetch origin $BASE_BRANCH
git rebase origin/$BASE_BRANCH
```

---

*"Ship working code, not perfect code. But never ship broken code."*

Alhamdulillah! Starting solve now...
