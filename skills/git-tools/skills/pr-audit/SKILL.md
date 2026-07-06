---
name: git-tools-pr-audit
description: PR auditor — reviews open PRs, merges sequentially with rebase, handles conflicts (auto org-aware)
argument-hint: "[--org] [--single] [--quick] [--dry-run] [--github|--gitlab] [PR#]"
allowed-tools: ["bash", "read", "write", "edit"]
---

# PR Audit - Conventional Branch Workflow

Bismillah! I'll audit open PRs (feat/fix/chore → base), merge sequentially with rebase between merges, and handle conflicts cleanly.

Arguments provided: $ARGUMENTS

---

## Audit Philosophy

**Sequential merge with rebase.**

PRs are merged one at a time. After each merge, remaining PRs are rebased on the updated base to prevent conflict accumulation.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  AUDIT FLOW                                                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  DISCOVERY:                                                                 │
│    feat/add-rate-limiting → main (3 issues, +450/-120)                     │
│    fix/stealth-validation → main (1 issue, +200/-50)                       │
│    chore/upgrade-deps → main (1 issue, +80/-10)                            │
│                                                                             │
│  MERGE ORDER:                                                               │
│    1. Review feat/add-rate-limiting → Merge ✓                              │
│    2. Rebase fix/stealth-validation on new base → Review → Merge ✓         │
│    3. Rebase chore/upgrade-deps on new base → Review → Merge ✓             │
│                                                                             │
│  RESULT: 3 clean merges, conflicts resolved incrementally                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Arguments

```bash
/git:pr-audit                  # Auto-detect: org mode if in org repo, else single repo
/git:pr-audit --org            # Force org mode (scan ALL repos in organization)
/git:pr-audit --org your-org  # Audit specific organization
/git:pr-audit --single         # Force single-repo mode (override auto-org detection)
/git:pr-audit --quick          # Fast mode: merge all passing PRs without review
/git:pr-audit --dry-run        # Preview mode: show what would happen
/git:pr-audit 123              # Audit specific PR #123
/git:pr-audit --gitlab         # Use GitLab instead of GitHub
```

**Parsing:**
```bash
QUICK_MODE=false && [[ "$ARGUMENTS" == *"--quick"* ]] && QUICK_MODE=true
DRY_RUN=false && [[ "$ARGUMENTS" == *"--dry-run"* ]] && DRY_RUN=true
GITLAB=false && [[ "$ARGUMENTS" == *"--gitlab"* ]] && GITLAB=true
FORCE_ORG=false && [[ "$ARGUMENTS" == *"--org"* ]] && FORCE_ORG=true
FORCE_SINGLE=false && [[ "$ARGUMENTS" == *"--single"* ]] && FORCE_SINGLE=true

CLI="gh" && [[ "$GITLAB" == true ]] && CLI="glab"

# Specific PR number
TARGET=$(echo "$ARGUMENTS" | grep -oE '^[0-9]+' | head -1)

# Extract org name if provided with --org flag
EXPLICIT_ORG_NAME=$(echo "$ARGUMENTS" | grep -oE '\-\-org [a-zA-Z0-9_-]+' | awk '{print $2}')
```

---

## Phase 0: Pre-Flight

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

# Detect org/owner from remote URL
REMOTE_URL=$(git remote get-url origin 2>/dev/null)
ORG=$(echo "$REMOTE_URL" | sed -E 's#(git@|https://)([^:/]+)[:/]([^/]+)/.*#\3#')

# Check if owner is an organization or personal account
IS_ORG=false
if [[ "$CLI" == "gh" ]]; then
    OWNER_TYPE=$(gh api "users/$ORG" --jq '.type' 2>/dev/null || echo "Unknown")
    [[ "$OWNER_TYPE" == "Organization" ]] && IS_ORG=true
fi

# Determine final mode
ORG_MODE=false
if [[ "$FORCE_SINGLE" == true ]]; then
    ORG_MODE=false
    echo "Mode: Single-repo (forced via --single)"
elif [[ "$FORCE_ORG" == true ]]; then
    ORG_MODE=true
    ORG_NAME="${EXPLICIT_ORG_NAME:-$ORG}"
    echo "Mode: Organization (forced via --org)"
elif [[ "$IS_ORG" == true ]]; then
    ORG_MODE=true
    ORG_NAME="$ORG"
    echo "Mode: Organization (auto-detected: $ORG_NAME)"
