---
name: git-tools-get-pr-review
description: Retrieve and analyze PR/MR review feedback to improve code quality
argument-hint: "[PR_NUMBER_OR_URL] [--github|--gitlab]"
---

Bismillah! I'll retrieve and analyze the PR/MR review feedback to help you improve your code based on the reviewer's suggestions.

Arguments provided: $ARGUMENTS

## Platform Selection

**Flags:**
- `--github` - Fetch PR review from GitHub using `gh` CLI (default)
- `--gitlab` - Fetch MR review from GitLab using `glab` CLI

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
    REVIEW_TYPE="merge request"
elif [[ "$ARGUMENTS" == *"--github"* ]]; then
    PLATFORM="github"
    CLI_TOOL="gh"
    REVIEW_TYPE="pull request"
elif [[ "$ARGUMENTS" == *"gitlab.com"* ]]; then
    PLATFORM="gitlab"
    CLI_TOOL="glab"
    REVIEW_TYPE="merge request"
elif [[ "$ARGUMENTS" == *"github.com"* ]]; then
    PLATFORM="github"
    CLI_TOOL="gh"
    REVIEW_TYPE="pull request"
else
    # Default to GitHub for plain numbers
    PLATFORM="github"
    CLI_TOOL="gh"
    REVIEW_TYPE="pull request"
fi

