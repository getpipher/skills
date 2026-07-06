---
name: git-tools-commit
description: Commit and push current changes to remote repository
argument-hint: [commit message]
allowed-tools: ["bash", "read", "write", "edit"]
---

Bismillah! I'll commit and push the current changes to the remote repository.

Arguments provided: $ARGUMENTS

## Using with Thinking Mode

When executing this command with **mode: think**, I will:

- Analyze each changed file thoroughly before staging
- Review the logical grouping of changes for better commit organization
- Consider if changes should be split into multiple commits for clarity
- Examine the impact and relationships between modified files
- Craft more descriptive commit messages based on the deeper analysis
- Verify that all changes align with the intended purpose

This thoughtful approach ensures higher quality commits with better documentation and logical organization.

Let me:

1. **Check repository status**
   - Verify we're in a git repository
   - Check for staged and unstaged changes
   - Review current branch status

2. **Analyze version bump necessity**
   - Examine changes to determine if version increment is needed
   - Analyze file modifications, additions, and deletions
   - Check for breaking changes, new features, or bug fixes
   - Review package.json, Cargo.toml, or similar version files
   - Provide intelligent suggestions based on change analysis:
     - **Major bump** (X.0.0): Breaking changes, removed APIs, incompatible changes
     - **Minor bump** (0.X.0): New features, added APIs, backward-compatible enhancements
     - **Patch bump** (0.0.X): Bug fixes, small improvements, security patches
     - **No bump needed**: Documentation, formatting, tests, internal refactoring
   - Consider conventional commit patterns in existing history
   - Alert user if version bump is recommended with specific reasoning

   **Version Bump Detection Logic:**
   - Scan git diff for API signature changes, removed exports, or breaking modifications
   - Check for new public functions, classes, or modules (minor bump indicators)
   - Look for bug fix patterns in code changes (patch bump indicators)
   - Analyze package.json/Cargo.toml/pyproject.toml for dependency changes
   - Review CHANGELOG.md or similar files for version patterns
   - Examine file extensions (.js, .ts, .py, .rs, .go) vs documentation (.md, .txt)
   - Parse conventional commits (feat:, fix:, BREAKING CHANGE:) if present
   - Consider migration files, schema changes, or configuration updates
   - Evaluate test additions/changes as feature indicators
   - Check for security-related fixes requiring immediate patch bumps

3. **Documentation Impact Analysis**
   - Analyze staged changes to identify potential documentation impacts
   - Discover related documentation files and cross-references
   - Detect outdated content that needs updating
   - Suggest specific documentation improvements

   **Documentation Discovery Logic:**
   - Find README.md, CHANGELOG.md, docs/ folders, *.md files in project
   - Locate API documentation, JSDoc, docstrings, inline comments
   - Identify configuration files (package.json, Config.toml, etc.)
   - Check for example files, tutorials, and usage guides
   - Map relationships between code files and their documentation

   **Outdated Content Detection:**
   - **Function/Class/API Changes**: Scan docs for references to renamed/deleted functions, classes, or APIs
   - **Import/Export Changes**: Find documentation examples using old import paths or module names
   - **Configuration Changes**: Detect outdated config examples, environment variables, or settings
   - **File Path Changes**: Identify broken file path references in documentation
   - **Version References**: Find hardcoded version numbers that need updating
   - **Installation Instructions**: Check if install commands, dependencies, or setup steps changed
   - **Code Examples**: Verify that code snippets still work with current implementation
   - **API Signatures**: Check for function signatures that changed in examples
   - **CLI Commands**: Validate command-line examples against actual script/tool changes
   - **Feature Descriptions**: Ensure feature lists match actual implemented functionality

   **Documentation Analysis Rules:**

   ```
   High Priority Updates:
   - Broken code examples → Fix immediately
   - Non-existent function references → Remove/update
   - Incorrect API signatures → Update signatures
   - Wrong file paths → Fix paths
   - Outdated installation steps → Update instructions

   Medium Priority Updates:
   - Version number mismatches → Update versions
   - Missing new features → Add documentation
   - Deprecated feature mentions → Mark as deprecated
   - Outdated configuration examples → Update configs

   Low Priority Updates:
   - Minor wording improvements → Suggest enhancements
   - Additional examples → Propose new examples
   - Cross-reference opportunities → Suggest links
   ```

   **Intelligent Content Mapping:**
   - For each modified code file, find documentation that references it
   - Extract function/class names from code changes and search docs
   - Check import statements and find corresponding documentation
   - Analyze configuration changes and find related config docs
   - Map CLI argument changes to command documentation
   - Link test file changes to testing documentation

