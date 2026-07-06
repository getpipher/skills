---
name: workspace-planning-sync
description: Analyze repo and provide intelligent planning documentation suggestions (read-only)
argument-hint: "[--force] [--type=<repo-type>] [--export]"
allowed-tools: ["bash", "read", "write", "edit"]
---

# Adaptive Planning Advisor (Read-Only Analysis)

Bismillah! I'll intelligently analyze your codebase and provide comprehensive planning documentation suggestions without modifying any files.

Arguments provided: $ARGUMENTS

---

## 🛡️ Safety Guarantees

**This command is READ-ONLY by default:**
- ✅ Analyzes codebase structure
- ✅ Detects repository type
- ✅ Evaluates existing documentation
- ✅ Generates suggestions and previews
- ✅ Provides actionable recommendations
- ❌ **NEVER writes files without explicit confirmation**
- ❌ **NEVER modifies existing documentation automatically**

**All outputs are:**
- Displayed to you for review
- Saved to `/tmp/` as drafts (if requested)
- Require your explicit approval before creating files

---

## Supported Repository Types

This command adapts to different repository types:

✅ **Product Development** - Web/mobile apps (Epic → Story → Task)
✅ **Library/SDK** - APIs and packages (Module → Feature → Function)
✅ **CLI Tools** - Command-line apps (Command Group → Command → Flag)
✅ **Infrastructure** - IaC and deployment (Environment → Resource → Config)
✅ **Configuration** - Dotfiles and configs (Tool → Config Area → Setting)
✅ **Documentation** - Static sites (Section → Page → Content)
✅ **Data Pipeline** - ETL and processing (Pipeline → Stage → Transform)
✅ **Multi-purpose** - Mixed repos (Feature Area → Feature → Component)

---

## Using with Thinking Mode

When executing with **mode: think** or **mode: think hard**, I will:

- **Repository Type Intelligence**: Deep analysis to detect repo purpose
- **Context-Aware Suggestions**: Recommendations matching your workflow
- **Adaptive Structure Proposals**: Terminology fitting repo type
- **Strategic Gap Identification**: Find planning blind spots
- **Evidence-Based Analysis**: Every finding backed by code references
- **Performance-Aware Processing**: Chunk large repos intelligently

---

## Implementation Process

### Phase 0: Pre-Flight Validation 🛫

**Verify assumptions before expensive analysis**

#### 0.1: Parse Command Arguments

```bash
FORCE_MODE=false
OVERRIDE_TYPE=""
EXPORT_MODE=false

for arg in $ARGUMENTS; do
  case $arg in
    --force)
      FORCE_MODE=true
      ;;
    --type=*)
      OVERRIDE_TYPE="${arg#*=}"
      ;;
    --export)
      EXPORT_MODE=true
      ;;
  esac
done
```

#### 0.2: Repository Size Check

```bash
# Count files (exclude common ignore patterns)
FILE_COUNT=$(find . -type f \
  -not -path "*/\.*" \
  -not -path "*/node_modules/*" \
  -not -path "*/vendor/*" \
  -not -path "*/venv/*" \
  -not -path "*/__pycache__/*" \
  -not -path "*/target/*" \
  -not -path "*/dist/*" \
  -not -path "*/build/*" \
  2>/dev/null | wc -l | tr -d ' ')

echo "📊 Repository Size: $FILE_COUNT files"

if [ "$FILE_COUNT" -gt 5000 ]; then
  echo ""
  echo "⚠️  Large repository detected"
  echo "Analysis may take 5-10 minutes"
  echo ""
  if [ "$FORCE_MODE" = false ]; then
    echo "Options:"
    echo "1. Continue with full analysis"
    echo "2. Sample analysis (faster, representative subset)"
    echo "3. Cancel"
    read -p "Choice (1/2/3): " SIZE_CHOICE
    case $SIZE_CHOICE in
      2) SAMPLE_MODE=true ;;
      3) exit 0 ;;
      *) SAMPLE_MODE=false ;;
    esac
  fi
fi
```

**Performance Estimates:**
- <1,000 files: ~30 seconds
- 1,000-5,000 files: 1-3 minutes
- 5,000-10,000 files: 5-10 minutes
- >10,000 files: Use sampling mode

#### 0.3: Git Repository Validation

```bash
if ! git rev-parse --is-inside-work-tree >/dev/null 2>&1; then
  echo ""
  echo "⚠️  Not a git repository"
  echo ""
  echo "Limitations without git:"
  echo "  - Cannot determine completion dates"
  echo "  - Cannot detect in-progress work (branches/PRs)"
  echo "  - Progress based on file timestamps only"
  echo ""
  if [ "$FORCE_MODE" = false ]; then
    read -p "Continue with limited analysis? (y/N): " GIT_CONTINUE
    if [[ ! "$GIT_CONTINUE" =~ ^[Yy]$ ]]; then
      echo "Analysis cancelled"
      exit 0
    fi
  fi
  GIT_AVAILABLE=false
else
  GIT_AVAILABLE=true

  # Check for shallow clone
  if [ -f .git/shallow ]; then
    echo "⚠️  Shallow git clone - dates may be inaccurate"
  fi

  # Check commit count
  COMMIT_COUNT=$(git rev-list --count HEAD 2>/dev/null || echo "0")
  if [ "$COMMIT_COUNT" -lt 5 ]; then
    echo "⚠️  Very few commits ($COMMIT_COUNT) - limited history"
  fi
fi
```

#### 0.4: Repository Type Detection

**Intelligent detection using multiple signals:**

