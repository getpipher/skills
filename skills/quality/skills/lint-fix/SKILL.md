---
name: quality-lint-fix
description: Run linter and auto-fix all issues in the project
---

# Project Linter and Auto-Fix

Bismillah! I'll run appropriate linters and auto-fix all issues in your project.

Arguments: [--backup] [--check-only] [file-pattern] — the target paths/globs supplied when invoking this skill.

## Implementation Steps

Let me:

1. **Project Analysis and Linter Detection**
   - Scan project structure to identify language/framework
   - Detect available linters (ESLint, Prettier, Black, rustfmt, gofmt, etc.)
   - Check for configuration files (.eslintrc, .prettierrc, pyproject.toml, etc.)
   - Identify package.json scripts or Makefile targets

2. **Backup Preparation** (if --backup flag provided)
   - Create timestamped backup of files before modification
   - Store backup location for rollback if needed

3. **Multi-Language Linting Strategy**
   - **JavaScript/TypeScript**: ESLint + Prettier integration
   - **Python**: Black, isort, flake8 with auto-fix capabilities
   - **Rust**: rustfmt + clippy suggestions
   - **Go**: gofmt, goimports, golangci-lint
   - **Shell**: shellcheck with applicable fixes
   - **Markdown**: markdownlint with auto-fix
   - **YAML/JSON**: yamllint, jsonlint with formatting

4. **Intelligent Execution Flow**
   - Run formatters first (Prettier, Black, rustfmt)
   - Follow with linters that can auto-fix (ESLint --fix)
   - Report remaining issues that need manual intervention
   - Show statistics: files processed, issues fixed, remaining warnings

5. **Comprehensive Reporting**
   - Display before/after comparison
   - List all fixes applied by category
   - Highlight remaining manual fixes needed
   - Provide next steps if configuration is missing

## Smart Detection Logic

### JavaScript/TypeScript Projects

```bash
# Check for package.json and common config files
if [[ -f "package.json" ]]; then
  # ESLint detection and execution
  if command -v eslint &> /dev/null && [[ -f ".eslintrc"* || $(grep -q "eslint" package.json) ]]; then
    npx eslint . --fix --ext .js,.jsx,.ts,.tsx
  fi
  
  # Prettier detection and execution
  if command -v prettier &> /dev/null && [[ -f ".prettierrc"* || $(grep -q "prettier" package.json) ]]; then
    npx prettier --write "**/*.{js,jsx,ts,tsx,json,css,md}"
  fi
fi
```

### Python Projects

```bash
# Check for Python project indicators
if [[ -f "pyproject.toml" || -f "requirements.txt" || -f "setup.py" ]]; then
  # Black formatting
  if command -v black &> /dev/null; then
    black .
  fi
  
  # isort import sorting
  if command -v isort &> /dev/null; then
    isort .
  fi
  
  # flake8 linting (reporting only)
  if command -v flake8 &> /dev/null; then
    flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
  fi
fi
```

### Rust Projects

```bash
# Check for Cargo.toml
if [[ -f "Cargo.toml" ]]; then
  # rustfmt formatting
  if command -v rustfmt &> /dev/null; then
    cargo fmt
  fi
  
  # clippy linting with suggestions
  if command -v cargo-clippy &> /dev/null; then
    cargo clippy --fix --allow-dirty --allow-staged
  fi
fi
```

## Advanced Features

### Configuration Auto-Setup

If no linter configuration found:

- Offer to create basic .eslintrc.js for JS projects
- Suggest pyproject.toml setup for Python projects
- Recommend clippy configuration for Rust projects

### Selective Fixing

Support file pattern arguments:

Invoke with: `quality-lint-fix "src/**/*.js"` — only JavaScript files in src
Invoke with: `quality-lint-fix "*.py"` — only Python files in root
Invoke with: `quality-lint-fix --check-only` — report issues without fixing

### Git Integration

- Check for staged files and offer to lint only those
- Respect .gitignore patterns
- Warn about uncommitted changes before major fixes

## Error Handling

Common scenarios and responses:

- **No linters found**: Provide installation instructions for detected project type
- **Configuration missing**: Offer to create basic configuration files
- **Permission issues**: Guide user to resolve file permissions
- **Syntax errors**: Report files with syntax errors that prevent linting
- **Conflicting configs**: Detect and report configuration conflicts

## Success Criteria

The command completes successfully when:

- ✅ All available linters executed without errors
- ✅ Auto-fixable issues resolved
- ✅ Comprehensive report generated
- ✅ Backup created if requested
- ✅ Remaining manual fixes clearly identified

## Usage Examples

Invoke with: `quality-lint-fix` — basic usage, fix everything
Invoke with: `quality-lint-fix --backup` — create backup before fixing
Invoke with: `quality-lint-fix --check-only` — check issues without fixing
Invoke with: `quality-lint-fix "src/**/*.ts"` — fix specific file pattern
Invoke with: `quality-lint-fix --staged` — fix only staged git files

## Integration with Other Commands

Works well with:

- `git-tools-commit` — lint before committing
- `git-tools-create-pr` — ensure clean code before PR
- CI lint jobs — lint as part of CI pipeline

This command provides comprehensive, intelligent linting that adapts to your project structure and available tools. Alhamdulillah, it will help maintain consistent code quality across different languages and frameworks!
