---
name: git-tools-standup
description: Morning standup - PRs, Issues, and activity digest across all GitHub orgs with actionable suggestions
argument-hint: "[timeframe] [--style=A|B|C|D] [--external]"
allowed-tools: ["bash", "read", "write", "edit"]
---

# Morning Standup

Bismillah! Time to catch up on what's happening across your GitHub universe.

Arguments provided: $ARGUMENTS

## Argument Parsing

```
TIMEFRAME (positional):
- No argument     → Last workday (smart default)
- "24h" or "1d"   → Last 24 hours
- "7d"            → Last 7 days
- "YYYY-MM-DD"    → Since specific date

FLAGS:
--style=X or -s X → Table style (A, B, C, D). Default: A
--external or -e  → Expand external contributions (show full table)

STYLE OPTIONS:
A = Pure ASCII (default) - 100% consistent, +--+ borders
B = Fixed-Width Unicode  - ┌──┐ borders, truncated text, emojis in headers
C = Minimal Lines        - Clean ─── separators, most compact
D = Status Codes         - Unicode borders + 4-char status codes with legend

Last Workday Calculation:
- Monday    → Friday (3 days ago)
- Sunday    → Friday (2 days ago)
- Tue-Sat   → Yesterday (1 day ago)
```

## Execution Plan

1. **Parse Arguments**
   - Extract timeframe (positional argument)
   - Extract --style flag (default: A)
   - Check for --external flag

2. **Discover GitHub Scope**
   - Fetch username via `gh api user`
   - Fetch all orgs dynamically via `gh api user/orgs`
   - Personal account always included

3. **Fetch Activity Data** (parallel for speed)
   - PRs you authored (open)
   - PRs requesting your review
   - Issues you authored (open)
   - Issues assigned to you
   - Unread notifications

4. **Categorize and Analyze**
   - Group by: Your Orgs | Personal | External
   - Status: Approved | Changes Requested | Waiting | Draft | Merged
   - Age: Calculate days since creation/update

5. **Render Output** (based on --style flag)
   - Use appropriate table format
   - External: collapsed summary unless --external flag

6. **Generate Today's Actions**
   - Priority-ordered actionable items

## Table Style Specifications

### Style A: Pure ASCII (DEFAULT)

```
================================================================================
  STANDUP · [Day] [Date] · [X] PRs · [Y] reviews · [Z] issues
================================================================================

YOUR PRS
+----------------------------------+--------------------------------+----------+------+
| Repo                             | Title                          | Status   | Age  |
+----------------------------------+--------------------------------+----------+------+
| owner/repo#123                   | PR title here                  | WAITING  | 1d   |
| owner/repo#456                   | Another PR title               | MERGED   | 2d   |
+----------------------------------+--------------------------------+----------+------+

REVIEWS NEEDED
+------------------------------------------------------------------------------+
| None - inbox zero!                                                           |
+------------------------------------------------------------------------------+

ISSUES (showing X of Y)
+----------------------------------+--------------------------------+----------+------+
| Repo                             | Title                          | Type     | Age  |
+----------------------------------+--------------------------------+----------+------+
| owner/repo#789                   | Issue title here               | authored | <1d  |
| owner/repo#101                   | Another issue                  | assigned | 5d   |
+----------------------------------+--------------------------------+----------+------+

EXTERNAL (X PRs)
+------------------------------------------------------------------------------+
| Nx org1 (Xd) | Nx org2 (Xd) | Nx legacy (Xd+)                                |
+------------------------------------------------------------------------------+

TODAY'S ACTIONS
+------------------------------------------------------------------------------+
| 1. CHECK  - repo#123 waiting Xd                                              |
| 2. DECIDE - repo#456 stale Xd                                                |
| 3. FOCUS  - description                                                      |
+------------------------------------------------------------------------------+
```

### Style B: Fixed-Width Unicode