```bash
# Technology indicators
HAS_PACKAGE_JSON=$([ -f "package.json" ] && echo true || echo false)
HAS_CARGO_TOML=$([ -f "Cargo.toml" ] && echo true || echo false)
HAS_GO_MOD=$([ -f "go.mod" ] && echo true || echo false)
HAS_PYPROJECT=$([ -f "pyproject.toml" ] && echo true || echo false)
HAS_GEMFILE=$([ -f "Gemfile" ] && echo true || echo false)

# Structure indicators
HAS_SRC=$([ -d "src" ] && echo true || echo false)
HAS_LIB=$([ -d "lib" ] && echo true || echo false)
HAS_CMD=$([ -d "cmd" ] && echo true || echo false)
IS_DOTFILES=$(basename "$PWD" | grep -iq "dotfile" && echo true || echo false)

# Purpose indicators (simplified for read-only check)
HAS_ROUTES=$(find . -name "*route*" -o -name "*controller*" 2>/dev/null | head -1)
HAS_CLI=$(find . -name "*cli*" -o -name "*command*" 2>/dev/null | head -1)
HAS_TERRAFORM=$(find . -name "*.tf" 2>/dev/null | head -1)

# Classification
REPO_TYPE="multi-purpose"

if [ "$IS_DOTFILES" = true ]; then
  REPO_TYPE="configuration"
elif [ -n "$HAS_TERRAFORM" ]; then
  REPO_TYPE="infrastructure"
elif [ -n "$HAS_CLI" ] && [ "$HAS_CMD" = true ]; then
  REPO_TYPE="cli-tool"
elif [ "$HAS_LIB" = true ] && [ -z "$HAS_ROUTES" ]; then
  REPO_TYPE="library"
elif [ -n "$HAS_ROUTES" ]; then
  REPO_TYPE="product"
fi

# Allow override
if [ -n "$OVERRIDE_TYPE" ]; then
  REPO_TYPE="$OVERRIDE_TYPE"
  echo "⚙️  Repository type manually set to: $REPO_TYPE"
fi
```

#### 0.5: Pre-Flight Summary

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PRE-FLIGHT VALIDATION COMPLETE

Repository: [name]
Type: [detected type]
Files: [count]
Git: [available/unavailable]
Estimated Time: [time]

Planning Model: [structure for type]
Analysis Mode: READ-ONLY (no files will be modified)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

If `--force` not set, ask: "Ready to proceed with analysis? (y/N)"

---

### Phase 1: Codebase Understanding 🔍

**Adaptive analysis based on repo type**

#### 1.1: Terminology Mapping

| Repo Type | Tier 1 | Tier 2 | Tier 3 | Example |
|-----------|--------|--------|--------|---------|
| **Product** | Epic | Story | Task | Epic: Auth → Story: Login → Task: JWT |
| **Library** | Module | Feature | Function | Module: HTTP → Feature: Retry → Function: backoff() |
| **CLI** | Cmd Group | Command | Flag | Group: Deploy → Cmd: start → Flag: --dry-run |
| **Infrastructure** | Environment | Resource | Config | Env: Prod → Resource: EKS → Config: nodes |
| **Configuration** | Tool | Config Area | Setting | Tool: Tmux → Area: Keys → Setting: prefix |
| **Documentation** | Section | Page | Block | Section: API → Page: Auth → Block: Example |
| **Pipeline** | Pipeline | Stage | Transform | Pipeline: ETL → Stage: Clean → Transform: dedupe() |
| **Multi-purpose** | Area | Feature | Component | Area: CLI → Feature: Sync → Component: Parser |

#### 1.2: Structure Analysis

**Focused discovery based on repo type:**

```bash
case "$REPO_TYPE" in
  product)
    # Routes, controllers, components, services
    FOCUS="user-facing features, API endpoints, UI components"
    ;;
  library)
    # Public APIs, exports, modules
    FOCUS="exported functions, public interfaces, modules"
    ;;
  cli-tool)
    # Commands, subcommands, flags
    FOCUS="commands, CLI parsers, option handlers"
    ;;
  infrastructure)
    # Resources, modules, environments
    FOCUS="IaC resources, configurations, deployments"
    ;;
  configuration)
    # Tools, configs, scripts
    FOCUS="configured tools, dotfiles, automation scripts"
    ;;
  documentation)
    # Sections, pages, content
    FOCUS="content structure, navigation, topics"
    ;;
  data-pipeline)
    # Pipelines, stages, transforms
    FOCUS="ETL jobs, data flows, transformations"
    ;;
  multi-purpose)
    # Feature areas broadly
    FOCUS="functional areas, major components"
    ;;
esac
```

**Analysis Steps:**
1. Directory structure and organization
2. Key files matching repo type focus
3. Technology stack and dependencies
4. Feature/component inventory (adapted to type)
5. Implementation state (completed, WIP, planned)
6. Quality indicators (tests, docs, error handling)

**Performance Optimization:**
- Sample representative files if >1,000 files
- Use parallel analysis for independent areas
- Cache intermediate results

---

### Phase 2: Documentation Discovery 📂

**Find existing planning docs:**

**Search Patterns:**
```
PRD: PRD.md, REQUIREMENTS.md, PRODUCT_REQUIREMENTS.md
Locations: /, /docs/, /planning/, /.github/   (also check host context dir: ~/.claude/ or ~/.pi/agent/ for global notes)

Execution: EXECUTION.md, EXECUTION_PLAN.md, IMPLEMENTATION.md, PROGRESS.md
Locations: /, /docs/, /planning/

Alternative: FEATURES.md, ROADMAP.md, TODO.md, CHANGELOG.md
```

**Validation:**
- File exists and >100 bytes
- Contains structured content
- Follows recognizable hierarchy
- Has progress/status indicators

