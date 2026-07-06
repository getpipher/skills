---
name: git-tools-create-pr
description: Create a pull request (GitHub) or merge request (GitLab) from FROM branch to TO branch
argument-hint: <FROM_BRANCH> <TO_BRANCH> [--github|--gitlab]
allowed-tools: ["bash", "read", "write", "edit"]
---

Bismillah! I'll create a pull request/merge request with the provided arguments: $ARGUMENTS

I need to parse the arguments as: FROM_BRANCH TO_BRANCH [--github|--gitlab]

## Platform Selection

**Flags:**
- `--github` - Create PR on GitHub using `gh` CLI (default)
- `--gitlab` - Create MR on GitLab using `glab` CLI

**Default behavior:** GitHub (if no flag specified)

### Platform Detection Logic

```bash
# Parse platform flag from arguments
if [[ "$ARGUMENTS" == *"--gitlab"* ]]; then
    PLATFORM="gitlab"
    CLI_TOOL="glab"
    PR_TYPE="merge request"
elif [[ "$ARGUMENTS" == *"--github"* ]]; then
    PLATFORM="github"
    CLI_TOOL="gh"
    PR_TYPE="pull request"
else
    # Default to GitHub
    PLATFORM="github"
    CLI_TOOL="gh"
    PR_TYPE="pull request"
fi

echo "📌 Platform: $PLATFORM ($PR_TYPE)"
```

## Using with Thinking Mode

When executing this command with **mode: think**, I will:

- Analyze the commit history between FROM_BRANCH and TO_BRANCH for context
- Review all changed files to understand the scope and impact of modifications
- Craft a comprehensive PR title that accurately reflects the changes
- Generate detailed PR description with:
  - Clear summary of what changed and why
  - Impact analysis and potential risks
  - Testing considerations and verification steps
  - Related issues or dependencies
- Consider if the PR scope is appropriate or should be split
- Validate that the target branch choice is optimal

This thoughtful approach ensures high-quality PRs with better documentation and reviewer context.

Let me:

1. **Parse the arguments and validate input**
   - Extract FROM_BRANCH and TO_BRANCH from the arguments
   - Validate that both arguments are provided

2. **Check repository status and prerequisites**
   - Verify we're in a git repository
   - Check if the appropriate CLI is installed and authenticated:
     - GitHub: `gh auth status`
     - GitLab: `glab auth status`
   - Confirm both branches exist (locally or remotely)

3. **Prepare branches for PR creation**
   - Fetch latest changes from remote
   - Switch to FROM_BRANCH and ensure it's up to date
   - Push FROM_BRANCH to remote if needed

4. **Create the pull request / merge request**

   **For GitHub (default):**
   ```bash
   gh pr create --base TO_BRANCH --head FROM_BRANCH --title "..." --body "..."
   ```

   **For GitLab:**
   ```bash
   glab mr create --source-branch FROM_BRANCH --target-branch TO_BRANCH --title "..." --description "..."
   ```

   - Generate meaningful PR/MR title based on recent commits
   - Create comprehensive description with changes summary
   - **IMPORTANT**: Mention '@claude' in the PR/MR description and request review with specific focus areas:
     - Critical code changes and their impact
     - Architecture decisions and patterns
     - Security considerations
     - Performance implications
     - Testing coverage and edge cases
   - Provide context about the changes and areas of concern
   - **NEVER include AI attribution, credits, or "Generated with assistance" footers**
   - Return the PR/MR URL

## Usage Examples

```bash
# Create PR on GitHub (default)
/git:create-pr dev main

# Create PR on GitHub (explicit)
/git:create-pr dev main --github

# Create MR on GitLab
/git:create-pr dev main --gitlab
```

Let me execute these steps now.

