---
name: workspace-organize-docs
description: Intelligently analyze and organize all .md files in repository (read-only advisor)
argument-hint: "[--apply] [--dry-run] [--backup] [--config CONFIG_FILE]"
allowed-tools: ["bash", "read", "write", "edit"]
---

# Documentation Organization Advisor v2

Bismillah! I'll analyze all markdown files in your repository and propose an intelligent organization structure.

Arguments provided: $ARGUMENTS

---

## 🛡️ Safety Guarantees

**This command is READ-ONLY by default:**
- ✅ Scans all .md files in repository
- ✅ Analyzes content and categorizes intelligently
- ✅ Proposes organization structure
- ✅ Shows file movement plan with previews
- ✅ Detects and lists internal link updates needed
- ❌ **NEVER moves files without explicit confirmation**
- ❌ **NEVER modifies content automatically**

**All operations:**
- Display proposed changes for review
- Create backup before any moves (if --apply used)
- Update internal links after confirmation
- Preserve git history (uses git mv)

---

## Organization Philosophy

### Flexible Structure Approaches

This command supports multiple documentation philosophies:

#### 1. **Default Structure** (Tutorial/Documentation-Heavy)
```
repository-root/
├── README.md              ← Project overview (KEEP IN ROOT)
├── AGENTS.md              ← AI context file (KEEP IN ROOT; CLAUDE.md is CC's legacy name — keep it too if present)
├── LICENSE                ← Legal (keep in root)
├── CHANGELOG.md           ← Version history (keep in root)
│
└── docs/                  ← ALL other documentation here
    ├── README.md          ← Docs index/navigation
    ├── guides/            ← How-to guides and tutorials
    ├── references/        ← API docs, command references
    ├── planning/          ← PRD, execution plans
    ├── technical/         ← Architecture documentation
    └── examples/          ← Example code, tutorials
```

#### 2. **Divio Documentation System** (Recommended for Libraries/Frameworks)
```
docs/
├── README.md
├── tutorials/          ← Learning-oriented (getting started)
├── how-to/             ← Problem-oriented (specific tasks)
├── explanation/        ← Understanding-oriented (concepts)
└── reference/          ← Information-oriented (technical specs)
```

#### 3. **Microsoft-Style** (Large Projects)
```
docs/
├── README.md
├── get-started/        ← Quickstart, installation
├── concepts/           ← Core ideas and architecture
├── samples/            ← Code examples
├── reference/          ← API reference
└── resources/          ← Additional materials
```

#### 4. **API-First** (Backend/API Projects)
```
docs/
├── README.md
├── api/                ← Endpoint documentation
│   ├── authentication.md
│   ├── endpoints/
│   └── schemas/
├── deployment/         ← Hosting and deployment
├── development/        ← Local setup and contributing
└── guides/             ← Integration guides
```

#### 5. **Operations/SRE** (Internal Tools/Infrastructure)
```
docs/
├── README.md
├── runbooks/           ← Operational procedures
├── architecture/       ← System design
├── troubleshooting/    ← Common issues
├── monitoring/         ← Alerts and dashboards
└── security/           ← Security policies
```

---

## Configuration Support

### Option 1: Auto-Detection (Recommended)

If no config file exists, the command will:
1. Analyze repository type (language, package files, existing docs)
2. Detect project category:
   - Library/Framework → Divio system
   - API/Backend → API-first structure
   - DevOps/Infra → Operations structure
   - Tutorial/Learning → Default structure
3. Propose appropriate organization
4. Ask for confirmation before categorizing

### Option 2: Configuration File (`.docs-organize.yml`)

Create `.docs-organize.yml` in repository root:

