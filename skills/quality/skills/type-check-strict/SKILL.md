---
name: quality-type-check-strict
description: Run TypeScript strict checks and create comprehensive plan to fix all errors
argument-hint: ""
allowed-tools: ["bash", "read", "write", "edit"]
---

# TypeScript Strict Type Checking & Error Resolution

Bismillah! This command runs `npm run type-check:strict` in thorough analysis mode, then creates a comprehensive plan to systematically fix all TypeScript errors to achieve zero errors.

## Implementation Process

Let me execute this with maximum thoroughness:

1. **Run Strict Type Check**
   - Execute `npm run type-check:strict` to get complete error analysis
   - Capture all TypeScript errors, warnings, and strict mode violations
   - Parse output to categorize error types and locations

2. **Deep Error Analysis**
   - Analyze each error for root cause and complexity
   - Group related errors by file, type, and dependency chain
   - Identify patterns and systematic issues
   - Assess impact and priority for each error category

3. **Comprehensive Fix Strategy Planning**
   - Outline a detailed plan with specific tasks for each error
   - Prioritize fixes by impact: breaking changes, type safety, performance
   - Plan incremental approach to avoid cascading issues
   - Include verification steps after each fix group

4. **Systematic Execution**
   - Fix errors in planned order using appropriate tools
   - Verify each fix doesn't introduce new errors
   - Run intermediate type checks to track progress
   - Update todo list with real-time progress

## Error Categories Handled

- **Type Annotation Issues**: Missing or incorrect type definitions
- **Strict Null/Undefined Checks**: Handling potential null/undefined values
- **No Implicit Any**: Adding explicit types where inferred as any
- **Strict Property Initialization**: Ensuring all properties are properly initialized
- **Unused Variables/Imports**: Cleaning up dead code
- **Type Compatibility**: Fixing type mismatches and assignments
- **Generic Constraints**: Properly constraining generic types
- **Index Signatures**: Handling dynamic property access safely

## Think Hard Mode Features

- **Multi-Pass Analysis**: Run multiple analysis passes to catch interdependencies
- **Impact Assessment**: Evaluate how each fix affects other parts of codebase
- **Regression Prevention**: Check for new errors introduced by fixes
- **Optimization Opportunities**: Identify type improvements beyond basic fixes
- **Documentation Updates**: Update type documentation as needed

## Progressive Fix Approach

1. **Foundation Fixes**: Core type definitions and interfaces first
2. **Propagation Fixes**: Address errors that cascade from foundation changes
3. **Edge Case Handling**: Tackle complex scenarios and edge cases
4. **Optimization Pass**: Improve type safety and performance
5. **Final Verification**: Comprehensive test to ensure zero errors

## Success Metrics

- ✅ Zero TypeScript errors in strict mode
- ✅ Maintained code functionality and performance
- ✅ Improved type safety and developer experience
- ✅ Clean, maintainable type definitions
- ✅ Comprehensive documentation of changes made

Now executing with maximum thoroughness and attention to detail...

## Step 1: Initial Type Check Analysis

Running `npm run type-check:strict` to establish baseline and identify all current issues:
