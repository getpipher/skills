---
name: git-tools-label-issues
description: Intelligently analyze and label unlabeled GitHub issues (works with any repo)
argument-hint: "[--dry-run] [--limit N]"
allowed-tools: ["bash", "read", "write", "edit"]
---

# Auto-Label Issues - Intelligent Issue Organization

Bismillah! I'll analyze unlabeled or incompletely labeled GitHub issues and suggest appropriate labels based on content analysis.

Arguments provided: $ARGUMENTS

## Safety & Compatibility

This command is designed to work safely with ANY repository:
- ✅ Validates GitHub CLI authentication
- ✅ Checks repository capabilities
- ✅ Respects existing labels (no overwrites)
- ✅ Works with any label naming convention
- ✅ Handles repos with 0 to 1000+ issues
- ✅ Validates permissions before making changes
- ✅ Provides preview mode for safety

## Implementation Strategy

### Phase 1: Environment Validation

```bash
# Check GitHub CLI authentication
if ! gh auth status &>/dev/null; then
    echo "❌ GitHub CLI not authenticated. Run: gh auth login"
    exit 1
fi

# Verify we're in a git repository
if ! git rev-parse --git-dir &>/dev/null 2>&1; then
    echo "❌ Not in a git repository"
    exit 1
fi

# Get repository information
REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner 2>/dev/null)
if [ -z "$REPO" ]; then
    echo "❌ Could not determine GitHub repository"
    exit 1
fi

# Check if issues are enabled
ISSUES_ENABLED=$(gh repo view --json hasIssuesEnabled -q .hasIssuesEnabled)
if [ "$ISSUES_ENABLED" != "true" ]; then
    echo "❌ Issues are not enabled for this repository"
    exit 1
fi

echo "✅ Repository: $REPO"
```

### Phase 2: Fetch Unlabeled Issues

```bash
# Parse arguments
DRY_RUN=false
LIMIT=50

if echo "$ARGUMENTS" | grep -q -- "--dry-run"; then
    DRY_RUN=true
    echo "🔍 DRY RUN MODE - No labels will be applied"
fi

if echo "$ARGUMENTS" | grep -q -- "--limit"; then
    LIMIT=$(echo "$ARGUMENTS" | grep -o -- "--limit [0-9]\+" | grep -o "[0-9]\+")
fi

# Fetch issues without labels (or with incomplete labels)
# NOTE: Explicitly NOT fetching 'body' to avoid token usage issues with large issue descriptions
echo "📊 Fetching unlabeled issues (limit: $LIMIT)..."

ISSUES=$(gh issue list \
    --repo "$REPO" \
    --state open \
    --limit "$LIMIT" \
    --json number,title,labels,url \
    --jq '.[] | select(.labels | length == 0)')

ISSUE_COUNT=$(echo "$ISSUES" | jq -s 'length')

if [ "$ISSUE_COUNT" -eq 0 ]; then
    echo "✅ No unlabeled issues found!"
    exit 0
fi

echo "Found $ISSUE_COUNT unlabeled issues"
```

### Phase 3: Fetch Available Labels

```bash
# Get all labels available in the repository
AVAILABLE_LABELS=$(gh label list --repo "$REPO" --json name --jq '.[].name')

if [ -z "$AVAILABLE_LABELS" ]; then
    echo "⚠️  No labels defined in this repository"
    echo "Create labels first using: gh label create <name>"
    exit 1
fi

echo "📋 Available labels: $(echo "$AVAILABLE_LABELS" | wc -l | tr -d ' ')"
```

### Phase 4: Intelligent Analysis

For each unlabeled issue, I will:

1. **Read the full context** (title + body)
2. **Analyze content semantically** (not just keyword matching)
3. **Match to available labels** (only suggest labels that exist)
4. **Consider label categories**:
   - Area/Component (area:*, component:*)
   - Type (type:*, kind:*)
   - Priority (priority:*)
   - Status (status:*)
   - Any custom labels

5. **Provide reasoning** for each suggestion

### Phase 5: Apply or Preview

```bash
if [ "$DRY_RUN" = true ]; then
    # Preview mode - show what would be done
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "DRY RUN - No changes will be made"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
else
    # Ask for confirmation before applying
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "Ready to apply labels to $ISSUE_COUNT issues"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

    # Will ask the user for confirmation inline
fi
```

## Label Matching Strategy

I use intelligent pattern matching that adapts to ANY repository:

### Generic Patterns (Work Everywhere)

- **Bug indicators**: "bug", "error", "broken", "crash", "fail", "issue", "problem"
- **Feature indicators**: "feature", "enhancement", "add", "implement", "support"
- **Documentation**: "docs", "documentation", "readme", "guide", "tutorial"
- **Performance**: "slow", "performance", "optimize", "speed"
- **Security**: "security", "vulnerability", "CVE", "exploit"
- **Testing**: "test", "testing", "spec", "coverage"

### Component Detection

