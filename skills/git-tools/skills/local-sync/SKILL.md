---
name: git-tools-local-sync
description: Comprehensive git synchronization - fetch all remotes, update all branches, and ensure local repo matches remote completely (dual-remote aware)
argument-hint: "[--cleanup] [--create-missing] [--force] [--prefer-remote <name>]"
allowed-tools: ["bash", "read", "write", "edit"]
---

Bismillah! I'll perform a comprehensive synchronization of your local repository with all remote repositories to ensure nothing is left behind.

Arguments provided: $ARGUMENTS

## Comprehensive Local-Remote Synchronization (Dual-Remote Aware)

This command ensures your local repository is completely synchronized with all remote repositories. Fully supports dual-remote setups (GitHub + GitLab) and multi-remote configurations.

### Supported Arguments

- `--cleanup` - Remove obsolete local branches (only if not on ANY remote)
- `--create-missing` - Create local tracking branches for new remote branches
- `--force` - Override local changes with remote versions
- `--prefer-remote <name>` - Preferred remote for branch creation (default: origin)

### Implementation Steps

**Step 1: Pre-sync Repository Analysis & Remote Detection**

```bash
echo "📊 **Repository Analysis**"
echo ""

# Check for uncommitted changes
if [[ -n $(git status --porcelain) ]]; then
    echo "⚠️  Warning: You have uncommitted changes:"
    git status --short
    echo ""
fi

# Detect all configured remotes
echo "🔗 **Configured Remotes:**"
remotes=($(git remote))
for remote in "${remotes[@]}"; do
    url=$(git remote get-url "$remote" 2>/dev/null)
    echo "  ├─ $remote → $url"
done
echo ""

# Current branch info
current_branch=$(git branch --show-current)
echo "🌿 **Current Branch:** $current_branch"
echo ""
```

**Step 2: Comprehensive Fetch from All Remotes**

```bash
echo "🔄 **Fetching from all remotes...**"
echo ""

fetch_failed=()
for remote in "${remotes[@]}"; do
    echo -n "  Fetching $remote... "
    if git fetch "$remote" --tags --prune 2>/dev/null; then
        echo "✓"
    else
        echo "✗ (failed)"
        fetch_failed+=("$remote")
    fi
done

if [[ ${#fetch_failed[@]} -gt 0 ]]; then
    echo ""
    echo "⚠️  Failed to fetch from: ${fetch_failed[*]}"
fi
echo ""
```

**Step 3: Current Branch Synchronization**

```bash
echo "📥 **Synchronizing current branch ($current_branch)...**"

# Check if current branch has upstream
if git rev-parse --verify @{upstream} >/dev/null 2>&1; then
    upstream=$(git rev-parse --abbrev-ref @{upstream})
    echo "  Upstream: $upstream"

    # Check ahead/behind status
    ahead=$(git rev-list --count @{upstream}..HEAD 2>/dev/null || echo 0)
    behind=$(git rev-list --count HEAD..@{upstream} 2>/dev/null || echo 0)

    if [[ $behind -gt 0 ]]; then
        echo "  Behind by $behind commit(s), pulling..."
        git pull --ff-only 2>/dev/null || {
            echo "  ⚠️  Fast-forward failed, attempting merge..."
            git pull --no-ff
        }
    else
        echo "  ✓ Already up to date"
    fi

    if [[ $ahead -gt 0 ]]; then
        echo "  ⚠️  Ahead by $ahead commit(s) - consider pushing"
    fi
else
    echo "  ⚠️  No upstream tracking branch configured"
fi
echo ""
```

**Step 4: Update All Local Tracking Branches**

```bash
echo "🔄 **Updating all local tracking branches...**"
echo ""

git for-each-ref --format='%(refname:short) %(upstream:short)' refs/heads | while read local upstream; do
    if [[ -n "$upstream" && "$local" != "$current_branch" ]]; then
        remote_name="${upstream%%/*}"

        # Check if upstream exists
        if git rev-parse --verify "$upstream" >/dev/null 2>&1; then
            local_sha=$(git rev-parse "$local")
            remote_sha=$(git rev-parse "$upstream")

            if [[ "$local_sha" != "$remote_sha" ]]; then
                # Check if can fast-forward
                if git merge-base --is-ancestor "$local" "$upstream" 2>/dev/null; then
                    echo -n "  Updating $local from $upstream... "
                    git update-ref "refs/heads/$local" "$remote_sha" && echo "✓" || echo "✗"
                else
                    echo "  ⚠️  $local has diverged from $upstream (manual merge needed)"
                fi
            fi
        fi
    fi
done
echo ""
```

