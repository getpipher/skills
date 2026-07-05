---
name: work-loc
description: Count lines of code (LOC) across multiple repositories using tokei
argument-hint: "[repo-list|path|--summary]"
allowed-tools: ["bash", "read", "write", "edit"]
---

# LOC Counter - Lines of Code Statistics

Bismillah! Count lines of code across repositories or directories using `tokei`.

Arguments provided: $ARGUMENTS

---

## Command Purpose

This command:
- Counts LOC across one or multiple repositories
- Uses `tokei` for accurate, language-aware counting
- Outputs detailed breakdown by language
- Supports summary mode for quick totals

---

## Implementation

### Step 1: Check tokei Installation

First, verify `tokei` is installed:

```bash
if ! command -v tokei &> /dev/null; then
    echo "tokei not found. Installing via cargo..."
    cargo install tokei
fi
```

### Step 2: Parse Arguments

The command accepts:
- **No args:** Count LOC in current directory
- **Repo names:** Space-separated list of repo names (looked up in `$LOC_WORKSPACE`, default `~/local-dev`)
- **Paths:** Absolute or relative paths to directories
- **--summary:** Only show totals, not per-repo breakdown
- **--json:** Output as JSON for parsing

### Step 3: Execute Count

**For current directory (no args):**
```bash
tokei .
```

**For specific repos:**
```bash
for repo in $ARGUMENTS; do
    path="${LOC_WORKSPACE:-$HOME/local-dev}/$repo"
    if [ -d "$path" ]; then
        echo "=== $repo ==="
        tokei "$path" --compact
    fi
done
```

**For organization-wide (example with your-org):**
```bash
REPOS="your-app your-api your-docs your-cli your-sdk your-website"
for repo in $REPOS; do
    path="${LOC_WORKSPACE:-$HOME/local-dev}/$repo"
    if [ -d "$path" ]; then
        echo "=== $repo ==="
        tokei "$path" --compact
        echo ""
    fi
done
```

### Step 4: Generate Summary Table

After counting all repos, generate a markdown summary table with:
- Repository name
- Total files
- Total lines
- Code lines
- Comments
- Blanks

---

## Usage Examples

```bash
# Count LOC in current directory
/stats:loc

# Count LOC in specific repos
/stats:loc your-app your-api

# Count with summary only (no per-language breakdown)
/stats:loc --summary

# Count in absolute path
/stats:loc /path/to/project

# Count all your-org repos
/stats:loc your-org
```

---

## Expected Output

**Per-repo breakdown:**
```
=== your-repo ===
 Language    Files   Lines    Code  Comments  Blanks
 Rust          42   22408   18127      1151    3130
 Markdown      40   16042       0     11568    4474
 ...
```

**Summary table:**
```
| Repository | Files | Lines | Code | Comments | Blanks |
|------------|-------|-------|------|----------|--------|
| your-repo  | 103   | 49888 | 25633| 15340    | 8915   |
| TOTAL      | 378   | 120K  | 70K  | 31K      | 18K    |
```

---

## Special Keywords

- `your-org` - Count all your-org organization repos
- `--compact` - Use compact tokei output
- `--summary` - Show only final summary table
- `--json` - Output raw JSON for parsing

---

## Notes

- **Performance:** Fast for most repos (< 5s per repo)
- **Prerequisites:** `tokei` must be installed (auto-installs via cargo if missing)
- **Excludes:** Auto-ignores .git, node_modules, target, vendor directories
- **Languages:** Supports 200+ programming languages

---

**Template Version:** 1.0
**Created:** 2025-12-07

Alhamdulillah!