else
    ORG_MODE=false
    echo "Mode: Single-repo (personal account: $ORG)"
fi
```

**Display:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PR AUDIT - PRE-FLIGHT COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Repo:      {REPO_NAME}
Owner:     {ORG} ({OWNER_TYPE})
Mode:      {org|single} {auto|forced}
Base:      origin/{BASE_BRANCH}
Platform:  {github|gitlab}
Dry-run:   {true|false}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Phase 1: PR Discovery

### Organization Mode (auto or --org)

When in org mode, discover PRs across ALL repos in the organization with CI status:

```bash
if [[ "$ORG_MODE" == true ]]; then
    echo ""
    echo "Scanning all repos in $ORG_NAME..."

    # Get list of repos in org (exclude .github meta repo)
    ORG_REPOS=$(gh repo list "$ORG_NAME" --json name --limit 100 --jq '.[].name' | grep -v '^\.github$')

    # Collect PRs from all repos with CI status
    ALL_ORG_PRS="[]"

    for repo in $ORG_REPOS; do
        REPO_PRS=$(gh pr list --repo "$ORG_NAME/$repo" --state open \
            --json number,title,headRefName,baseRefName,additions,deletions,mergeable,createdAt \
            2>/dev/null || echo "[]")

        if [[ "$REPO_PRS" != "[]" ]]; then
            # Get CI status for each PR
            PR_NUMS=$(echo "$REPO_PRS" | jq -r '.[].number')
            for pr_num in $PR_NUMS; do
                CI_CHECK=$(gh pr checks $pr_num --repo "$ORG_NAME/$repo" 2>/dev/null | head -1)
                if echo "$CI_CHECK" | grep -q "fail"; then
                    CI_STATUS="failing"
                elif echo "$CI_CHECK" | grep -q "pending"; then
                    CI_STATUS="pending"
                elif echo "$CI_CHECK" | grep -q "pass"; then
                    CI_STATUS="passing"
                else
                    CI_STATUS="unknown"
                fi
                REPO_PRS=$(echo "$REPO_PRS" | jq --arg num "$pr_num" --arg ci "$CI_STATUS" \
                    '[.[] | if .number == ($num | tonumber) then . + {ciStatus: $ci} else . end]')
            done

            # Add repo name to each PR
            REPO_PRS=$(echo "$REPO_PRS" | jq --arg repo "$repo" '[.[] | . + {repo: $repo}]')
            ALL_ORG_PRS=$(echo "$ALL_ORG_PRS $REPO_PRS" | jq -s 'add')
        fi
    done

    # Sort PRs: failing CI first, then by creation date
    ALL_ORG_PRS=$(echo "$ALL_ORG_PRS" | jq 'sort_by(
        if .ciStatus == "failing" then 0
        elif .ciStatus == "pending" then 1
        else 2 end,
        .createdAt
    )')

    # Display enhanced org summary
    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "ORGANIZATION: $ORG_NAME"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

    TOTAL_PRS=0
    FAILING_PRS=0

    for repo in $ORG_REPOS; do
        repo_prs=$(echo "$ALL_ORG_PRS" | jq --arg r "$repo" '[.[] | select(.repo == $r)]')
        pr_count=$(echo "$repo_prs" | jq 'length')
        failing_count=$(echo "$repo_prs" | jq '[.[] | select(.ciStatus == "failing")] | length')

        if [[ $pr_count -gt 0 ]]; then
            TOTAL_PRS=$((TOTAL_PRS + pr_count))
            FAILING_PRS=$((FAILING_PRS + failing_count))

            status="⚡ $pr_count PR(s)"
            [[ $failing_count -gt 0 ]] && status="$status [✗ $failing_count failing]"
            printf "  %-25s %s\n" "$repo" "$status"

            # List PRs for this repo
            echo "$repo_prs" | jq -r '.[] | "    #\(.number) [\(.headRefName) → \(.baseRefName)] \(.title | .[0:40])... (\(.ciStatus))"'
        else
            printf "  %-25s %s\n" "$repo" "✓ Clean"
        fi
    done

    echo ""
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "TOTAL: $TOTAL_PRS open PRs ($FAILING_PRS with failing CI)"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
fi
```

### Single Repo Mode (default for personal, or --single)

### Find Open PRs (conventional branches)

```bash
ALL_PRS=$($CLI pr list --state open \
    --json number,title,headRefName,baseRefName,additions,deletions,mergeable,createdAt \
    --jq '[.[] | select(.headRefName | test("^(feat|fix|chore|docs|refactor)/"))]')
