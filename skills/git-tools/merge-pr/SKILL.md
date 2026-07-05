---
name: git-tools-merge-pr
description: Merge a pull request (GitHub) or merge request (GitLab) by number or URL with configurable merge strategy
argument-hint: <PR_NUMBER_OR_URL> [merge|squash|rebase] [--delete-branch] [--github|--gitlab]
---

Bismillah! I'll merge the specified pull request/merge request with the provided arguments: $ARGUMENTS

## Platform Selection

**Flags:**
- `--github` - Merge PR on GitHub using `gh` CLI
- `--gitlab` - Merge MR on GitLab using `glab` CLI

**Auto-Detection (default):**
- URLs containing `github.com` → GitHub
- URLs containing `gitlab.com` → GitLab
- Plain numbers → GitHub (default)

### Platform Detection Logic

```bash
# Parse platform flag or auto-detect from URL
if [[ "$ARGUMENTS" == *"--gitlab"* ]]; then
    PLATFORM="gitlab"
    CLI_TOOL="glab"
    PR_TYPE="merge request"
elif [[ "$ARGUMENTS" == *"--github"* ]]; then
    PLATFORM="github"
    CLI_TOOL="gh"
    PR_TYPE="pull request"
elif [[ "$ARGUMENTS" == *"gitlab.com"* ]]; then
    PLATFORM="gitlab"
    CLI_TOOL="glab"
    PR_TYPE="merge request"
elif [[ "$ARGUMENTS" == *"github.com"* ]]; then
    PLATFORM="github"
    CLI_TOOL="gh"
    PR_TYPE="pull request"
else
    # Default to GitHub for plain numbers
    PLATFORM="github"
    CLI_TOOL="gh"
    PR_TYPE="pull request"
fi

echo "📌 Platform: $PLATFORM ($PR_TYPE)"
```

## Using with Thinking Mode

When executing this command with **mode: think**, I will:

- Thoroughly review the PR content, including all changed files and their impact
- Analyze the commit history to understand the development approach
- Evaluate the appropriateness of the chosen merge strategy based on:
  - Repository branching strategy and conventions
  - Type and scope of changes in the PR
  - Team preferences and project guidelines
- Assess potential risks and conflicts before merging
- Consider the impact on downstream branches and deployments
- Verify that all checks, reviews, and requirements are properly satisfied
- Provide detailed reasoning for the merge strategy recommendation

This comprehensive analysis ensures optimal merge decisions with full context awareness.

Let me parse the arguments and merge the PR intelligently:

## **Argument Processing**

**Input**: `$ARGUMENTS`

I need to:

1. **Extract PR identifier** (number or URL) from the first argument
2. **Parse merge strategy** from second argument (default: merge)
3. **Check for --delete-branch flag** to determine post-merge branch handling
4. **Validate and normalize inputs**

## **Comprehensive PR Merge Process**

### 1. **Parse and Validate Input**

- Extract PR number from URL if full GitHub URL provided
- Support formats like:
  - `3` (simple number)
  - `https://github.com/owner/repo/pull/3`
  - `https://github.com/owner/repo/pull/3#issuecomment-123`
- Validate merge strategy: `merge`, `squash`, or `rebase`
- Default to `merge` if no strategy specified
- Parse `--delete-branch` flag for post-merge branch deletion
- **Default behavior: preserve branch after merge**

### 2. **Repository and Authentication Checks**

- Verify we're in a git repository
- Confirm GitHub CLI (gh) is installed and authenticated
- Check if the specified PR exists and is in mergeable state
- Validate user has merge permissions

### 3. **Pre-Merge Validation**

- Check PR status (open, approved, CI passing)
- Verify no merge conflicts exist
- Confirm all required checks have passed
- Display PR details for confirmation

### 4. **Execute Merge with Strategy**

- **Merge Commit** (`merge`): `gh pr merge <PR> --merge`
  - Preserves commit history
  - Creates merge commit
  - Maintains branch context

- **Squash Merge** (`squash`): `gh pr merge <PR> --squash`
  - Combines all commits into single commit
  - Clean linear history
  - Ideal for feature branches

- **Rebase Merge** (`rebase`): `gh pr merge <PR> --rebase`
  - Replays commits without merge commit
  - Linear history without merge bubbles
  - Preserves individual commits

### 5. **Post-Merge Actions**

- Display merge success confirmation
- Show the merged commit SHA
- **Preserve merged branch by default** (no automatic deletion)
- Only delete branch if explicitly requested by user with `--delete-branch` flag
- Update local repository if needed
- Display next steps or recommendations

### 6. **Error Handling and Recovery**

- Handle PR not found scenarios
- Manage permission errors gracefully
- Provide guidance for merge conflicts
- Suggest alternative strategies if current fails
- Rollback options if merge encounters issues

## **Advanced Features**

### **Smart URL Parsing**

Support multiple URL formats for both platforms:

**GitHub:**
- `https://github.com/owner/repo/pull/123`
- `https://github.com/owner/repo/pull/123#discussion_r456`
- `https://github.com/owner/repo/pull/123/files`

**GitLab:**
- `https://gitlab.com/owner/repo/-/merge_requests/123`
- `https://gitlab.com/group/subgroup/repo/-/merge_requests/45`

Short URLs and redirects supported for both platforms.

### **Merge Strategy Intelligence**

- Analyze PR characteristics to suggest optimal merge strategy
- Consider branch protection rules
- Respect repository merge preferences
- Warn about destructive operations

### **Interactive Confirmations**

- Display PR summary before merging
- Show impact of merge strategy choice
- **Branch preservation by default** - no deletion confirmations needed
- Only confirm branch deletion if `--delete-branch` flag is used
- Allow abort at any step

### **Integration Benefits**

- Seamless workflow within the agent session
- Consistent with existing slash commands
- Leverages GitHub CLI for reliability
- Maintains audit trail and proper attribution

## **Example Usage**

```bash
# Merge PR by number on GitHub (default)
/git:merge-pr 3

# Merge PR by number with squash on GitHub
/git:merge-pr 3 squash

# Merge PR by GitHub URL (auto-detected)
/git:merge-pr https://github.com/user/repo/pull/15 rebase

# Merge MR on GitLab by number
/git:merge-pr 45 --gitlab

# Merge MR by GitLab URL (auto-detected)
/git:merge-pr https://gitlab.com/user/repo/-/merge_requests/45

# Merge with explicit merge commit on GitHub
/git:merge-pr 7 merge --github

# Merge and delete the branch after merge
/git:merge-pr 3 squash --delete-branch

# Merge GitLab MR with rebase and delete branch
/git:merge-pr 15 rebase --delete-branch --gitlab
```

### Platform-Specific Merge Commands

**GitHub:**
```bash
gh pr merge <PR_NUMBER> --merge|--squash|--rebase [--delete-branch]
```

**GitLab:**
```bash
glab mr merge <MR_NUMBER> [--squash] [--remove-source-branch]
```

Note: GitLab's `glab mr merge` uses `--squash` and `--remove-source-branch` flags.

Alhamdulillah, let me execute this comprehensive PR merge process now!

