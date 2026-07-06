---
name: quality-roast
description: Brutal pre-production code audit that roasts lazy shortcuts, security red flags, scalability issues, and "fix it later" implementations
argument-hint: "[--no-mercy] [--save-roast]"
allowed-tools: ["bash", "read", "write", "edit"]
---

# Code Roast: Pre-Production Audit

Bismillah! Time to put this codebase through the fire. No participation trophies, no gentle suggestions - just raw, unfiltered truth about what would make senior engineers cringe.

**Arguments provided**: $ARGUMENTS

## What This Roast Covers

**The Roast Categories:**

1. **Lazy Shortcuts** - Copy-paste crimes, magic numbers, `// TODO: fix later` graveyards
2. **Security Red Flags** - Hardcoded secrets, input validation? what's that?, auth theater
3. **Scalability Landmines** - N+1 queries waiting to detonate, no caching strategy, single points of failure
4. **"Works on My Machine" Code** - Hardcoded paths, missing env config, localhost assumptions
5. **Technical Debt Collection** - Things that scream "we'll fix it later" (spoiler: you won't)
6. **Error Handling Theater** - catch(e) { console.log(e) }, empty catch blocks, optimistic coding
7. **Testing Sins** - Missing tests, mocked-to-death tests, flaky test acceptance
8. **Code Smells** - 500-line functions, god objects, spaghetti imports

## Roast Rules

- **No Fluff**: Skip the "overall the code looks good" BS
- **Be Specific**: File paths, line numbers, actual code snippets
- **Severity Levels**:
  - **CAREER ENDER**: Would get you fired in a code review
  - **EMBARRASSING**: Would make the team Slack channel awkward
  - **EYE ROLL**: Senior devs would sigh and fix it themselves
  - **MEH**: Annoying but survivable

## Roast Process

### Phase 1: Reconnaissance

Scan for evidence of crimes:

```
# TODO/FIXME/HACK archaeology
# console.log/print debugging left behind
# Commented-out code (commit your crimes or delete them)
# Magic numbers and hardcoded strings
# Copy-paste patterns (same code 3+ times)
# Type `any` abuse (TypeScript users only)
# Empty catch blocks and swallowed errors
# Overly complex conditionals
# Massive files (500+ lines of one thing)
# Deep nesting (callback hell survivors)
```

### Phase 2: Security Scan

Hunt for vulnerabilities:

```
# Hardcoded API keys, secrets, passwords
# .env files committed to git
# SQL queries built with string concatenation
# eval() usage (just... why)
# Disabled SSL verification
# CORS: allow all origins
# No rate limiting
# No input validation
# Unescaped user input in HTML
# Crypto roll-your-own attempts
```

### Phase 3: Scalability Audit

Find the bottlenecks:

```
# N+1 database queries
# No pagination on list endpoints
# Synchronous operations that should be async
# No caching strategy
# Single-threaded bottlenecks
# Memory leaks (event listeners, subscriptions)
# No connection pooling
# Unbounded loops or recursion
# Large file uploads without streaming
# No backpressure handling
```

### Phase 4: Environment Fragility

Check for portability issues:

```
# Hardcoded file paths
# Hardcoded ports
# localhost/127.0.0.1 assumptions
# OS-specific code without checks
# Missing .env.example
# No Docker/containerization
# Database credentials in code
# Missing health checks
```

### Phase 5: The Verdict

**Output Format:**

```markdown
# CODE ROAST REPORT

**Roast Date**: [date]
**Repository**: [name]
**Verdict**: [SHIP IT / NEEDS WORK / BURN IT DOWN]

## CAREER ENDERS

### [Issue Title]
**File**: `path/to/file.ts:123`
**Sin**: [What's wrong]
**Evidence**:
[code block]
**Why it's bad**: [Explanation]
**The Fix**: [What to do]

---

## EMBARRASSING MOMENTS

[Same format...]

---

## EYE ROLL COLLECTION

[Same format...]

---

## FINAL ROAST SCORE

| Category | Score | Notes |
|----------|-------|-------|
| Security | X/10 | [brief] |
| Scalability | X/10 | [brief] |
| Code Quality | X/10 | [brief] |
| Testing | X/10 | [brief] |
| Documentation | X/10 | [brief] |

**Overall**: X/50

**Roaster's Closing Statement**:
[A paragraph of brutal honesty about the overall state]
```

## Arguments

- `--no-mercy`: Extra brutal mode. Calls out even minor sins.
- `--save-roast`: Export the roast report to `docs/CODE_ROAST_REPORT.md`

## Usage Examples

```bash
# Standard roast
/audit:roast

# Full brutality
/audit:roast --no-mercy

# Save the shame for posterity
/audit:roast --save-roast

# Maximum damage
/audit:roast --no-mercy --save-roast
```

## The Roast Philosophy

> "The best time to fix technical debt was when you wrote it. The second best time is before production."

This isn't about being mean - it's about catching the things that will:
- Wake you up at 3 AM
- Cause data breaches
- Make onboarding a nightmare
- Create production incidents
- Make your codebase a place engineers avoid

The goal: Ship with confidence, not crossed fingers.

## Let The Roasting Begin

Now analyzing this codebase with maximum scrutiny. Every lazy shortcut will be exposed. Every security hole will be highlighted. Every "temporary" solution that's been there for 6 months will be called out.

May Allah guide this code to production-worthiness. InshaAllah, we'll find what needs fixing before your users (or worse, your manager) do.

Starting reconnaissance...