```

### Cross-Repo Notice (Single-Repo Mode)

When in single-repo mode but org has other PRs, show a notice:

```bash
if [[ "$ORG_MODE" == false ]] && [[ "$IS_ORG" == true ]]; then
    # Quick scan for PRs in other org repos
    OTHER_REPOS=$(gh repo list "$ORG" --json name --limit 100 --jq '.[].name' | grep -v "^$REPO_NAME$" | grep -v '^\.github$')

    OTHER_PR_COUNT=0
    OTHER_PR_REPOS=""
    for repo in $OTHER_REPOS; do
        count=$(gh pr list --repo "$ORG/$repo" --state open --json number --jq 'length' 2>/dev/null || echo "0")
        if [[ $count -gt 0 ]]; then
            OTHER_PR_COUNT=$((OTHER_PR_COUNT + count))
            OTHER_PR_REPOS="$OTHER_PR_REPOS $repo($count)"
        fi
    done

    if [[ $OTHER_PR_COUNT -gt 0 ]]; then
        echo ""
        echo "┌────────────────────────────────────────────────────────────┐"
        echo "│ NOTE: $OTHER_PR_COUNT more PR(s) in other $ORG repos:      "
        echo "│$OTHER_PR_REPOS"
        echo "│ Run '/git:pr-audit --org' to audit all repos              │"
        echo "└────────────────────────────────────────────────────────────┘"
        echo ""
    fi
fi
```

### Display Summary

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OPEN PR SUMMARY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌─────┬───────────────────────────┬─────────────────────────────┬───────┬───────────┬──────────┐
│ PR  │ Branch                    │ Title                       │ +/-   │ Mergeable │ CI       │
├─────┼───────────────────────────┼─────────────────────────────┼───────┼───────────┼──────────┤
│ 123 │ feat/add-rate-limiting    │ feat: add rate limiting...  │ +450  │ ✓         │ passing  │
│ 124 │ fix/stealth-validation    │ fix: stealth addr valid...  │ +200  │ ✓         │ passing  │
│ 125 │ chore/upgrade-deps        │ chore: upgrade deps         │ +80   │ ✗         │ failing  │
└─────┴───────────────────────────┴─────────────────────────────┴───────┴───────────┴──────────┘

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Phase 2: Merge Order Calculation

**Priority:**
1. Failing CI first (needs attention)
2. By creation date (oldest first)

**Conflict Detection:**
```bash
check_pr_conflicts() {
    local pr_num=$1
    local head_branch=$2

    # Get files changed in PR
    local pr_files=$($CLI pr diff $pr_num --name-only)

    # Check if any files overlap with already-merged PRs
    # This is a heuristic - actual conflicts detected at merge time
}
```

**Cross-Repo Dependency Detection:**
```bash
check_cross_repo_dependencies() {
    local pr_num=$1

    # Get files changed in this PR
    local changed_files=$($CLI pr diff $pr_num --name-only)

    # Check for shared package changes
    local shared_packages=""
    if echo "$changed_files" | grep -q "packages/types/"; then
        shared_packages="$shared_packages @your-org/types"
    fi
    if echo "$changed_files" | grep -q "packages/sdk/"; then
        shared_packages="$shared_packages @your-org/sdk"
    fi

    if [[ -n "$shared_packages" ]]; then
        echo "Warning: This PR modifies shared packages:$shared_packages"
        echo "   Dependent repos may need updates after merge."
    fi
}
```

**Display suggested order:**
```
SUGGESTED MERGE ORDER:
  1. PR #125 (chore/upgrade-deps) - CI FAILING - fix first
  2. PR #123 (feat/add-rate-limiting) - clean
  3. PR #124 (fix/stealth-validation) - may conflict after #123