#### 2.1: Detect Scattered Documentation (Organization Check)

**Scan for all .md files in repository:**

```bash
# Find all .md files excluding common ignore patterns
ALL_MD_FILES=$(find . -type f -name "*.md" \
  -not -path "*/node_modules/*" \
  -not -path "*/vendor/*" \
  -not -path "*/.git/*" \
  -not -path "*/dist/*" \
  -not -path "*/build/*" \
  2>/dev/null)

# Categorize by location
ROOT_DOCS=$(echo "$ALL_MD_FILES" | grep "^\./[^/]*\.md$" | grep -v "./README.md" | grep -v -E "./(CLAUDE|AGENTS)\.md")
DOCS_DIR=$(echo "$ALL_MD_FILES" | grep "^\./docs/")
SCATTERED=$(echo "$ALL_MD_FILES" | grep -v "^\./docs/" | grep -v "^\./README.md$" | grep -v -E "^\./(CLAUDE|AGENTS)\.md$")

# Count scattered docs
SCATTERED_COUNT=$(echo "$SCATTERED" | grep -c "\.md$" || echo "0")
```

**Organization Evaluation:**

**✅ Well-Organized Repository:**
- README.md in root ✓
- AGENTS.md (or CLAUDE.md) in root (if exists) ✓
- All other .md files in docs/ ✓
- No scattered documentation ✓

**⚠️ Needs Organization:**
- .md files scattered across multiple directories
- Documentation mixed with source code
- Inconsistent location patterns
- No clear docs/ directory structure

**Note for report:** If scattered documentation is detected (>3 .md files outside docs/), mention this in the recommendations section and suggest using `/docs:organize` command for comprehensive reorganization.

---

### Phase 3: Accuracy Validation ✅

**(Only if docs exist)**

#### 3.1: Match Documented vs Actual

**For each documented item:**
- Search codebase for implementation
- Verify technical specs match reality
- Check completion status accuracy
- Validate progress percentages

**Example Checks:**
```
Documented: "JWT authentication complete"
Code Search:
  - JWT library installed? ✓
  - Token generation? ✓
  - Validation middleware? ✓
  - Refresh endpoint? ✗ MISSING
Verdict: 75% complete (not 100%)
```

#### 3.2: Find Ghost Features

**Undocumented implementations:**
- Features in code but not in docs
- API endpoints not tracked
- Components without planning entries
- Scripts/configs not documented

#### 3.3: Cross-Reference Validation

**Structural consistency:**
- Task IDs reference valid items
- Progress rollup calculations correct
- No orphaned references
- Bidirectional links work

---

### Phase 4: Gap Analysis 🔍

**Categorize findings:**

**🔴 HIGH PRIORITY:**
- Completed features marked incomplete
- Implemented but undocumented features
- Critical spec mismatches
- Progress >20% off

**🟡 MEDIUM PRIORITY:**
- Task descriptions don't match code
- Progress 10-20% off
- Missing acceptance criteria
- Partial documentation

**🟢 LOW PRIORITY:**
- Formatting issues
- Minor wording improvements
- Timestamp discrepancies

---

### Phase 5: Intelligent Suggestion Generation 💡

**Generate comprehensive report with:**