echo "📌 Platform: $PLATFORM ($REVIEW_TYPE review)"
```

## Using with Thinking Mode: **mode: think hard**

When executing this command with **mode: think hard**, I will perform an exceptionally deep and comprehensive analysis:

### **Advanced Review Analysis**

- **Deep Context Understanding**: Analyze the entire codebase context, not just the PR changes
- **Cross-Reference Analysis**: Connect reviewer feedback to broader architectural patterns and project conventions
- **Hidden Issue Detection**: Identify potential issues that reviewers might have missed
- **Performance Impact Assessment**: Evaluate performance implications of suggested changes
- **Security Review**: Conduct security analysis beyond basic review comments
- **Maintainability Evaluation**: Assess long-term code maintainability implications

### **Reviewer Psychology & Communication**

- **Intent Interpretation**: Deeply analyze the underlying intent behind each reviewer comment
- **Communication Style Analysis**: Understand reviewer preferences and communication patterns
- **Priority Inference**: Determine true urgency levels beyond surface-level feedback
- **Relationship Dynamics**: Consider team dynamics and reviewer expertise areas

### **Strategic Implementation Planning**

- **Multi-Level Solution Analysis**: Explore multiple approaches for each piece of feedback
- **Dependency Mapping**: Create detailed dependency graphs for implementation order
- **Risk Assessment Matrix**: Evaluate risks and benefits of each potential change
- **Future-Proofing**: Consider how changes will affect future development and scalability

### **Advanced Code Quality Enhancement**

- **Pattern Recognition**: Identify recurring patterns in feedback for systematic improvements
- **Best Practice Synthesis**: Combine reviewer suggestions with industry best practices
- **Optimization Opportunities**: Find performance and efficiency improvements beyond the review scope
- **Technical Debt Analysis**: Evaluate and address technical debt highlighted by review feedback

This intensive analysis ensures not just addressing review feedback, but elevating the entire codebase quality and team collaboration effectiveness.

Let me:

1. **Identify and fetch the PR/MR review**
   - Parse PR/MR number or URL from arguments or detect current branch's PR/MR
   - Use appropriate CLI based on platform:
     - GitHub: `gh pr view <number> --json reviews,comments`
     - GitLab: `glab mr view <number>` and `glab api` for discussions
   - Retrieve all review threads, suggestions, and approval status

2. **Analyze the review feedback comprehensively**
   - Identify Claude agent reviews vs. human reviews for specialized handling
   - Categorize feedback by type (bugs, improvements, style, architecture, security)
   - Distinguish between critical issues that must be addressed vs. suggestions
   - Extract specific code locations and recommended changes with line numbers
   - Assess overall review sentiment and approval likelihood
   - Parse Claude agent's structured feedback format and recommendations

3. **Create actionable improvement plan**
   - Prioritize feedback items by impact and effort required
   - Generate specific code changes needed for each comment
   - Suggest commit organization for addressing feedback
   - Identify any breaking changes or architectural considerations

4. **Provide clear recommendations**
   - If review is positive: Highlight what's working well, celebrate good practices, note minor improvements
   - If Claude agent found the code satisfactory: Summarize positive aspects and readiness for merge
   - If review needs work: Create step-by-step action items with specific code examples
   - For Claude agent feedback: Interpret AI suggestions in context of best practices
   - Suggest response strategy for reviewer comments and how to address concerns
   - Estimate time and complexity for implementing each change
   - Recommend whether to request re-review after changes

5. **Generate implementation roadmap**
   - Break down changes into logical, atomic commits with clear messages
   - Suggest order of implementation to minimize conflicts and build failures
   - Identify dependencies between different feedback items
   - Provide templates for updating PR description with changes made
   - Include testing strategy for validating fixes
   - Suggest documentation updates if needed

6. **Execute and verify improvements**
   - Apply high-priority fixes immediately where possible
   - Run existing tests to ensure no regressions
   - Validate that feedback has been properly addressed
   - Prepare summary of changes made for reviewer response

7. **Commit and push improvements**
   - Stage all implemented changes with `git add .`
   - Create descriptive commit message summarizing the addressed feedback
   - Use format: "address PR feedback: [specific changes made]"
   - Push changes to remote branch to update the PR
   - Ensure all improvements are visible to reviewers before requesting re-review

8. **Conditional re-review request (SMART EXECUTION)**

   **IF changes were made:**
   - MANDATORY: Post re-review request comment using appropriate CLI
   - Use exact format: "@claude I've addressed the review feedback. Please re-review the changes:"
   - Include specific changes made with file:line references
   - List all modified files
   - End with "**Ready for re-review** ✅"
   - Verify comment was posted successfully

   **IF NO changes were made:**
   - Report to user only - NO PR/MR comment needed
   - Provide summary: "Analysis complete: No improvements required"
   - Explain why no changes were needed (already implemented, meets standards, etc.)
   - Confirm PR/MR is ready for merge
   - Note: "No re-review request posted (no changes made)"

   **Implementation Logic:**
   ```bash
   # Track if any changes were made during the process
   CHANGES_MADE=false

   # After each improvement attempt, check for actual changes
   if ! git diff --quiet HEAD~1 HEAD; then
       CHANGES_MADE=true
   fi

   # Final decision - only ping @claude if changes exist
   if [ "$CHANGES_MADE" = true ]; then
       if [ "$PLATFORM" = "github" ]; then
           gh pr comment $PR_NUMBER --body "@claude I've addressed the review feedback..."
       else
           glab mr note $MR_NUMBER --message "@claude I've addressed the review feedback..."
       fi
   else
       echo "✅ Analysis complete: No improvements required - reporting to user only"
   fi
   ```

9. **Provide suggested improvements**
   - Based on deep analysis, present additional code quality improvements beyond the review feedback
   - Suggest architectural enhancements, performance optimizations, or maintainability improvements
   - Recommend best practices that could elevate the code further
   - Offer insights on patterns, testing strategies, or documentation that could benefit the codebase

## Usage Examples

```bash
# Get GitHub PR review (default)
/git:get-pr-review 42

# Get GitHub PR review (explicit)
/git:get-pr-review 123 --github

# Get GitLab MR review
/git:get-pr-review 45 --gitlab

# Auto-detect from GitHub URL
/git:get-pr-review https://github.com/owner/repo/pull/15

# Auto-detect from GitLab URL
/git:get-pr-review https://gitlab.com/owner/repo/-/merge_requests/45
```

Alhamdulillah, let me fetch and analyze the review feedback to help you create an excellent PR/MR!