```

---

## Phase 3: Audit Loop

**For each PR in order:**

### 3a. Show PR Details

```bash
PR_DATA=$($CLI pr view $PR_NUM --json title,body,headRefName,baseRefName,files,additions,deletions,reviews,mergeable,mergeStateStatus)
```

**Display:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PR #$PR_NUM: $TITLE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Repo:      $REPO_NAME (or $ORG_NAME/$REPO_NAME in org mode)
Branch:    $HEAD_BRANCH → $BASE_BRANCH
Files:     $FILE_COUNT changed
Changes:   +$ADDITIONS / -$DELETIONS
Mergeable: $MERGEABLE
CI Status: $CI_STATUS

Files Changed:
  $FILE_LIST

Issues Addressed:
  $ISSUES_FROM_COMMITS

Cross-Repo Dependencies:
  $DEPENDENCY_NOTICE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 3b. Show Diff Summary

```bash
$CLI pr diff $PR_NUM --stat
```

### 3c. Check CI Status

```bash
$CLI pr checks $PR_NUM
```

### 3d. Action Menu

**Present options:**
```
OPTIONS:
  [M] Merge      - Merge this PR (--merge strategy)
  [R] Review     - Deep dive into code changes
  [F] Fix        - Checkout branch, fix issues, push, then merge
  [B] Rebase     - Rebase on latest base (for conflicts)
  [I] Issue      - Create follow-up issue
  [C] Changes    - Request changes (leave review)
  [X] Close      - Close PR without merging
  [S] Skip       - Move to next PR
  [J] Jump       - Jump to different repo (org mode)
  [Q] Quit       - End audit session
```

---

## Phase 4: Actions

### [M] Merge

```bash
if [[ "$DRY_RUN" == true ]]; then
    echo "[DRY-RUN] Would merge PR #$PR_NUM to $BASE_BRANCH"
else
    # Verify base branch is correct
    CURRENT_BASE=$($CLI pr view $PR_NUM --json baseRefName -q '.baseRefName')
    if [[ "$CURRENT_BASE" != "$BASE_BRANCH" ]]; then
        echo "Changing base from $CURRENT_BASE to $BASE_BRANCH"
        $CLI pr edit $PR_NUM --base $BASE_BRANCH
    fi

    # Merge with merge commit (preserve history)
    $CLI pr merge $PR_NUM --merge

    echo "Merged PR #$PR_NUM"

    # ═══════════════════════════════════════════════════════════════════
    # POST-MERGE: Close linked issues if merging to non-default branch
    # ═══════════════════════════════════════════════════════════════════
    # GitHub only auto-closes issues when merging to default branch (main/master)

    DEFAULT_BRANCH=$($CLI repo view --json defaultBranchRef -q '.defaultBranchRef.name')

    if [[ "$BASE_BRANCH" != "$DEFAULT_BRANCH" ]]; then
        echo ""
        echo "━━━ POST-MERGE: Closing linked issues (non-default branch) ━━━"

        LINKED_ISSUES=$($CLI pr view $PR_NUM --json body,commits \
            --jq '[.body // "", (.commits[]? | .messageBody // "", .messageHeadline // "")] | join("\n")' \
            | grep -oiE "(close[sd]?|fix(es|ed)?|resolve[sd]?)\s*#[0-9]+" \
            | grep -oE "#[0-9]+" | tr -d '#' | sort -u)

        if [[ -n "$LINKED_ISSUES" ]]; then
            for issue in $LINKED_ISSUES; do
                ISSUE_STATE=$($CLI issue view $issue --json state -q '.state' 2>/dev/null || echo "NOT_FOUND")

                if [[ "$ISSUE_STATE" == "OPEN" ]]; then
                    $CLI issue close $issue --comment "Closed via PR #$PR_NUM merge to \`$BASE_BRANCH\`"
                    echo "  ✓ Closed #$issue"
                elif [[ "$ISSUE_STATE" == "CLOSED" ]]; then
                    echo "  ○ #$issue already closed"
                else
                    echo "  ✗ #$issue not found or inaccessible"
                fi
            done
        else
            echo "  No linked issues found in PR body/commits"
        fi
    fi

    # POST-MERGE: Delete remote branch (conventional branches are disposable)
    HEAD_BRANCH=$($CLI pr view $PR_NUM --json headRefName -q '.headRefName')
    if [[ "$HEAD_BRANCH" =~ ^(feat|fix|chore|docs|refactor)/ ]]; then
        git push origin --delete "$HEAD_BRANCH" 2>/dev/null && echo "  ✓ Deleted remote branch $HEAD_BRANCH" || true
        git branch -d "$HEAD_BRANCH" 2>/dev/null || true
    fi

    # Trigger rebase for remaining PRs
    NEED_REBASE=true
fi
```

### [R] Review (Deep Dive)

```bash
# Show full diff
$CLI pr diff $PR_NUM