1. **Current State Assessment**
   - What exists (or doesn't)
   - Accuracy evaluation
   - Completeness percentage

2. **Detected Structure**
   - Reverse-engineered hierarchy
   - Feature inventory with status
   - Progress calculations

3. **Suggested Documentation**
   - Appropriate structure for repo type
   - Sample content previews
   - Recommended file names/locations

4. **Actionable Recommendations**
   - Priority-ordered improvements
   - Specific next steps
   - Copy-paste commands

5. **Interactive Options**
   - Review detailed previews
   - Export to markdown
   - Generate draft files in /tmp/
   - Skip if not needed

---

## Comprehensive Report Format

### Scenario A: No Docs Exist

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 PLANNING ANALYSIS REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Repository: dotfiles
Type: Configuration
Files: 847 files analyzed
Analysis: 2025-10-13 09:30 UTC

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📄 DOCUMENTATION STATUS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

❌ No planning documentation found

Searched locations:
  - /docs/PRD.md ❌
  - /PRD.md ❌
  - /.claude/PRD.md ❌
  - /EXECUTION.md ❌

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 DETECTED STRUCTURE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Overall Progress: 68% (19/28 config areas implemented)

Tool 1: Tmux Configuration
├─ Status: 82% Complete (9/11 config areas)
├─ Config Area 1.1: Session Management ✅
│  └─ Settings: resurrect plugin, auto-save script
├─ Config Area 1.2: Key Bindings ✅
│  └─ Settings: prefix key (C-a), navigation shortcuts
├─ Config Area 1.3: Status Line 🚧
│  └─ Settings: Custom modules (partial), themes
├─ Config Area 1.4: Plugin System ⏳
│  └─ Settings: TPM setup (not started)

Tool 2: Shell Configuration (Zsh)
├─ Status: 70% Complete (7/10 config areas)
├─ Config Area 2.1: Core Setup ✅
│  └─ Settings: .zshrc, PATH config
├─ Config Area 2.2: Aliases 🚧
│  └─ Settings: Basic aliases (incomplete categorization)
├─ Config Area 2.3: Functions ⏳
│  └─ Settings: Custom functions (planned)

Tool 3: Claude Code Configuration
├─ Status: 55% Complete (6/11 config areas)
├─ Config Area 3.1: Slash Commands ✅
│  └─ Settings: 15 commands across 6 categories
├─ Config Area 3.2: Global Config ✅
│  └─ Settings: context-file (AGENTS.md) workflow
├─ Config Area 3.3: Hooks ⏳
│  └─ Settings: Not configured yet

Evidence Files:
  - tmux/.tmux.conf (234 lines)
  - tmux/plugins/ (12 plugins installed)
  - zsh/.zshrc (456 lines)
  - claude/.claude/commands/ (15 slash commands)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
💡 RECOMMENDATIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. 📋 Create Configuration Planning Documentation

   For a dotfiles repo, I recommend tool-based structure:
   - Tool → Config Area → Setting/Script
   - Installation checklist per tool
   - Sync strategy documentation

2. 🎯 Suggested Documentation Files:

   /docs/CONFIG_PLAN.md
     Purpose: Track what you configure and why
     Structure: Tool-based hierarchy
     Status: Progress tracking per tool

   /docs/SETUP.md
     Purpose: Installation and sync instructions
     Structure: Step-by-step per tool
     Status: Bootstrap and maintenance procedures

   /docs/EXECUTION.md (optional)
     Purpose: Track implementation progress
     Structure: Tool → Config Area → Status
     Status: Current completion state

3. ⚠️  Identified Configuration Gaps:

   🔴 HIGH PRIORITY (Finish These):
     - Tmux status line modules incomplete
       Files: tmux/statusline/*.sh
       TODO: Weather and git status modules

     - Shell aliases need organization
       Files: zsh/aliases.zsh
       TODO: Categorize and document purpose

   🟡 MEDIUM PRIORITY (Consider Adding):
     - Claude Code hooks not configured
       Suggested: Pre-commit formatting hook

     - Shell functions lacking documentation
       Files: zsh/functions/*.zsh
       TODO: Add usage comments

   🟢 LOW PRIORITY (Nice to Have):
     - Plugin documentation for installed plugins
     - Theme customization guide

4. 📚 Documentation Organization Issue Detected:

   ⚠️  Found 8 .md files scattered across repository

   **Scattered documentation locations:**
     - tmux/TMUX_SETUP.md
     - zsh/ALIASES_GUIDE.md
     - claude/COMMANDS_REFERENCE.md
     - [5 more files in various directories]

   **Recommended structure:**
     - README.md (root) ← Keep
     - AGENTS.md (root) ← Keep (also CLAUDE.md if present)
     - docs/ ← Move all other .md files here

   **Why organize:**
     - Easier discovery and maintenance
     - Clear separation from code
     - Professional appearance
     - Better documentation navigation

   **How to fix:**
     Run: `/docs:organize` for intelligent reorganization suggestions

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📝 PROPOSED STRUCTURE PREVIEW
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Here's what I suggest for docs/CONFIG_PLAN.md:

```markdown
# Configuration Management Plan

**Repository:** Personal Dotfiles
**Type:** Configuration
**Last Updated:** 2025-10-13
**Overall Progress:** 68% (19/28 config areas)

---

## Tool 1: Tmux Configuration
**Purpose:** Terminal multiplexer with persistent sessions
**Status:** 82% Complete (9/11 areas)
**Priority:** High

### Config Area 1.1: Session Management ✅
**What:** Automatic session save/restore
**Why:** Preserve work across system restarts
**Implementation:**
  - resurrect plugin (tmux/.tmux.conf:45-52)
  - Auto-save script (tmux/scripts/session_save.sh)
  - Restore on tmux start
**Status:** Complete
**Tested:** ✅ Working as of 2024-10-01

### Config Area 1.2: Key Bindings ✅
**What:** Custom keyboard shortcuts
**Why:** Vim-like navigation, ergonomic prefix
**Implementation:**
  - Prefix: C-a (tmux/.tmux.conf:12)
  - Navigation: hjkl pane movement (lines 89-95)
  - Copy mode: vim bindings (lines 120-135)
**Status:** Complete
**Tested:** ✅ Daily use validated

### Config Area 1.3: Status Line 🚧
**What:** Custom status bar with system info
**Why:** At-a-glance system monitoring
**Implementation:**
  - Status modules: tmux/statusline/*.sh
  - Configuration: tmux/.tmux.conf:150-180
  - Theme: Catppuccin integration
**Status:** 60% Complete
**Remaining:**
  - [ ] Weather module (tmux/statusline/weather.sh)
  - [ ] Git status integration
  - [x] CPU/Memory display
  - [x] Date/time formatting
**Blocked by:** API key for weather service
**Next Step:** Complete weather.sh or remove module

### Config Area 1.4: Plugin System ⏳
**What:** TPM (Tmux Plugin Manager) setup
**Why:** Easy plugin management and updates
**Implementation:**
  - TPM installation: Planned
  - Plugin list: tmux/.tmux.conf:200-220
  - Auto-install script: Planned
**Status:** Not started
**Priority:** Medium
**Next Step:** Install TPM and test plugin auto-loading

---

## Tool 2: Shell Configuration (Zsh)
**Purpose:** Primary shell environment
**Status:** 70% Complete (7/10 areas)
**Priority:** High

### Config Area 2.1: Core Setup ✅
**What:** Zsh initialization and PATH configuration
**Why:** Foundation for shell environment
**Implementation:**
  - .zshrc (zsh/.zshrc)
  - PATH additions (lines 10-25)
  - Shell options (lines 30-50)
**Status:** Complete
**Tested:** ✅ Working across macOS and Linux

### Config Area 2.2: Aliases 🚧
**What:** Command shortcuts and abbreviations
**Why:** Faster common operations
**Implementation:**
  - Aliases file: zsh/aliases.zsh
  - Loaded in .zshrc:120
  - 45+ aliases defined
**Status:** 65% Complete
**Issues:** Needs categorization and documentation
**Remaining:**
  - [ ] Categorize by function (git, file ops, system)
  - [ ] Add purpose comments
  - [ ] Remove unused aliases
  - [x] Basic aliases working
**Next Step:** Audit and organize aliases.zsh

### Config Area 2.3: Functions ⏳
**What:** Custom shell functions
**Why:** Complex operations beyond simple aliases
**Implementation:**
  - Functions directory: zsh/functions/
  - Auto-load: Planned in .zshrc
  - 8 function files created
**Status:** 40% Complete
**Issues:** Functions exist but not documented or tested
**Remaining:**
  - [ ] Add usage documentation
  - [ ] Test each function
  - [ ] Set up auto-loading
  - [x] Create function files
**Next Step:** Document function usage and test

---

## Tool 3: Claude Code Configuration
**Purpose:** AI-powered development assistant setup
**Status:** 55% Complete (6/11 areas)
**Priority:** High

### Config Area 3.1: Slash Commands ✅
**What:** Custom Claude Code commands
**Why:** Automate common workflows
**Implementation:**
  - Command directory: claude/.claude/commands/
  - 15 commands across 6 categories:
    - git/ (7 commands): commit, create-pr, merge-pr, etc.
    - init/ (2 commands): repo initialization
    - planning/ (1 command): planning:sync
    - quality/ (2 commands): lint-fix, type-check
    - superteam/ (2 commands): bounty, hackathon
    - meta/ (1 command): make-slash
**Status:** Complete
**Tested:** ✅ All commands functional

### Config Area 3.2: Global Configuration ✅
**What:** CLAUDE.md/AGENTS.md workflow preferences
**Why:** Consistent AI assistance behavior
**Implementation:**
  - Global config: host context file (`~/.claude/CLAUDE.md` for CC, `~/.pi/agent/AGENTS.md` for pi — whichever you maintain as your global agent config)
  - Workflow definitions
  - Code standards and preferences
**Status:** Complete
**Maintained:** Regularly updated

### Config Area 3.3: Hooks ⏳
**What:** Automated Claude Code triggers
**Why:** Pre-commit checks, auto-formatting
**Implementation:**
  - Hook directory: .claude/hooks/ (not created)
  - Pre-commit: Planned linting/formatting
  - Post-write: Planned test triggers
**Status:** Not started
**Priority:** Medium
**Suggested Hooks:**
  - pre-commit: Run linters before commit
  - on-file-write: Trigger related tests
**Next Step:** Research Claude Code hook capabilities

---

## Priority Matrix

| Priority | Tool | Config Area | Effort | Impact |
|----------|------|-------------|--------|--------|
| 🔴 HIGH | Tmux | Status line modules | 2h | High |
| 🔴 HIGH | Shell | Organize aliases | 1h | Medium |
| 🟡 MED | Tmux | TPM plugin system | 1.5h | Medium |
| 🟡 MED | Shell | Document functions | 2h | Medium |
| 🟡 MED | Claude | Configure hooks | 3h | High |
| 🟢 LOW | All | Plugin docs | 4h | Low |

## Next Session Goals

1. **Immediate (This Week):**
   - Complete tmux status line (finish weather.sh)
   - Organize and document shell aliases
   - Test all shell functions

2. **Short-term (This Month):**
   - Set up TPM for tmux plugins
   - Configure Claude Code hooks
   - Document all configuration choices

3. **Long-term (This Quarter):**
   - Create comprehensive setup guide
   - Automate dotfiles installation
   - Add configuration testing suite

---

## Notes

- All configurations tested on macOS Sonoma 14.6
- Linux compatibility validated on Ubuntu 22.04
- Plugin versions pinned in respective config files
- Breaking changes documented in CHANGELOG.md

Alhamdulillah! Configuration management is 68% complete.
```

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🚀 NEXT STEPS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Choose an option:

1️⃣ Generate Draft Documentation
   Create full CONFIG_PLAN.md in /tmp/ for your review
   (No files written to your repo yet)

2️⃣ Show Detailed Analysis for Specific Tool
   Example: "Show detailed Tmux analysis"

3️⃣ Export This Report to Markdown
   Save to: planning-sync-report-2025-10-13.md

4️⃣ Create Documentation in Repository
   Review drafts first, then write to docs/

5️⃣ Skip Planning Documentation
   Dotfiles may not need formal tracking

Which option? (1-5 or 'help' for details)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Scenario B: Docs Exist but Inaccurate

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 PLANNING SYNCHRONIZATION REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Repository: my-app
Type: Product Development
Files: 1,234 analyzed
Analysis: 2025-10-13 09:45 UTC

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📄 DOCUMENTATION STATUS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Planning documents found:
  - /docs/PRD.md (Last updated: 2024-09-15)
  - /docs/EXECUTION.md (Last updated: 2024-09-20)

⚠️  Documentation Accuracy: 67% (needs sync)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 DISCREPANCY ANALYSIS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Found 12 discrepancies across documentation:

🔴 HIGH PRIORITY (5 issues):

1. Epic 2, Story 2.3 - Marked In Progress but Complete
   **Location:** PRD.md:78-95
   **Documented:** Status: In Progress (60%)
   **Reality:** Fully implemented and merged to main
   **Evidence:**
     - All tasks completed (commits: abc123, def456, ghi789)
     - Merged 2024-10-01
     - Tests passing (15/15)
   **Recommendation:** Update to "Status: Complete (100%)"

2. Missing Epic 4 - Notification System Undocumented
   **Location:** PRD.md (not present)
   **Reality:** Complete notification system exists
   **Evidence:**
     - Implementation: src/services/notifications/
     - WebSocket server: src/websocket.js
     - 12 notification types
     - UI components in src/components/Notifications/
     - Tests: notifications.test.js (passing)
   **Recommendation:** Add new Epic 4 with Stories

3. PRD Claims OAuth Integration - Not Found in Code
   **Location:** PRD.md:120-135, Story 1.4
   **Documented:** "OAuth integration with Google/GitHub"
   **Reality:** No OAuth implementation found
   **Evidence:**
     - No OAuth library in package.json
     - No OAuth routes or handlers
     - No provider configuration
   **Recommendation:** Remove from PRD or add to pending

4. Progress Tracking Off by 18%
   **Location:** EXECUTION.md:15
   **Documented:** "Overall Progress: 55%"
   **Reality:** Actual completion is 73%
   **Evidence:**
     - 48/67 tasks complete (71.6%)
     - 3 epics fully done, 1 at 80%
   **Recommendation:** Update overall and per-Epic percentages

5. Eight Completed Tasks Not Marked Done
   **Location:** EXECUTION.md throughout
   **Evidence:**
     - Task 1.2.3: JWT refresh (complete, merged 2024-09-28)
     - Task 2.1.2: Payment webhook (complete, merged 2024-10-05)
     - Task 2.3.1: Subscription UI (complete, merged 2024-10-08)
     - [5 more tasks listed with evidence]
   **Recommendation:** Batch update task statuses

🟡 MEDIUM PRIORITY (4 issues):

6. Task 1.3.2 Description Doesn't Match Code
   **Location:** PRD.md:145, Task 1.3.2
   **Documented:** "Password reset via email"
   **Reality:** Implemented via SMS also
   **Evidence:** src/services/auth/reset.js supports email + SMS
   **Recommendation:** Update description to mention SMS

[Additional medium priority items...]

🟢 LOW PRIORITY (3 issues):

11. Formatting Inconsistencies in Task Lists
   **Location:** EXECUTION.md throughout
   **Issue:** Mixed checkbox styles (- [ ], * [ ], - [])
   **Recommendation:** Standardize to - [ ] format

[Additional low priority items...]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
💡 RECOMMENDATIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. 🔄 Update Documentation Immediately

   Documentation is 3+ weeks stale. High priority issues
   create confusion about actual project state.

2. 📝 Suggested Update Approach:

   Option A: Batch update all HIGH priority issues
     - Update 5 critical discrepancies
     - Time estimate: 15 minutes
     - Impact: Accuracy → 85%

   Option B: Comprehensive sync (all 12 issues)
     - Fix all discrepancies
     - Time estimate: 30 minutes
     - Impact: Accuracy → 98%

   Option C: Interactive review (per-issue approval)
     - Review and approve each change
     - Time estimate: 45 minutes
     - Impact: 100% confidence in updates

3. ⏰ Suggested Maintenance Schedule:

   To prevent this:
   - Sync docs weekly (every Monday)
   - Update EXECUTION.md after each PR merge
   - Run /planning:sync before major releases

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📝 PROPOSED UPDATES PREVIEW
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Here's what I would change in PRD.md:

**Change 1: Update Epic 2, Story 2.3 Status**
File: docs/PRD.md
Location: Lines 78-80

Current:
```markdown
### Story 2.3: Subscription Management
**Status:** In Progress (60%)
**Last Updated:** 2024-09-15
```

Proposed:
```markdown
### Story 2.3: Subscription Management
**Status:** Complete (100%)
**Completed:** 2024-10-01
**Last Updated:** 2024-10-13
```

Rationale: All tasks complete, merged, and tested.

---

**Change 2: Add Missing Epic 4**
File: docs/PRD.md
Location: After Epic 3 (new section)

Proposed addition:
```markdown
## Epic 4: Real-time Notifications
**Objective:** Provide real-time user notifications via WebSocket
**Priority:** High
**Status:** Complete (100%)

### Story 4.1: WebSocket Infrastructure
**As a** system
**I want** reliable WebSocket connections
**So that** real-time notifications are delivered

**Implementation Evidence:**
- WebSocket server: src/websocket.js
- Connection handling: src/services/notifications/connection.js
- Tests: src/tests/websocket.test.js (12/12 passing)

#### Tasks:
- [x] Task 4.1.1: WebSocket server setup
- [x] Task 4.1.2: Connection authentication
- [x] Task 4.1.3: Reconnection logic

### Story 4.2: Notification Types
[Full story structure...]
```

Rationale: Complete feature discovered in codebase.

---

**Change 3: Remove OAuth Story (Not Implemented)**
File: docs/PRD.md
Location: Lines 120-135

Current:
```markdown
### Story 1.4: OAuth Integration
**As a** user
**I want** to login with Google/GitHub
**So that** I don't need another password

#### Tasks:
- [ ] Task 1.4.1: OAuth provider setup
- [ ] Task 1.4.2: Callback handling
- [ ] Task 1.4.3: Account linking
```

Action: REMOVE this story entirely
Or: Move to "Future Enhancements" section

Rationale: No implementation found, should not be in active PRD.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🚀 NEXT STEPS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Choose an option:

1️⃣ Generate Complete Update Proposal
   Create detailed diff with all changes in /tmp/

2️⃣ Show Detailed Evidence for Specific Issue
   Example: "Show evidence for issue #2"

3️⃣ Export This Report
   Save to: planning-sync-report-2025-10-13.md

4️⃣ Apply Updates to Documentation
   Review proposals → Confirm → Write to docs/

5️⃣ Create Update Branch for Review
   Generate git branch with proposed changes for PR

Which option? (1-5 or 'help')
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Scenario C: Docs Exist and Accurate

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 PLANNING SYNCHRONIZATION REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Repository: my-project
Type: Library/SDK
Files: 456 analyzed
Analysis: 2025-10-13 10:00 UTC

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📄 DOCUMENTATION STATUS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Planning documents found and verified:
  - /docs/LIBRARY_SPEC.md (Last updated: 2024-10-10)
  - /docs/IMPLEMENTATION.md (Last updated: 2024-10-12)

✅ Documentation Accuracy: 98% (excellent!)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 VALIDATION RESULTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Verified:
  ✅ 4 Modules documented and match implementation
  ✅ 15 Features accurately described
  ✅ 48 Functions match public API
  ✅ Progress tracking accurate (87% complete)
  ✅ All completion dates verified against git history
  ✅ No ghost features found
  ✅ Cross-references valid

Minor findings:
  🟢 2 recently added functions not yet documented
  🟢 1 deprecated function still in docs (marked)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
💡 RECOMMENDATIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. ✅ Excellent Documentation Quality!

   Your planning documents are accurate and up-to-date.
   Last sync was 1 day ago - great maintenance cadence.

2. 🟢 Minor Improvements (Optional):

   Add documentation for 2 new functions:
     - calculateRetryDelay() in src/retry.js:45
     - isRetryableError() in src/errors.js:23

   These were added in recent commits but not yet documented.

3. 📅 Suggested Next Review: October 20, 2024

   Current sync frequency: Every 3-4 days (optimal)
   Continue this cadence to maintain accuracy.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🚀 NEXT STEPS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Options:

1️⃣ Add Documentation for New Functions
   Generate entries for 2 undocumented functions

2️⃣ Export Validation Report
   Save this report for audit trail

3️⃣ Done - Documentation is Current
   No action needed, continue excellent work!

Which option? (1-3)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

MashaAllah! Your planning documentation is in excellent shape! 🎉
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Interactive Options After Report

### Option 1: Generate Draft Documentation

**User selects Option 1:**

```bash
# Create drafts in /tmp/ for review
mkdir -p /tmp/planning-sync-drafts
echo "Creating draft documentation..."

# Generate CONFIG_PLAN.md draft
cat > /tmp/planning-sync-drafts/CONFIG_PLAN.md <<'EOF'
[Full generated content]
EOF

# Generate EXECUTION.md draft (if applicable)
cat > /tmp/planning-sync-drafts/EXECUTION.md <<'EOF'
[Full generated content]
EOF

echo "✅ Draft documentation created"
echo ""
echo "📁 Location: /tmp/planning-sync-drafts/"
echo ""
echo "Review files:"
echo "  - CONFIG_PLAN.md"
echo "  - EXECUTION.md"
echo ""
echo "Next steps:"
echo "1. Review drafts: cd /tmp/planning-sync-drafts && ls -la"
echo "2. Edit if needed: open /tmp/planning-sync-drafts/CONFIG_PLAN.md"
echo "3. Copy to repo: cp /tmp/planning-sync-drafts/*.md docs/"
echo ""
echo "Or run: /planning:sync --apply-drafts"
```

### Option 2: Show Detailed Analysis

**User selects "Show detailed Tmux analysis":**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 DETAILED ANALYSIS: Tmux Configuration
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Tool:** Tmux
**Purpose:** Terminal multiplexer with session management
**Overall Status:** 82% Complete (9/11 config areas)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📂 FILE STRUCTURE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

tmux/
├── .tmux.conf (234 lines, last modified: 2024-10-10)
│   ├── Core settings (lines 1-30)
│   ├── Key bindings (lines 31-120)
│   ├── Status line config (lines 121-180)
│   └── Plugin definitions (lines 181-234)
│
├── plugins/ (12 plugins installed)
│   ├── tpm/ (plugin manager)
│   ├── tmux-resurrect/
│   ├── tmux-continuum/
│   └── ... (9 more plugins)
│
├── statusline/ (custom status modules)
│   ├── cpu.sh (✅ Complete)
│   ├── memory.sh (✅ Complete)
│   ├── git_status.sh (🚧 Partial)
│   └── weather.sh (❌ Incomplete, 15 lines)
│
└── scripts/
    ├── session_save.sh (✅ Complete, tested)
    └── session_restore.sh (✅ Complete, tested)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ COMPLETE CONFIG AREAS (9)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. Session Management
   - resurrect plugin configured
   - Auto-save every 15 minutes
   - Restore on tmux start
   - Verified: ✅ Working
   - Last test: 2024-10-01

2. Key Bindings
   - Prefix: C-a (ergonomic for Vim users)
   - Navigation: hjkl pane switching
   - Copy mode: Vim bindings
   - Verified: ✅ Daily use

3. Color Scheme
   - Theme: Catppuccin Mocha
   - True color support enabled
   - Consistent with terminal theme
   - Verified: ✅ Renders correctly

4. Pane Management
   - Split bindings: | and -
   - Resize shortcuts: M-hjkl
   - Automatic pane naming
   - Verified: ✅ Functional

5. Window Management
   - Base index: 1 (not 0)
   - Auto-renumber windows
   - Last-window shortcut: prefix + L
   - Verified: ✅ Works as expected

6. Copy Mode
   - Vim-style selection
   - System clipboard integration
   - Search keybindings
   - Verified: ✅ Copy/paste working

7. Mouse Support
   - Mouse enabled for scrolling
   - Pane selection with mouse
   - Resize panes with mouse drag
   - Verified: ✅ Functions properly

8. Status Line (CPU/Memory)
   - CPU usage display
   - Memory usage display
   - Update interval: 5 seconds
   - Verified: ✅ Accurate readings

9. Plugin Management
   - 12 plugins installed
   - All plugins functional
   - No conflicts detected
   - Verified: ✅ All working

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🚧 IN-PROGRESS CONFIG AREAS (1)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

10. Status Line (Git Status) - 50% Complete
    **What:** Display current git branch and status in status bar
    **Why:** At-a-glance repo awareness
    **Implementation:**
      - Script: tmux/statusline/git_status.sh (45 lines)
      - Config: .tmux.conf:165-170
      - Format: " branch_name ✓ modified_files"
    **Status:** Partially working
    **Issues:**
      - Branch name displays ✅
      - Modified file count not working ❌
      - Update interval too long (60s, should be 5s) ⚠️
    **Remaining Tasks:**
      - [ ] Fix file count detection (line 32 bug)
      - [ ] Reduce update interval in config
      - [ ] Add uncommitted changes indicator
      - [x] Display branch name
      - [x] Basic git detection
    **Estimated Time:** 30 minutes
    **Blocker:** None
    **Next Step:** Debug file count logic in git_status.sh:32

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⏳ PENDING CONFIG AREAS (1)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

11. Status Line (Weather) - 0% Complete
    **What:** Display current weather in status bar
    **Why:** Convenient weather at-a-glance
    **Implementation:**
      - Script: tmux/statusline/weather.sh (started)
      - Config: .tmux.conf:171-175 (commented out)
      - API: OpenWeatherMap (requires key)
    **Status:** Not started
    **Issues:**
      - Need API key from openweathermap.org
      - Script skeleton exists (15 lines) but not functional
      - Integration not tested
    **Remaining Tasks:**
      - [ ] Get OpenWeatherMap API key
      - [ ] Complete weather.sh implementation
      - [ ] Test API integration
      - [ ] Add error handling for network failures
      - [ ] Configure update interval
      - [ ] Uncomment in .tmux.conf
    **Estimated Time:** 2 hours (including API setup)
    **Blocker:** Need API key
    **Next Step:** Sign up for OpenWeatherMap API
    **Alternative:** Remove weather module if not needed

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 PROGRESS BREAKDOWN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Complete:    9 areas  ████████████████████████████  82%
In Progress: 1 area   ███                           9%
Pending:     1 area   ███                           9%

Total: 11 config areas

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 RECOMMENDATIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Priority Actions:

1. 🔴 Fix Git Status Module (30 min)
   Quick win to complete status line
   File: tmux/statusline/git_status.sh:32
   Bug: File count not detecting staged changes
   Fix: Change `git diff --name-only` to `git diff --staged --name-only`

2. 🟡 Decide on Weather Module (5 min decision)
   Options:
   a) Get API key and complete (2 hours)
   b) Remove module and call it done (5 minutes)
   c) Replace with simpler time/date module (30 minutes)

   My recommendation: Option B or C
   Weather APIs add complexity and failure points

3. 🟢 Document Tmux Config (15 min)
   Create tmux/README.md with:
   - Purpose of each plugin
   - Key binding reference
   - Troubleshooting tips
   - Update instructions

After completing #1, Tmux config will be 91% complete!

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Option 3: Export Report

**User selects Option 3:**

```bash
REPORT_FILE="planning-sync-report-$(date +%Y-%m-%d).md"

# Save the entire report to markdown file
cat > "$REPORT_FILE" <<'EOF'
[Full report content with all sections]
EOF

echo "✅ Report exported to: $REPORT_FILE"
echo ""
echo "You can:"
echo "  - Review: cat $REPORT_FILE"
echo "  - Share: Email or attach to issue"
echo "  - Archive: mv $REPORT_FILE docs/reports/"
```

### Option 4: Apply Updates (with confirmation)

**User selects Option 4:**

```
⚠️  CONFIRMATION REQUIRED

This will modify files in your repository:
  - docs/CONFIG_PLAN.md (will be CREATED)
  - docs/EXECUTION.md (will be CREATED)

Preview:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
File: docs/CONFIG_PLAN.md
Size: ~8.5 KB
Lines: ~245
Content: Tool-based configuration tracking
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

View full content before creating? (y/N):
[If yes, show full preview]

Proceed with file creation? (yes/NO):
[Require explicit "yes" to proceed]

[If confirmed]
✅ Created docs/CONFIG_PLAN.md
✅ Created docs/EXECUTION.md

Files written successfully!

Next step: Review and commit
  git add docs/CONFIG_PLAN.md docs/EXECUTION.md
  git commit -m "Add configuration planning documentation"
```

---

## Command Arguments

**Supported flags:**
- `--force` - Skip all confirmations (use with caution)
- `--type=<type>` - Override auto repo type detection
- `--export` - Automatically export report to markdown file

**Valid types:** product, library, cli-tool, infrastructure, configuration, documentation, data-pipeline, multi-purpose

---

## Context-File Alignment

✅ Aligns with the project context file (AGENTS.md):
- "Clarify Before Acting" - Multiple confirmation points
- Read-only by default - Zero risk
- Evidence-based recommendations
- Respects your PRD/Execution Plan workflow
- Adapts to repo type intelligently

✅ Supports your planning workflow:
- Maintains Epic → Story → Task hierarchy
- Or adapts to Tool → Config → Setting
- Or any appropriate structure per repo type
- Tracks progress accurately
- Provides actionable next steps

Alhamdulillah! This read-only advisor ensures you maintain accurate planning documentation while keeping full control over what gets created and when. InshaAllah, it will bring clarity to your development process! 🚀