```yaml
# Documentation organization configuration

# Choose structure type: default, divio, microsoft, api-first, operations, custom
structure_type: divio

# Files to always keep in repository root (regex patterns)
keep_in_root:
  - README\.md
  - AGENTS\.md
  - CLAUDE\.md          # CC's legacy name — keep too if present
  - CHANGELOG\.md
  - CONTRIBUTING\.md
  - CODE_OF_CONDUCT\.md
  - LICENSE
  - SECURITY\.md

# Custom category definitions (only if structure_type: custom)
categories:
  tutorials:
    keywords: ["tutorial", "getting started", "introduction", "beginner"]
    patterns: ["*tutorial*", "*getting-started*", "*intro*"]
    weight: 0.7

  how-to:
    keywords: ["how to", "guide", "step by step", "walkthrough"]
    patterns: ["*guide*", "*howto*", "*how-to*"]
    weight: 0.6

  reference:
    keywords: ["api", "reference", "specification", "documentation"]
    patterns: ["*api*", "*reference*", "*spec*"]
    weight: 0.8

# Categorization weights (how to score files)
categorization:
  filename_weight: 0.6      # Weight for filename pattern matching
  content_weight: 0.3       # Weight for content analysis
  location_weight: 0.1      # Weight for current location context
  min_confidence: 0.5       # Minimum confidence to auto-categorize

# Link update behavior
links:
  update_markdown: true     # Update markdown links
  update_html: false        # Update HTML <a> tags
  update_images: true       # Update image references
  check_external: false     # Check external links (slow)

# Safety settings
safety:
  require_git: true         # Require git repository
  check_uncommitted: true   # Check for uncommitted changes
  create_backup: true       # Create backup branch
  backup_prefix: "docs-reorg-backup"
  confirmation_required: true  # Require "YES I UNDERSTAND"

# Exclusions
exclude:
  - node_modules
  - vendor
  - .git
  - dist
  - build
  - target
  - __pycache__
  - venv
  - .venv
```

### Option 3: Interactive Detection

If no config exists and auto-detection is uncertain:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 PROJECT TYPE DETECTION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Detected:
  - Language: TypeScript
  - Project: React component library
  - Existing docs: API references, examples

Suggested Structure: Divio Documentation System
  ✓ tutorials/ - Getting started guides
  ✓ how-to/ - Specific task instructions
  ✓ explanation/ - Conceptual documentation
  ✓ reference/ - API documentation

Does this fit your project? (y/n/suggest alternative):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Implementation Process

### Phase 0: Configuration and Detection 🛫

#### 0.1: Parse Arguments
```bash
APPLY_CHANGES=false
DRY_RUN=true
CREATE_BACKUP=true
CONFIG_FILE=".docs-organize.yml"

for arg in $ARGUMENTS; do
  case $arg in
    --apply)
      APPLY_CHANGES=true
      DRY_RUN=false
      ;;
    --dry-run)
      DRY_RUN=true
      APPLY_CHANGES=false
      ;;
    --backup)
      CREATE_BACKUP=true
      ;;
    --no-backup)
      CREATE_BACKUP=false
      ;;
    --config)
      shift
      CONFIG_FILE="$1"
      ;;
    --config=*)
      CONFIG_FILE="${arg#*=}"
      ;;
  esac
done
```

#### 0.2: Load or Generate Configuration

**Step 1: Check for existing config**
```bash
if [ -f "$CONFIG_FILE" ]; then
  echo "✅ Found configuration: $CONFIG_FILE"
  # Parse YAML config (use Read tool)
else
  echo "📋 No configuration found, using auto-detection..."
  # Proceed to project type detection
fi
```

**Step 2: Auto-detect project type if no config**