# Show commits
$CLI pr view $PR_NUM --json commits --jq '.commits[] | "\(.oid[:7]) \(.messageHeadline)"'

# Check for common issues
# - Console.log statements
# - TODO/FIXME comments
# - Security concerns
```

### [F] Fix & Merge

```bash
PR_BRANCH=$($CLI pr view $PR_NUM --json headRefName -q '.headRefName')

# Determine if this is a different repo (org mode)
if [[ -n "$CURRENT_PR_REPO" ]] && [[ "$CURRENT_PR_REPO" != "$REPO_NAME" ]]; then
    # Different repo - clone to temp location
    TEMP_DIR="$HOME/local-dev/${CURRENT_PR_REPO}-pr-${PR_NUM}-fix"

    if [[ "$DRY_RUN" == true ]]; then
        echo "[DRY-RUN] Would clone $ORG_NAME/$CURRENT_PR_REPO for fixes"
    else
        if [[ ! -d "$TEMP_DIR" ]]; then
            git clone "git@github.com:$ORG_NAME/$CURRENT_PR_REPO.git" "$TEMP_DIR"
        fi

        cd "$TEMP_DIR"
        git fetch origin "$PR_BRANCH"
        git checkout "$PR_BRANCH"

        # Install dependencies
        if [[ -f "pnpm-lock.yaml" ]]; then pnpm install
        elif [[ -f "package.json" ]]; then npm install
        fi

        echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
        echo "MULTI-REPO FIX MODE"
        echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
        echo "Repo:   $ORG_NAME/$CURRENT_PR_REPO"
        echo "Path:   $TEMP_DIR"
        echo "Branch: $PR_BRANCH"
        echo ""
        echo "Make fixes, then I'll push and merge."
        echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

        # ... Make fixes ...

        git add .
        git commit -m "fix: address audit feedback"
        git push origin "$PR_BRANCH"

        cd "$REPO_ROOT"

        gh pr merge $PR_NUM --repo "$ORG_NAME/$CURRENT_PR_REPO" --merge

        # POST-MERGE: Delete remote + local branch (conventional branches are disposable)
        HEAD_BRANCH=$(gh pr view $PR_NUM --repo "$ORG_NAME/$CURRENT_PR_REPO" --json headRefName -q '.headRefName')
        if [[ "$HEAD_BRANCH" =~ ^(feat|fix|chore|docs|refactor)/ ]]; then
            git push origin --delete "$HEAD_BRANCH" 2>/dev/null && echo "  ✓ Deleted remote branch $HEAD_BRANCH" || true
            git branch -d "$HEAD_BRANCH" 2>/dev/null || true
        fi

        # Cleanup temp clone
        rm -rf "$TEMP_DIR"
    fi
else
    # Same repo - checkout branch directly
    if [[ "$DRY_RUN" == true ]]; then
        echo "[DRY-RUN] Would checkout $PR_BRANCH for fixes"
    else
        # Stash any current changes
        git stash push -m "pr-audit-stash" 2>/dev/null || true

        git fetch origin "$PR_BRANCH"
        git checkout "$PR_BRANCH"

        # Install dependencies
        if [[ -f "pnpm-lock.yaml" ]]; then pnpm install; fi

        echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
        echo "FIX MODE"
        echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
        echo "Branch: $PR_BRANCH"
        echo ""
        echo "Make fixes, then I'll push and merge."
        echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

        # ... Make fixes ...

        git add .
        git commit -m "fix: address audit feedback"
        git push origin "$PR_BRANCH"

        # Return to base branch
        git checkout "$BASE_BRANCH"
        git stash pop 2>/dev/null || true

        $CLI pr merge $PR_NUM --merge

        # POST-MERGE: Delete remote + local branch (conventional branches are disposable)
        HEAD_BRANCH=$($CLI pr view $PR_NUM --json headRefName -q '.headRefName')
        if [[ "$HEAD_BRANCH" =~ ^(feat|fix|chore|docs|refactor)/ ]]; then
            git push origin --delete "$HEAD_BRANCH" 2>/dev/null && echo "  ✓ Deleted remote branch $HEAD_BRANCH" || true
            git branch -d "$HEAD_BRANCH" 2>/dev/null || true
        fi
    fi
fi
```

### [B] Rebase (for conflicts)

```bash
PR_BRANCH=$($CLI pr view $PR_NUM --json headRefName -q '.headRefName')