```
════════════════════════════════════════════════════════════════════════════════
  STANDUP · [Day] [Date] · [X] PRs · [Y] reviews · [Z] issues
════════════════════════════════════════════════════════════════════════════════

🚀 YOUR PRS
┌──────────────────────────────┬──────────────────────────┬──────────┬──────┐
│ Repo                         │ Title                    │ Status   │ Age  │
├──────────────────────────────┼──────────────────────────┼──────────┼──────┤
│ owner/repo#123               │ PR title here truncat... │ WAITING  │ 1d   │
│ owner/repo#456               │ Another PR title tru...  │ MERGED   │ 2d   │
└──────────────────────────────┴──────────────────────────┴──────────┴──────┘

👀 REVIEWS NEEDED
┌──────────────────────────────────────────────────────────────────────────────┐
│ None - inbox zero!                                                           │
└──────────────────────────────────────────────────────────────────────────────┘

📋 ISSUES (X/Y)
┌──────────────────────────────┬──────────────────────────┬──────────┬──────┐
│ Repo                         │ Title                    │ Type     │ Age  │
├──────────────────────────────┼──────────────────────────┼──────────┼──────┤
│ owner/repo#789               │ Issue title here tru...  │ authored │ <1d  │
│ owner/repo#101               │ Another issue trunca...  │ assigned │ 5d   │
└──────────────────────────────┴──────────────────────────┴──────────┴──────┘

🌍 EXTERNAL (X PRs)
┌──────────────────────────────────────────────────────────────────────────────┐
│ Nx org1 (Xd) │ Nx org2 (Xd) │ Nx legacy (Xd+)                                │
└──────────────────────────────────────────────────────────────────────────────┘

💡 TODAY'S ACTIONS
┌──────────────────────────────────────────────────────────────────────────────┐
│ 1. CHECK  → repo#123 waiting Xd                                              │
│ 2. DECIDE → repo#456 stale Xd                                                │
│ 3. FOCUS  → description                                                      │
└──────────────────────────────────────────────────────────────────────────────┘
```

### Style C: Minimal Lines

```
══════════════════════════════════════════════════════════════════════════════
  STANDUP · [Day] [Date] · [X] PRs · [Y] reviews · [Z] issues
══════════════════════════════════════════════════════════════════════════════

🚀 YOUR PRS

Repo                                Title                             Status    Age
──────────────────────────────────────────────────────────────────────────────────
owner/repo#123                      PR title here                     WAITING   1d
owner/repo#456                      Another PR title                  MERGED    2d

👀 REVIEWS NEEDED

None - inbox zero!

📋 ISSUES (X/Y)

Repo                                Title                             Type      Age
──────────────────────────────────────────────────────────────────────────────────
owner/repo#789                      Issue title here                  authored  <1d
owner/repo#101                      Another issue                     assigned  5d

🌍 EXTERNAL: Nx org1 (Xd) │ Nx org2 (Xd) │ Nx legacy (Xd+)

💡 TODAY'S ACTIONS
──────────────────────────────────────────────────────────────────────────────────
1. CHECK  → repo#123 waiting Xd
2. DECIDE → repo#456 stale Xd
3. FOCUS  → description
```

### Style D: Status Codes + Legend