4. **Comprehensive Change Decision Point**
   - Present complete analysis with actionable recommendations
   - Show version bump analysis AND documentation impact assessment
   - Provide user with clear options for comprehensive updates

   **Decision Matrix:**

   ```
   Scenario A: Version bump needed + Documentation outdated
   → "🔍 **Comprehensive Update Needed**"
   → "Version: X.Y.Z → A.B.C | Documentation: N outdated references found"
   → Options: 1) Fix all now, 2) Version bump only, 3) Docs only, 4) Commit as-is

   Scenario B: Version bump needed + Documentation current  
   → "🔍 **Version Bump Detected**: X.Y.Z → A.B.C | Documentation appears current"
   → Options: 1) Bump version first, 2) Commit as-is

   Scenario C: No version bump + Documentation outdated
   → "📚 **Documentation Updates Needed**: N outdated references found"
   → Options: 1) Update docs first, 2) Commit as-is

   Scenario D: No version bump + Documentation current
   → Proceed directly to commit
   ```

   **Smart Recommendations:**
   - If breaking changes detected: "⚠️ BREAKING CHANGE: Consider major version bump + comprehensive doc review"
   - If new features added: "✨ NEW FEATURES: Minor version bump + feature documentation recommended"
   - If API examples outdated: "🔧 API EXAMPLES: Update code examples before release"
   - If installation changed: "📦 INSTALLATION: Update setup instructions for users"

5. **Execute Selected Actions** (based on user choice)
   - **Option 1 - Fix All**: Update version + documentation + commit + push
   - **Option 2 - Version Only**: Bump version + commit + push (leave docs for later)
   - **Option 3 - Docs Only**: Update documentation + commit + push (leave version for later)
   - **Option 4 - Commit As-Is**: Direct commit + push (user accepts current state)

6. **Documentation Update Execution** (when selected)
   - Automatically fix high-priority documentation issues
   - Update broken code examples with working implementations
   - Fix incorrect API signatures and function references
   - Update file paths and import statements
   - Correct version references and installation instructions
   - Present medium/low priority suggestions for user review

7. **Smart Commit Message Generation**
   - Analyze all changes to create comprehensive commit message
   - Include version bump information if applicable
   - Summarize documentation updates if performed
   - Follow conventional commit format when appropriate
   - Use provided argument as override if specified

8. **Push to remote** (only after successful commit)
   - Push current branch to remote repository
   - Handle upstream branch setup if needed
   - **Dual-push**: If configured, `git push` automatically syncs to both GitHub and GitLab

9. **Comprehensive Completion Report**
   - Show final status and remote sync confirmation
   - Summarize what was updated (code + version + docs)
   - Highlight any remaining recommendations for future commits
   - Provide links to updated documentation if applicable

## Implementation Steps

**Step 1:** Repository Analysis

- Check git repository status and analyze staged changes
- Identify modified, added, and deleted files
- Extract function/class/API changes from diffs

**Step 2:** Version Bump Analysis

```
Version Bump Rules:
- MAJOR (X.0.0): Breaking changes, API removal, incompatible modifications
- MINOR (0.X.0): New features, API additions, backward-compatible enhancements  
- PATCH (0.0.X): Bug fixes, small improvements, security patches
- NONE: Documentation only, formatting, comments, tests only

Smart Detection:
- API signature changes → MAJOR
- New public functions/features → MINOR
- Bug fixes in logic → PATCH
- install.sh enhancements → MINOR
- Configuration changes → MINOR/PATCH
- Documentation only → NONE
```

**Step 3:** Documentation Impact Analysis

```
Discovery Process:
1. Find all documentation files (*.md, docs/, inline comments)
2. Map code changes to related documentation
3. Extract references (functions, imports, configs, examples)
4. Cross-reference against actual changes

Outdated Content Detection:
- Broken function/API references → HIGH PRIORITY
- Incorrect code examples → HIGH PRIORITY  
- Wrong file paths → HIGH PRIORITY
- Outdated version numbers → MEDIUM PRIORITY
- Missing new features → MEDIUM PRIORITY
- Minor improvements → LOW PRIORITY

Intelligent Mapping:
- Code file changes → Related documentation files
- Function name changes → Documentation references
- Configuration changes → Config documentation
- CLI changes → Command documentation
```

**Step 4:** Comprehensive Decision Point
Present complete analysis with 4 scenarios:

- A) Version bump + Documentation outdated → Full update recommended
- B) Version bump + Documentation current → Version-only update
- C) No version bump + Documentation outdated → Documentation-only update  
- D) No version bump + Documentation current → Direct commit

**Step 5:** Execute User Choice

- **Fix All**: Version bump + documentation updates + commit + push
- **Version Only**: Version bump + commit + push (docs later)
- **Docs Only**: Documentation updates + commit + push (version later)
- **Commit As-Is**: Direct commit + push (accept current state)

**Step 6:** Smart Documentation Updates (when selected)

- Automatically fix high-priority issues (broken references, examples)
- Update version numbers and file paths
- Suggest medium/low priority improvements
- Validate all code examples work with current implementation

**Step 7:** Intelligent Commit Message

- Combine all changes into meaningful commit message
- Include version bump details if applicable
- Summarize documentation updates if performed
- Use conventional commit format when appropriate

**Step 8:** Completion and Reporting

- Commit and push changes
- Provide comprehensive summary of all updates
- Highlight remaining recommendations for future commits

This enhanced approach ensures every commit maintains comprehensive consistency between code, version, and documentation - creating a more maintainable and professional codebase. InshaAllah, this will significantly improve the quality and completeness of all commits.