if [[ "$DRY_RUN" == true ]]; then
    echo "[DRY-RUN] Would rebase $PR_BRANCH on $BASE_BRANCH"
else
    git fetch origin "$BASE_BRANCH" "$PR_BRANCH"

    # Stash, checkout, rebase
    git stash push -m "pr-audit-rebase-stash" 2>/dev/null || true
    git checkout "$PR_BRANCH"

    if git rebase "origin/$BASE_BRANCH"; then
        echo "Rebase successful"
        git push origin "$PR_BRANCH" --force-with-lease
    else
        echo "━━━ REBASE CONFLICTS ━━━"
        git diff --name-only --diff-filter=U
        # Option: Let AI help resolve, or ask user
    fi

    git checkout "$BASE_BRANCH"
    git stash pop 2>/dev/null || true
fi
```

### [I] Create Follow-up Issue

```bash
PR_TITLE=$($CLI pr view $PR_NUM --json title -q '.title')

if [[ "$DRY_RUN" == true ]]; then
    echo "[DRY-RUN] Would create follow-up issue for PR #$PR_NUM"
else
    $CLI issue create \
        --title "Follow-up: $PR_TITLE" \
        --body "$(cat <<EOF
Related to PR #$PR_NUM

## Issues Found During Audit

- [Details]

## Action Items

- [ ] [Action 1]
- [ ] [Action 2]

---
Created via \`/git:pr-audit\`
EOF
)" \
        --label "follow-up"
fi
```

### [C] Request Changes

```bash
if [[ "$DRY_RUN" == true ]]; then
    echo "[DRY-RUN] Would request changes on PR #$PR_NUM"
else
    $CLI pr review $PR_NUM --request-changes --body "$(cat <<EOF
## Changes Requested

[Specific feedback]

---
Review via \`/git:pr-audit\`
EOF
)"
fi
```

### [X] Close PR

```bash
if [[ "$DRY_RUN" == true ]]; then
    echo "[DRY-RUN] Would close PR #$PR_NUM"
else
    $CLI pr close $PR_NUM --comment "Closing: [reason]"
fi
```

### [J] Jump to Different Repo (Org Mode Only)

```bash
if [[ "$ORG_MODE" == true ]]; then
    echo "Available repos with open PRs:"
    echo "$ALL_ORG_PRS" | jq -r '[.[].repo] | unique | .[]' | nl

    # Ask the user to select a repo inline
fi
```

---

## Phase 5: Post-Merge Rebase

**After each merge, offer rebase for remaining PRs:**

```bash
if [[ "$NEED_REBASE" == true ]]; then
    echo ""
    echo "━━━ POST-MERGE: Checking remaining PRs ━━━"

    for remaining_pr in "${REMAINING_PRS[@]}"; do
        PR_NUM=$remaining_pr
        HEAD_BRANCH=$($CLI pr view $PR_NUM --json headRefName -q '.headRefName')

        MERGEABLE=$($CLI pr view $PR_NUM --json mergeable -q '.mergeable')

        if [[ "$MERGEABLE" == "CONFLICTING" ]]; then
            echo "PR #$PR_NUM ($HEAD_BRANCH) now has conflicts."
            echo "Rebase recommended before review."
        fi
    done
fi
```

---

## Phase 6: Quick Mode (--quick)

**Auto-merge all clean PRs without review:**