```
╔══════════════════════════════════════════════════════════════════════════════╗
║  STANDUP · [Day] [Date] · [X] PRs · [Y] reviews · [Z] issues                 ║
╚══════════════════════════════════════════════════════════════════════════════╝

🚀 YOUR PRS
┌──────────────────────────────────┬──────────────────────────────┬──────┬──────┐
│ Repo                             │ Title                        │  ST  │ Age  │
├──────────────────────────────────┼──────────────────────────────┼──────┼──────┤
│ owner/repo#123                   │ PR title here                │ WAIT │ 1d   │
│ owner/repo#456                   │ Another PR title             │ MRGD │ 2d   │
└──────────────────────────────────┴──────────────────────────────┴──────┴──────┘
  ST: WAIT=Waiting · MRGD=Merged · APRV=Approved · CHGS=Changes Requested · DRFT=Draft

👀 REVIEWS NEEDED: None - inbox zero!

📋 ISSUES (X/Y)
┌──────────────────────────────────┬──────────────────────────────┬──────┬──────┐
│ Repo                             │ Title                        │  TY  │ Age  │
├──────────────────────────────────┼──────────────────────────────┼──────┼──────┤
│ owner/repo#789                   │ Issue title here             │ AUTH │ <1d  │
│ owner/repo#101                   │ Another issue                │ ASGN │ 5d   │
└──────────────────────────────────┴──────────────────────────────┴──────┴──────┘
  TY: AUTH=Authored · ASGN=Assigned

🌍 EXTERNAL (X PRs): Nx org1 · Nx org2 · Nx legacy

╔══════════════════════════════════════════════════════════════════════════════╗
║  💡 TODAY'S ACTIONS                                                          ║
║  1. CHECK  → repo#123 waiting Xd                                             ║
║  2. DECIDE → repo#456 stale Xd                                               ║
║  3. FOCUS  → description                                                     ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

## Column Width Standards

All styles use consistent column widths for alignment:

```
| Column | Width | Truncate | Notes                          |
|--------|-------|----------|--------------------------------|
| Repo   | 34    | Yes      | owner/repo#number format       |
| Title  | 30    | Yes      | Add "..." if truncated         |
| Status | 8     | No       | WAITING, MERGED, etc (Style D: 4 char) |
| Type   | 8     | No       | authored, assigned             |
| Age    | 6     | No       | <1d, 1d, 156d format           |
```

## External Contributions Handling

```
DEFAULT (collapsed):
- Show one-line summary: "Nx org1 (Xd) | Nx org2 (Xd) | Nx legacy (Xd+)"
- Group by repo owner
- Show count and oldest age per group

WITH --external FLAG:
- Show full table with all external PRs
- Same format as "YOUR PRS" section
- Sorted by age (newest first)
```

## gh CLI Commands

```bash
# Get username
gh api user --jq '.login'

# Get all orgs (dynamic)
gh api user/orgs --jq '.[].login'

# PRs authored by user
gh search prs --author=USERNAME --state=open --json number,title,repository,updatedAt,isDraft,url --limit 50

# Get review status for specific PR (do this per PR)
gh pr view NUMBER --repo OWNER/REPO --json reviewDecision,state

# PRs requesting user review
gh search prs --review-requested=USERNAME --state=open --json number,title,repository,author,createdAt,url --limit 50

# Issues authored by user
gh search issues --author=USERNAME --state=open --json number,title,repository,updatedAt,url --limit 50

# Issues assigned to user
gh search issues --assignee=USERNAME --state=open --json number,title,repository,labels,createdAt,url --limit 50

# Notifications count
gh api notifications --jq '[.[] | select(.unread==true)] | length'
```

## Today's Actions Priority

```
Priority 1 - MERGE: PRs with APPROVED status
Priority 2 - FIX: PRs with CHANGES_REQUESTED
Priority 3 - REVIEW: PRs waiting for your review (oldest first)
Priority 4 - FOCUS: High-priority assigned issues
Priority 5 - CLEANUP: Stale items (>7 days)
```

## External Detection Logic

```
1. Get user's orgs list + personal account username
2. For each PR/Issue, extract repo owner from nameWithOwner
3. If owner NOT in user's orgs/username → External contribution
```

## Edge Cases

| Scenario | Handling |
|----------|----------|
| No activity | "All clear! Inbox zero - time for deep work" |
| API rate limited | Show partial results + warning |
| No PRs but issues | Show issues section only |
| Weekend standup | Smart lookback to Friday |
| Invalid --style | Warn and fallback to A |

Now execute this standup workflow with the parsed arguments.

IMPORTANT:
- Parse the style flag from arguments (default: A)
- Use printf with fixed widths for consistent alignment
- NO emojis inside table data cells (only in section headers for B, C, D)
- Truncate long text with "..." suffix
- External contributions: collapsed unless --external flag present