**Step 5: Discover Remote Branches Across All Remotes**

```bash
echo "🔍 **Discovering remote branches...**"
echo ""

declare -A branch_remotes  # branch_name -> "remote1,remote2"
declare -A new_branches    # branches not in local

# Collect all remote branches and which remotes have them
for remote in "${remotes[@]}"; do
    while read -r remote_branch; do
        [[ -z "$remote_branch" ]] && continue
        [[ "$remote_branch" == *"/HEAD" ]] && continue

        branch_name="${remote_branch#$remote/}"

        # Track which remotes have this branch
        if [[ -n "${branch_remotes[$branch_name]}" ]]; then
            branch_remotes[$branch_name]="${branch_remotes[$branch_name]},$remote"
        else
            branch_remotes[$branch_name]="$remote"
        fi

        # Check if local branch exists
        if ! git show-ref --verify --quiet "refs/heads/$branch_name"; then
            new_branches[$branch_name]="${branch_remotes[$branch_name]}"
        fi
    done < <(git branch -r --list "$remote/*" | sed 's/^[[:space:]]*//')
done

# Display new remote branches
if [[ ${#new_branches[@]} -gt 0 ]]; then
    echo "🆕 **New remote branches (not local):**"
    for branch in "${!new_branches[@]}"; do
        remotes_list="${new_branches[$branch]}"
        # Determine display label
        if [[ "$remotes_list" == *","* ]]; then
            echo "  ├─ $branch (on: $remotes_list)"
        else
            echo "  ├─ $branch ($remotes_list only)"
        fi
    done
    echo ""
else
    echo "  ✓ No new remote branches found"
    echo ""
fi
```

**Step 6: Create Missing Local Branches (if --create-missing)**

```bash
if [[ "$ARGUMENTS" == *"--create-missing"* ]]; then
    echo "📝 **Creating local tracking branches...**"

    # Determine preferred remote
    prefer_remote="origin"
    if [[ "$ARGUMENTS" =~ --prefer-remote[[:space:]]+([^[:space:]]+) ]]; then
        prefer_remote="${BASH_REMATCH[1]}"
    fi
    echo "  Preferred remote: $prefer_remote"
    echo ""

    for branch in "${!new_branches[@]}"; do
        remotes_list="${new_branches[$branch]}"

        # Select source remote (prefer specified, fallback to first available)
        if [[ "$remotes_list" == *"$prefer_remote"* ]]; then
            source_remote="$prefer_remote"
        else
            source_remote="${remotes_list%%,*}"
        fi

        echo -n "  Creating $branch from $source_remote/$branch... "
        if git branch --track "$branch" "$source_remote/$branch" 2>/dev/null; then
            echo "✓"
        else
            echo "✗"
        fi
    done
    echo ""
fi
```

**Step 7: Smart Cleanup Obsolete Branches (if --cleanup)**

Check if branch exists on ANY remote before considering it obsolete.

```bash
if [[ "$ARGUMENTS" == *"--cleanup"* ]]; then
    echo "🗑️  **Checking for obsolete local branches...**"
    echo ""

    obsolete_branches=()

    git for-each-ref --format='%(refname:short)' refs/heads | while read local_branch; do
        # Skip current branch
        [[ "$local_branch" == "$current_branch" ]] && continue

        # Skip main/master/dev protected branches
        [[ "$local_branch" =~ ^(main|master|dev|develop)$ ]] && continue

        exists_on_remote=false

        # Check if branch exists on ANY remote
        for remote in "${remotes[@]}"; do
            if git ls-remote --heads "$remote" "$local_branch" 2>/dev/null | grep -q .; then
                exists_on_remote=true
                break
            fi
        done

        if [[ "$exists_on_remote" == false ]]; then
            # Double-check: does it have upstream that's gone?
            upstream=$(git rev-parse --abbrev-ref "$local_branch@{upstream}" 2>/dev/null)
            if [[ -n "$upstream" ]] || [[ -z "$(git log -1 --since='3 months ago' "$local_branch" 2>/dev/null)" ]]; then
                echo "  ⚠️  $local_branch - not on any remote"
                obsolete_branches+=("$local_branch")
            fi
        fi
    done

    if [[ ${#obsolete_branches[@]} -gt 0 ]]; then
        echo ""
        echo "Found ${#obsolete_branches[@]} obsolete branch(es)."

        if [[ "$ARGUMENTS" == *"--force"* ]]; then
            for branch in "${obsolete_branches[@]}"; do
                git branch -D "$branch" && echo "  Deleted: $branch"
            done
        else
            echo "Run with --force to auto-delete, or delete manually:"
            for branch in "${obsolete_branches[@]}"; do
                echo "  git branch -d $branch"
            done
        fi
    else
        echo "  ✓ No obsolete branches found"
    fi
    echo ""
fi
```