```bash
if [[ "$QUICK_MODE" == true ]]; then
    echo "QUICK MODE: Merging all passing PRs..."

    if [[ "$ORG_MODE" == true ]]; then
        for repo in $ORG_REPOS; do
            repo_prs=$(echo "$ALL_ORG_PRS" | jq --arg r "$repo" '[.[] | select(.repo == $r)]')
            for pr in $(echo "$repo_prs" | jq -r '.[].number'); do
                MERGEABLE=$(gh pr view $pr --repo "$ORG_NAME/$repo" --json mergeable -q '.mergeable')
                CI_STATUS=$(echo "$repo_prs" | jq -r --arg num "$pr" '.[] | select(.number == ($num | tonumber)) | .ciStatus')

                if [[ "$MERGEABLE" == "MERGEABLE" ]] && [[ "$CI_STATUS" == "passing" ]]; then
                    if [[ "$DRY_RUN" == true ]]; then
                        echo "[DRY-RUN] Would merge PR #$pr in $repo"
                    else
                        gh pr merge $pr --repo "$ORG_NAME/$repo" --merge
                        echo "Merged PR #$pr in $repo"

                        # POST-MERGE: Delete remote + local branch (conventional branches are disposable)
                        HEAD_BRANCH=$(gh pr view $pr --repo "$ORG_NAME/$repo" --json headRefName -q '.headRefName')
                        if [[ "$HEAD_BRANCH" =~ ^(feat|fix|chore|docs|refactor)/ ]]; then
                            git push origin --delete "$HEAD_BRANCH" 2>/dev/null && echo "  ✓ Deleted remote branch $HEAD_BRANCH" || true
                            git branch -d "$HEAD_BRANCH" 2>/dev/null || true
                        fi
                    fi
                else
                    echo "Skipped PR #$pr in $repo (not mergeable or CI not passing)"
                fi
            done
        done
    else
        for pr in $(echo "$ALL_PRS" | jq -r '.[].number'); do
            MERGEABLE=$($CLI pr view $pr --json mergeable -q '.mergeable')

            if [[ "$MERGEABLE" == "MERGEABLE" ]]; then
                if [[ "$DRY_RUN" == true ]]; then
                    echo "[DRY-RUN] Would merge PR #$pr"
                else
                    $CLI pr merge $pr --merge
                    echo "Merged PR #$pr"

                    # POST-MERGE: Delete remote + local branch (conventional branches are disposable)
                    HEAD_BRANCH=$($CLI pr view $pr --json headRefName -q '.headRefName')
                    if [[ "$HEAD_BRANCH" =~ ^(feat|fix|chore|docs|refactor)/ ]]; then
                        git push origin --delete "$HEAD_BRANCH" 2>/dev/null && echo "  ✓ Deleted remote branch $HEAD_BRANCH" || true
                        git branch -d "$HEAD_BRANCH" 2>/dev/null || true
                    fi
                fi
            else
                echo "Skipped PR #$pr (not mergeable or CI failing)"
            fi
        done
    fi
fi
```

---

## Phase 7: Session Summary

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PR AUDIT SESSION SUMMARY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Merged: 3 PRs
   - #123 (feat/add-rate-limiting) → Issues #101, #104, #107
   - #124 (fix/stealth-validation) → Issue #102
   - #157 (chore/upgrade-deps) → Issue #112

Issues Auto-Closed: 5
   - #101, #104, #107 (via PR #123)
   - #102 (via PR #124)
   - #112 (via PR #157)

Branches Cleaned: 3
   - feat/add-rate-limiting ✓
   - fix/stealth-validation ✓
   - chore/upgrade-deps ✓

Issues Created: 1
   - #200: Follow-up for #125

Changes Requested: 0

Skipped: 0 PRs

Closed: 0 PRs

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

NEXT STEPS:
  - Check dependent repos for shared package updates

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Usage Examples

```bash
# Auto-detect mode (org mode if in org repo)
/git:pr-audit

# Audit all PRs across ALL repos in organization
/git:pr-audit --org

# Audit specific organization
/git:pr-audit --org your-org

# Force single-repo mode (ignore org detection)
/git:pr-audit --single

# Quick merge all passing PRs (org-aware)
/git:pr-audit --quick

# Quick merge all PRs across org
/git:pr-audit --org --quick

# Preview mode
/git:pr-audit --dry-run

# Specific PR
/git:pr-audit 123

# GitLab
/git:pr-audit --gitlab
```

---

## Error Handling

**No PRs found:**
- Display "No open PRs found" and exit gracefully

**Permission denied:**
- Check merge permissions
- Suggest requesting access

**Merge conflicts:**
- Offer rebase option
- Checkout branch for resolution
- Never auto-merge with conflicts

**CI failing:**
- Show test output
- Options: Fix, Skip, Create Issue

**Multi-repo clone failure:**
- Check SSH keys / HTTPS auth
- Verify repo access permissions
- Try alternative clone method

---

## Safety Rules

1. **NEVER merge to main/master** without explicit confirmation
2. **NEVER force push** without explicit confirmation
3. **Always confirm** before merge/close
4. **Dry-run first** for unfamiliar repos
5. **Warn about cross-repo dependencies** before merging shared packages

---

*"Review with Ihsan (excellence), merge with confidence."*

Alhamdulillah! Starting PR audit now...