Use these heuristics:
1. **Check package.json/Cargo.toml/pyproject.toml** - Language and project type
2. **Analyze existing docs/** - Current organization patterns
3. **Count doc types** - More API docs vs more tutorials
4. **Check for patterns**:
   - `openapi.yml` or `swagger.json` → API-first structure
   - `Dockerfile`, `k8s/`, `.github/workflows/` → Operations structure
   - `examples/`, `tutorials/` → Library/learning structure
   - `docs/api/`, `docs/endpoints/` → API structure

**Step 3: Present detected structure and confirm**

#### 0.3: Repository Validation

```bash
# Verify git repo
if ! git rev-parse --is-inside-work-tree >/dev/null 2>&1; then
  if [ "$REQUIRE_GIT" = true ]; then
    echo "❌ Not a git repository (required by config)"
    exit 1
  else
    echo "⚠️  Not a git repository"
    echo ""
    echo "This command uses 'git mv' to preserve file history."
    echo "Recommendation: Initialize git first or proceed with caution."
    echo ""
    read -p "Continue without git? (y/N): " CONTINUE
    if [[ ! "$CONTINUE" =~ ^[Yy]$ ]]; then
      exit 0
    fi
    GIT_AVAILABLE=false
  fi
else
  GIT_AVAILABLE=true
fi

# Check for uncommitted changes
if [ "$GIT_AVAILABLE" = true ] && [ "$CHECK_UNCOMMITTED" = true ]; then
  if ! git diff-index --quiet HEAD --; then
    echo "⚠️  Uncommitted changes detected"
    echo ""
    echo "Recommendation: Commit or stash changes before reorganizing."
    echo "This prevents confusion about what changed."
    echo ""
    read -p "Continue anyway? (y/N): " CONTINUE
    if [[ ! "$CONTINUE" =~ ^[Yy]$ ]]; then
      exit 0
    fi
  fi
fi
```

---

### Phase 1: Discovery and Analysis 🔍

#### 1.1: Find All Markdown Files

```bash
echo "📂 Scanning for markdown files..."

# Build exclusion patterns from config
FIND_EXCLUDES=""
for exclude_dir in "${EXCLUDE_DIRS[@]}"; do
  FIND_EXCLUDES="$FIND_EXCLUDES -not -path \"*/$exclude_dir/*\""
done

# Find all .md files
ALL_MD_FILES=$(eval "find . -type f -name '*.md' $FIND_EXCLUDES 2>/dev/null | sort")

TOTAL_COUNT=$(echo "$ALL_MD_FILES" | wc -l | tr -d ' ')
echo "Found $TOTAL_COUNT markdown files"
```

#### 1.2: Categorize by Current Location

**Current state analysis:**

```bash
# Root-level .md files
ROOT_DOCS=$(echo "$ALL_MD_FILES" | grep "^\\./[^/]*\\.md$")
ROOT_COUNT=$(echo "$ROOT_DOCS" | grep -c "\\.md$" || echo "0")

# Already in docs/
DOCS_DIR_FILES=$(echo "$ALL_MD_FILES" | grep "^\\./docs/")
DOCS_COUNT=$(echo "$DOCS_DIR_FILES" | grep -c "\\.md$" || echo "0")

# Scattered (outside docs/, not in root)
SCATTERED=$(echo "$ALL_MD_FILES" | grep -v "^\\./[^/]*\\.md$" | grep -v "^\\./docs/")
SCATTERED_COUNT=$(echo "$SCATTERED" | grep -c "\\.md$" || echo "0")

# Files to keep in root (from config)
KEEP_IN_ROOT_PATTERN=$(echo "${KEEP_IN_ROOT[@]}" | sed 's/ /|/g')
KEEP_IN_ROOT=$(echo "$ROOT_DOCS" | grep -E "($KEEP_IN_ROOT_PATTERN)$")
MOVE_FROM_ROOT=$(echo "$ROOT_DOCS" | grep -v -E "($KEEP_IN_ROOT_PATTERN)$")
MOVE_FROM_ROOT_COUNT=$(echo "$MOVE_FROM_ROOT" | grep -c "\\.md$" || echo "0")
```

**Summary:**
- Total: {TOTAL_COUNT} files
- In root (keep): {KEEP_COUNT} files
- In root (should move): {MOVE_FROM_ROOT_COUNT} files
- In docs/ already: {DOCS_COUNT} files
- Scattered elsewhere: {SCATTERED_COUNT} files

---

#### 1.3: Intelligent Categorization

**For each file, analyze using configured weights:**

```python
# Pseudo-code for categorization logic

def categorize_file(file_path, config):
    scores = {}

    for category_name, category_config in config.categories.items():
        # Filename scoring
        filename_score = match_patterns(file_path, category_config.patterns)

        # Content scoring (read first 50 lines)
        content = read_file_preview(file_path, lines=50)
        content_score = match_keywords(content, category_config.keywords)

        # Location scoring
        location_score = match_location_context(file_path, category_config)

        # Weighted total
        total_score = (
            filename_score * config.categorization.filename_weight +
            content_score * config.categorization.content_weight +
            location_score * config.categorization.location_weight
        ) * category_config.weight

        scores[category_name] = total_score

    # Get best match
    best_category = max(scores, key=scores.get)
    confidence = scores[best_category]

    if confidence >= config.categorization.min_confidence:
        return (best_category, confidence, "High" if confidence > 0.7 else "Medium")
    else:
        return (None, confidence, "Low")
```

**Confidence levels:**
- **High** (>0.7): Clear category match (name + content align)
- **Medium** (0.5-0.7): Category inferred (partial match)
- **Low** (<0.5): Unclear category (needs manual review)

---

### Phase 2: Proposal Generation 📝

#### 2.1: Build Movement Plan

**For each file to move:**
```
Source: ./path/to/current/DOC.md
Destination: docs/category/doc.md
Reasoning: [category match explanation]
Confidence: High/Medium/Low
Categorization:
  - Filename match: 0.8 (matched pattern: *guide*)
  - Content match: 0.6 (keywords: tutorial, how to, step by step)
  - Location match: 0.2 (in scattered location)
  - Total score: 0.68 → Medium confidence
```

#### 2.2: Detect Conflicts

**Check for:**
- Destination file already exists
- Name conflicts within category
- Case-insensitive duplicates
- Similar names that may confuse

**Resolution strategies:**
- Append suffix: `doc.md` → `doc-v2.md`
- Merge similar files (with approval)
- Keep both with descriptive names

#### 2.3: Analyze Internal Links

**Scan all .md files for links (based on config):**
- Relative links: `[text](../other.md)`
- Absolute links: `[text](/docs/file.md)`
- Reference links: `[text]: other.md`
- Image links: `![alt](../images/pic.png)` (if update_images: true)
- HTML links: `<a href="...">` (if update_html: true)

**Build link update map:**
```
File: src/component/GUIDE.md (moving to docs/tutorials/component.md)
Links to update:
  - [Setup](../../SETUP.md) → [Setup](../how-to/setup.md)
  - [API](../api/API.md) → [API](../reference/api.md)
```

**Link update strategy:**
- Preserve link targets after file moves
- Update relative paths accordingly
- Verify linked files still exist
- Warn about broken links

---

### Phase 3: Generate Report 📊

**Comprehensive reorganization report:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📚 DOCUMENTATION ORGANIZATION ANALYSIS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Repository: example-project
Structure Type: Divio Documentation System (auto-detected)
Total Markdown Files: 23
Analysis: 2025-10-14 10:30 UTC

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 CURRENT STATE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Files in root: 5
  ✅ README.md (keep)
  ✅ AGENTS.md (keep)
  ⚠️  INSTALLATION.md (should move)
  ⚠️  TROUBLESHOOTING.md (should move)
  ⚠️  CREDITS.md (should move)

Files already in docs/: 3
  ✅ docs/API.md (will reorganize to docs/reference/api.md)
  ✅ docs/SETUP.md (will reorganize to docs/tutorials/setup.md)
  ✅ docs/FAQ.md (will reorganize to docs/explanation/faq.md)

Scattered files: 15
  ⚠️  tmux/TMUX_GUIDE.md
  ⚠️  zsh/SHELL_CONFIG.md
  ⚠️  claude/.claude/COMMANDS.md
  ⚠️  scripts/SCRIPTS_README.md
  ... [11 more files]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 PROPOSED ORGANIZATION (Divio System)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Total moves: 18 files
Updates needed: 12 internal links

📂 Root (2 files - no changes)
  ✅ README.md
  ✅ AGENTS.md

📂 docs/ (new Divio structure - 18 files)
  │
  ├── 📁 tutorials/ (7 files) - Learning-oriented
  │   ├── getting-started.md   ← MOVE: ./INSTALLATION.md
  │   ├── tmux-setup.md        ← MOVE: ./tmux/TMUX_GUIDE.md
  │   ├── shell-config.md      ← MOVE: ./zsh/SHELL_CONFIG.md
  │   ├── first-steps.md       ← MOVE: ./docs/SETUP.md
  │   └── ...
  │
  ├── 📁 how-to/ (4 files) - Problem-oriented
  │   ├── troubleshooting.md   ← MOVE: ./TROUBLESHOOTING.md
  │   ├── configure-tmux.md    ← MOVE: ./tmux/CONFIG.md
  │   └── ...
  │
  ├── 📁 explanation/ (3 files) - Understanding-oriented
  │   ├── architecture.md      ← MOVE: ./ARCHITECTURE.md
  │   ├── faq.md               ← MOVE: ./docs/FAQ.md
  │   └── ...
  │
  └── 📁 reference/ (4 files) - Information-oriented
      ├── api.md               ← REORGANIZE: ./docs/API.md
      ├── commands.md          ← MOVE: ./claude/.claude/COMMANDS.md
      ├── scripts.md           ← MOVE: ./scripts/SCRIPTS_README.md
      └── ...

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔗 INTERNAL LINKS TO UPDATE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

12 files contain links that need updating:

1. README.md
   Current links work, no updates needed ✅

2. INSTALLATION.md → docs/tutorials/getting-started.md
   Links to update:
   - [Troubleshooting](TROUBLESHOOTING.md)
     → [Troubleshooting](../how-to/troubleshooting.md)
   - [Tmux Guide](tmux/TMUX_GUIDE.md)
     → [Tmux Setup](tmux-setup.md)

... [10 more files with link updates]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  POTENTIAL ISSUES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. 🟡 Low Confidence: scripts/NOTES.md (score: 0.42)
   Category: Unclear (contains mixed content)
   Suggested: docs/explanation/scripts-notes.md
   Recommendation: Review content manually

2. 🟢 Broken Link: INSTALLATION.md links to DEPRECATED.md (file not found)
   Action: Remove link or update after reorganization

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
💡 RECOMMENDATIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. 📋 Create docs/README.md as Navigation Hub

2. 🗂️  Standardize File Naming
   Recommended convention:
   - All lowercase
   - Hyphens for spaces: getting-started.md
   - Descriptive names: tmux-setup.md

3. 🔍 Review Low-Confidence Categorizations
   Files needing manual review: 1
   - scripts/NOTES.md (mixed content)

4. 💾 Save Configuration
   Run: --config .docs-organize.yml --save
   Preserves detected structure for future use

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🚀 NEXT STEPS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Choose an option:

1️⃣ Show Detailed Plan for Specific File
   Example: "Show details for tmux/TMUX_GUIDE.md"

2️⃣ Preview Link Updates
   Show exact link changes for all files

3️⃣ Export This Report
   Save to: docs-reorganization-plan-2025-10-14.md

4️⃣ Save Configuration
   Create .docs-organize.yml with detected settings

5️⃣ Apply Reorganization (with backup)
   Execute the moves and link updates

6️⃣ Apply with Manual Review
   Step-by-step confirmation for each move

7️⃣ Try Different Structure
   Switch to: [default/microsoft/api-first/operations]

8️⃣ Cancel
   No changes will be made

Which option? (1-8)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Phase 4: Interactive Options

[Rest of the implementation remains similar to v1, but with:]
- Configuration-aware execution
- Structure-specific categorization
- Flexible category handling
- Project-type detection feedback

---

## Generated docs/README.md

**Structure-specific navigation hub based on detected type:**

### For Divio Structure:
```markdown
# Documentation

Complete documentation for [Project Name] using the [Divio Documentation System](https://documentation.divio.com/).

## 📚 Quick Navigation

### 🎓 Tutorials - *Learning-oriented*
When you want to learn how to use the project.

- [Getting Started](tutorials/getting-started.md)
- [Your First Project](tutorials/first-project.md)

### 🔧 How-To Guides - *Problem-oriented*
When you need to solve a specific problem.

- [Troubleshooting](how-to/troubleshooting.md)
- [Configuration](how-to/configuration.md)

### 💡 Explanation - *Understanding-oriented*
When you want to understand concepts and decisions.

- [Architecture](explanation/architecture.md)
- [Design Decisions](explanation/design-decisions.md)
- [FAQ](explanation/faq.md)

### 📖 Reference - *Information-oriented*
When you need technical specifications.

- [API Reference](reference/api.md)
- [CLI Commands](reference/commands.md)

---

**Documentation Structure**: Divio Documentation System
- **Tutorials**: Step-by-step lessons for beginners
- **How-To Guides**: Recipes for specific tasks
- **Explanation**: Background and context
- **Reference**: Technical descriptions

Last updated: 2025-10-14
Generated by: `/docs:organize` command
```

### For API-First Structure:
```markdown
# API Documentation

Complete API documentation for [Project Name].

## 🚀 Quick Start

- [Authentication](api/authentication.md)
- [Getting Started](get-started/quickstart.md)

## 📡 API Reference

### Core APIs
- [Users API](api/endpoints/users.md)
- [Projects API](api/endpoints/projects.md)

### Schemas
- [Request/Response Schemas](api/schemas/)

## 🛠️ Development

- [Local Setup](development/setup.md)
- [Contributing](development/contributing.md)

## 🚢 Deployment

- [Production Deployment](deployment/production.md)
- [Environment Variables](deployment/configuration.md)

---

Last updated: 2025-10-14
Generated by: `/docs:organize` command
```

---

## Advanced Features

### 1. Project Type Auto-Detection

**Detection heuristics:**

```python
def detect_project_type(repo_path):
    indicators = {
        "library": 0,
        "api": 0,
        "operations": 0,
        "learning": 0,
        "application": 0
    }

    # Check package files
    if exists("package.json"):
        package = read_json("package.json")
        if "library" in package.keywords or package.get("main"):
            indicators["library"] += 3
        if "express" in package.dependencies or "fastify" in package.dependencies:
            indicators["api"] += 3

    # Check for API indicators
    if exists("openapi.yml") or exists("swagger.json"):
        indicators["api"] += 5

    # Check for operations indicators
    if exists("Dockerfile") or exists_dir("k8s") or exists_dir(".github/workflows"):
        indicators["operations"] += 3

    # Check existing docs structure
    if exists_dir("docs/tutorials") or exists_dir("docs/examples"):
        indicators["library"] += 2
        indicators["learning"] += 2

    if exists_dir("docs/api") or exists_dir("docs/endpoints"):
        indicators["api"] += 4

    if exists_dir("docs/runbooks") or exists_dir("docs/operations"):
        indicators["operations"] += 4

    # Return highest scoring type
    return max(indicators, key=indicators.get)
```

### 2. Smart Link Detection

**Handles various markdown link formats:**

```markdown
Standard: [text](path/to/file.md)
Reference: [text][ref]
Reference def: [ref]: path/to/file.md
Image: ![alt](path/to/image.png)
Relative: [text](../../../file.md)
Anchor: [text](file.md#section)
HTML: <a href="file.md">text</a>
```

**Link update algorithm:**
1. Parse current file location
2. Parse link target location
3. Calculate relative path after moves
4. Update link in file
5. Preserve anchors and query strings

### 3. Conflict Resolution

**Strategy for conflicts:**

1. **Same name, different content**
   - Compare file content (diff)
   - Offer: merge, rename, keep both

2. **Similar names**
   - Flag potential confusion
   - Suggest: clarify names

3. **Circular links**
   - Detect and warn
   - Ensure updates don't break

### 4. Configuration Templates

**Generate config for common structures:**

```bash
# Generate configuration template
/docs:organize --generate-config divio
/docs:organize --generate-config microsoft
/docs:organize --generate-config api-first
```

Creates `.docs-organize.yml` with sensible defaults for chosen structure.

---

## Safety Measures

1. **Backup Branch** - Always create backup before moves
2. **Git MV** - Preserve file history
3. **Link Validation** - Verify links before and after
4. **Rollback Option** - Easy to revert
5. **Progress Tracking** - Can resume if interrupted
6. **Dry Run Default** - Must explicitly apply
7. **Configuration Validation** - Check YAML syntax before running
8. **Confirmation Required** - Type "YES I UNDERSTAND" to proceed

---

## Integration with Workflow

✅ **Complements `/planning:sync`:**
- `/planning:sync` detects scattered docs → suggests `/docs:organize`
- `/docs:organize` fixes organization → keeps planning docs synced

✅ **Context-File Alignment (AGENTS.md):**
- "Clarify Before Acting" - Multiple confirmation points
- Read-only by default - Zero risk without --apply
- Evidence-based suggestions - Shows reasoning for each categorization
- Respects existing structure - Preserves intentional organization

✅ **Best Practices:**
- Run after major documentation additions
- Use before public releases for professional appearance
- Integrate into CI/CD for documentation linting
- Generate config and commit for team consistency

---

## Usage Examples

```bash
# Dry run with auto-detection (safe, read-only analysis)
/docs:organize

# Dry run with specific config
/docs:organize --config .docs-organize.yml

# Generate configuration template
/docs:organize --generate-config divio

# Apply reorganization (with confirmation)
/docs:organize --apply

# Apply with backup (recommended)
/docs:organize --apply --backup

# Apply without backup (use with caution)
/docs:organize --apply --no-backup

# Try different structure type
/docs:organize --structure microsoft --dry-run

# Export report without applying
/docs:organize --export docs-plan.md
```

---

## Error Handling

**Common scenarios:**

- **Not a git repo**: Warns, offers to continue without git mv
- **Uncommitted changes**: Warns, suggests commit first
- **Conflicts detected**: Shows conflicts, offers resolutions
- **Broken links**: Lists broken links, suggests fixes
- **Permission errors**: Reports issues, provides solutions
- **Invalid YAML config**: Shows syntax errors, line numbers
- **Low confidence categorization**: Flags for manual review

---

## Comparison: v1 vs v2

| Feature | v1 (Original) | v2 (General Abstraction) |
|---------|---------------|--------------------------|
| **Structure** | Hard-coded (guides/references/planning) | Configurable + Auto-detect |
| **Project Types** | Tutorial/docs-heavy only | 5+ structure types supported |
| **Configuration** | None | YAML config + templates |
| **Categorization** | Keyword-only | Weighted scoring system |
| **Keep-in-root** | Hard-coded list | Configurable regex patterns |
| **Detection** | None | Auto-detect project type |
| **Flexibility** | Low | High (custom categories) |
| **Safe for any repo** | Mostly (but opinionated) | Yes (adapts to project) |

---

Alhamdulillah! This v2 command brings intelligent, flexible organization to any documentation repository, adapting to your project's needs while maintaining safety and clarity. InshaAllah, it will help you maintain excellent documentation organization across diverse project types! 📚✨