- **Parser/Compiler**: "parse", "compile", "syntax", "grammar"
- **CLI**: "command", "cli", "terminal", "flag", "argument"
- **API**: "api", "endpoint", "rest", "graphql"
- **UI/Frontend**: "ui", "frontend", "interface", "button", "form"
- **Database**: "database", "db", "query", "migration", "schema"
- **Infrastructure**: "ci", "cd", "deploy", "build", "docker"

### Priority Detection

- **Critical**: "critical", "urgent", "blocking", "broken", "crash"
- **High**: "important", "soon", "high priority", "regression"
- **Medium**: "should", "nice to have", "enhancement"
- **Low**: "minor", "low priority", "future", "sometime"

## Error Handling

- **Network failures** → Retry with exponential backoff
- **Rate limiting** → Respect GitHub API limits, pause if needed
- **Permission errors** → Clear instructions for authentication
- **Invalid labels** → Only suggest labels that exist in repo
- **Malformed issues** → Skip gracefully with warning

## Usage Examples

```bash
# Preview what labels would be applied (safe, no changes)
/git:label-issues --dry-run

# Preview first 10 issues only
/git:label-issues --dry-run --limit 10

# Apply labels to unlabeled issues (asks for confirmation)
/git:label-issues

# Apply labels to first 20 issues
/git:label-issues --limit 20
```

## Output Format

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🏷️  Issue Labeling Analysis
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Repository: your-org/your-repo
Unlabeled Issues: 7

─────────────────────────────────────────
Issue #43: Add deprecation warnings for old schema fields
URL: https://github.com/your-org/your-repo/issues/43

Analysis:
• Discusses schema evolution and compiler warnings
• Mentions parser and code generation
• Feature enhancement, not a bug

Suggested Labels:
  ✓ area:core     (mentions parser, compiler, schema)
  ✓ type:feature  (requests new functionality)
  ✓ priority:medium (enhancement, not critical)

Reasoning:
This issue is about adding a new compiler feature to warn
developers about deprecated schema fields. It affects the
core compiler and is a valuable enhancement for migration.

─────────────────────────────────────────
Issue #42: VSCode syntax highlighting broken
URL: https://github.com/your-org/your-repo/issues/42

Analysis:
• User reports broken syntax highlighting
• Specific to VSCode extension
• Affects user experience

Suggested Labels:
  ✓ area:vscode   (VSCode extension issue)
  ✓ type:bug      (something is broken)
  ✓ priority:high (impacts all VSCode users)

Reasoning:
Clear bug report for VSCode extension. Syntax highlighting
is a core feature, and when broken, affects all users.

─────────────────────────────────────────
```

## Advanced Features

### 1. Partial Labeling
If an issue has SOME labels but is missing others:
- Detect missing label categories
- Suggest complementary labels
- Don't remove existing labels

### 2. Confidence Scoring
Each suggestion includes confidence level:
- 🟢 **High confidence** (90%+): Strong keyword matches
- 🟡 **Medium confidence** (60-89%): Contextual inference
- 🔴 **Low confidence** (<60%): Uncertain, needs review

### 3. Batch Processing
For repos with many issues:
- Process in batches of 10
- Show progress bar
- Handle rate limits gracefully

### 4. Learning Mode
Remember user corrections:
- Track which suggestions were accepted/rejected
- Improve future suggestions
- Store in your host's cache dir: `.claude/cache/label-patterns.json` (CC) or `~/.pi/agent/cache/label-patterns.json` (pi) — optional. Detect host via `PI_CODING_AGENT=true` (pi) being set.

## Integration with Workflows

This command complements the auto-labeling workflow:

- **Workflow**: Labels NEW issues automatically
- **This command**: Labels EXISTING unlabeled issues

## Permissions Required

- `repo` scope (read issues)
- `repo:write` scope (apply labels) - only when not in dry-run mode

## Limitations

- Cannot create new labels (use `gh label create` for that)
- Cannot remove existing labels (only adds missing ones)
- Respects repository's existing label taxonomy
- Works best with repos that have clear label conventions

## Philosophy

> **Ihsan in Organization**: Every issue deserves proper categorization. Good labels help maintainers prioritize work and help contributors find tasks that match their skills.

> **Amanah in Automation**: This tool suggests labels but respects human judgment. Always review suggestions before applying, especially for complex or nuanced issues.

## Expected Outcomes

After running this command:

1. ✅ **Clear Overview** - Understand which issues lack labels
2. ✅ **Intelligent Suggestions** - AI-powered label recommendations
3. ✅ **Safe Application** - Preview mode prevents mistakes
4. ✅ **Better Organization** - Properly categorized issue tracker
5. ✅ **Improved Workflow** - Easier to filter and prioritize issues

MashaAllah! This command brings order to chaos, helping teams maintain clean and organized issue trackers across ANY repository.

---

**Note**: This command is completely generic and adapts to any repository's label structure. It never assumes specific labels exist and always validates against the actual labels in your repo.

*"Indeed, Allah loves those who are organized and systematic in their work."*

Alhamdulillah! Let's bring organization and clarity to your issue tracker, InshaAllah! 🏷️