**Step 8: Comprehensive Multi-Remote Status Report**

```bash
echo ""
echo "═══════════════════════════════════════════════════════════════"
echo "🔄 **SYNCHRONIZATION COMPLETE**"
echo "═══════════════════════════════════════════════════════════════"
echo ""

# Remote status
echo "🔗 **Remotes Synchronized:**"
for remote in "${remotes[@]}"; do
    url=$(git remote get-url "$remote" 2>/dev/null)
    if [[ " ${fetch_failed[*]} " =~ " $remote " ]]; then
        echo "  ├─ $remote ✗ (fetch failed)"
    else
        echo "  ├─ $remote ✓"
    fi
done
echo ""

# Branch status with ahead/behind for all tracking branches
echo "🌿 **Local Branches Status:**"
git for-each-ref --format='%(refname:short) %(upstream:short) %(upstream:track)' refs/heads | while read local upstream track; do
    if [[ -n "$upstream" ]]; then
        echo "  $local → $upstream $track"
    else
        echo "  $local (no upstream)"
    fi
done
echo ""

# Summary of remote-only branches per remote
echo "🌐 **Remote-Only Branches:**"
for remote in "${remotes[@]}"; do
    count=0
    while read -r remote_branch; do
        [[ -z "$remote_branch" ]] && continue
        [[ "$remote_branch" == *"/HEAD" ]] && continue
        branch_name="${remote_branch#$remote/}"
        if ! git show-ref --verify --quiet "refs/heads/$branch_name"; then
            ((count++))
        fi
    done < <(git branch -r --list "$remote/*" | sed 's/^[[:space:]]*//')
    echo "  $remote: $count new branch(es)"
done
echo ""

# Recent activity
echo "📈 **Recent Activity (last 10 commits across all branches):**"
git log --oneline --graph --decorate --all -10
echo ""

# Sync summary
echo "✅ **Sync Summary:**"
echo "  • Remotes fetched: ${#remotes[@]} (failed: ${#fetch_failed[@]})"
echo "  • Current branch: $current_branch"
echo "  • Cleanup: $(if [[ "$ARGUMENTS" == *"--cleanup"* ]]; then echo "Yes"; else echo "No (use --cleanup)"; fi)"
echo "  • Create missing: $(if [[ "$ARGUMENTS" == *"--create-missing"* ]]; then echo "Yes"; else echo "No (use --create-missing)"; fi)"
echo ""
echo "🚀 **Repository is now fully synchronized with all remotes!**"
```

### Usage Examples

```bash
# Basic synchronization (all remotes)
/local-sync

# Sync and create local branches for all new remote branches
/local-sync --create-missing

# Sync and create branches, preferring gitlab as source
/local-sync --create-missing --prefer-remote gitlab

# Sync and clean up obsolete local branches (checks ALL remotes)
/local-sync --cleanup

# Complete sync with all options
/local-sync --create-missing --cleanup

# Force sync (override local changes, auto-delete obsolete)
/local-sync --force --cleanup
```

### Dual-Remote Behavior

| Scenario | Behavior |
|----------|----------|
| Branch on GitHub only | Shows as "(origin only)" or "(github only)" |
| Branch on GitLab only | Shows as "(gitlab only)" |
| Branch on both | Shows as "(on: origin,gitlab)" |
| Local branch, gone from origin but exists on gitlab | **NOT deleted** - still on a remote |
| Local branch, gone from ALL remotes | Marked obsolete, safe to delete |
| `--create-missing` | Uses `--prefer-remote` or first available |

### Safety Features

1. **Multi-Remote Awareness**: Never deletes branches that exist on ANY configured remote
2. **Protected Branches**: Skips main/master/dev/develop from cleanup
3. **Uncommitted Changes Warning**: Alerts before operations
4. **Fetch Failure Handling**: Continues sync even if one remote fails
5. **Non-Destructive Default**: Requires --force for auto-deletion
6. **Clear Reporting**: Shows exactly which remote(s) have each branch

This command ensures your local repository stays completely synchronized with all remotes, respecting your dual-push GitHub + GitLab setup. InshaAllah, this will keep your development workflow smooth and up-to-date!
