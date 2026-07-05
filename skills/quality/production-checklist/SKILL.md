---
name: quality-production-checklist
description: Analyze any codebase for production readiness with comprehensive security, performance, and deployment checklist
argument-hint: "[--full-audit] [--export-report]"
allowed-tools: ["bash", "read", "write", "edit"]
---

# Production Readiness Checker (Universal)

Bismillah! This command performs a comprehensive, tech-stack-agnostic analysis of any repository to assess production readiness. It's safe to run anywhere - purely read-only analysis with optional report export.

## What This Command Does

Analyzes your codebase across 10 critical production dimensions:

1. **Security Audit**: Secrets, vulnerabilities, dependencies, authentication
2. **Environment Configuration**: ENV vars, config management, secrets handling
3. **Error Handling & Logging**: Exception handling, monitoring, observability
4. **Performance & Optimization**: Bundle size, caching, database queries, CDN
5. **Testing & Quality**: Test coverage, CI/CD, linting, type safety
6. **Infrastructure & Deployment**: Docker, orchestration, scaling, rollback
7. **Database & Data**: Migrations, backups, connection pooling, indexes
8. **Monitoring & Observability**: APM, alerts, dashboards, SLAs
9. **Documentation**: README, API docs, runbooks, architecture diagrams
10. **Legal & Compliance**: Licenses, GDPR, ToS, privacy policy

## Arguments

- `--full-audit`: Deep dive into each category with code examples (default: quick scan)
- `--export-report`: Generate markdown report in `docs/production-readiness-report.md`

## Implementation Process

### Phase 1: Repository Discovery & Analysis

1. **Detect Tech Stack**
   - Scan for package.json, requirements.txt, go.mod, Cargo.toml, pom.xml, etc.
   - Identify frameworks (Next.js, React, Django, Rails, Spring Boot, etc.)
   - Detect infrastructure files (Dockerfile, docker-compose.yml, k8s/, terraform/)
   - Map build tools (webpack, vite, rollup, esbuild, etc.)

2. **Analyze Project Structure**
   - Identify source code directories
   - Find test directories and files
   - Locate configuration files
   - Detect documentation

### Phase 2: Production Readiness Checks

#### 1. Security Audit ✅

**What I Check:**
- Hardcoded secrets (API keys, passwords, tokens) in code
- `.env` files tracked in git (should be in .gitignore)
- Dependency vulnerabilities (npm audit, pip check, cargo audit)
- Authentication/authorization implementation
- HTTPS/SSL enforcement
- Input validation and sanitization
- CORS configuration
- Rate limiting implementation
- SQL injection vulnerabilities
- XSS protection

**Tech-Specific:**
- **Node.js**: Check for `npm audit`, helmet.js, bcrypt/argon2
- **Python**: bandit, safety check, secrets detection
- **Go**: gosec, vulnerability scanning
- **Rust**: cargo-audit, dependency review
- **Java**: OWASP dependency check, SpotBugs

#### 2. Environment Configuration ⚙️

**What I Check:**
- Environment variable management (.env, .env.example)
- Configuration for dev/staging/production
- Secret management (Vault, AWS Secrets Manager, etc.)
- Feature flags infrastructure
- API endpoint configuration
- Database connection strings (should be env vars)
- Third-party service keys (Stripe, SendGrid, etc.)

**Red Flags:**
- Production credentials in code
- Missing .env.example template
- No distinction between dev/prod configs

#### 3. Error Handling & Logging 🔍

**What I Check:**
- Global error handlers implemented
- Try-catch blocks in critical paths
- Error logging service (Sentry, Rollbar, LogRocket, etc.)
- Log levels properly configured (debug, info, warn, error)
- Request ID tracing
- Error boundaries (React/frontend)
- Graceful degradation
- User-friendly error messages (no stack traces to users)

**Tech-Specific:**
- **Node.js**: Express error middleware, uncaughtException handler
- **Python**: logging module configured, exception handling
- **Frontend**: Error boundaries, window.onerror, unhandledrejection

#### 4. Performance & Optimization ⚡

**What I Check:**
- Bundle size analysis (webpack-bundle-analyzer, etc.)
- Code splitting / lazy loading
- Image optimization (compression, WebP, responsive)
- CDN configuration
- Caching strategy (Redis, in-memory, HTTP caching)
- Database query optimization (N+1 queries, indexes)
- Asset minification and compression (gzip, brotli)
- Service worker / PWA capabilities
- Server-side rendering / static generation

**Metrics to Report:**
- JavaScript bundle sizes
- Page load time estimates
- Database query count in hot paths

#### 5. Testing & Quality 🧪

**What I Check:**
- Test suite exists (unit, integration, e2e)
- Test coverage reports
- CI/CD pipeline configured (.github/workflows, .gitlab-ci.yml, etc.)
- Linting rules enforced (ESLint, Pylint, Clippy, etc.)
- Type checking (TypeScript strict mode, MyPy, etc.)
- Pre-commit hooks (Husky, pre-commit, etc.)
- Automated testing on PRs
- Performance testing / load testing

**Coverage Targets:**
- Critical paths: 90%+
- Overall coverage: 70%+
- E2E tests for core user flows

#### 6. Infrastructure & Deployment 🚀

**What I Check:**
- Dockerfile present and optimized (multi-stage builds)
- docker-compose.yml for local development
- Kubernetes manifests / Helm charts
- CI/CD deployment pipeline
- Zero-downtime deployment strategy
- Rollback capabilities
- Health check endpoints (/health, /readiness)
- Graceful shutdown handling
- Auto-scaling configuration
- Load balancer setup

**Deployment Checklist:**
- [ ] Blue-green or canary deployment
- [ ] Database migration strategy
- [ ] Backup and restore procedures
- [ ] Disaster recovery plan

#### 7. Database & Data 💾

**What I Check:**
- Migration files present and versioned
- Database connection pooling
- Indexes on frequently queried columns
- Backup strategy documented
- Transaction handling
- Data validation at DB level
- Soft delete vs hard delete strategy
- PII data handling (encryption at rest)
- Database credentials from env vars
- Read replicas configured (if needed)

**Red Flags:**
- No migrations directory
- Missing indexes on foreign keys
- No backup documentation
- Hardcoded DB credentials

#### 8. Monitoring & Observability 📊

**What I Check:**
- APM tool integrated (New Relic, Datadog, AppDynamics, etc.)
- Uptime monitoring (Pingdom, UptimeRobot, etc.)
- Custom metrics/dashboards
- Alert rules configured
- Log aggregation (ELK, Splunk, CloudWatch, etc.)
- Distributed tracing (Jaeger, Zipkin, if microservices)
- SLA/SLO definitions
- On-call rotation setup

**Must-Have Alerts:**
- 5xx error rate spikes
- Response time degradation
- Disk/memory usage thresholds
- Database connection pool exhaustion

#### 9. Documentation 📚

**What I Check:**
- README.md with setup instructions
- Architecture documentation (diagrams, ADRs)
- API documentation (Swagger/OpenAPI, Postman, etc.)
- Deployment runbook
- Incident response playbook
- Contributing guidelines
- Code comments for complex logic
- Environment variable documentation

**Completeness Score:**
- README: Getting started, prerequisites, commands
- API docs: Endpoints, auth, examples
- Runbook: Deploy steps, rollback, common issues

#### 10. Legal & Compliance ⚖️

**What I Check:**
- LICENSE file present
- Open source dependency licenses compatible
- Privacy policy (if collecting user data)
- Terms of Service
- Cookie consent (GDPR)
- Data retention policies
- CCPA/GDPR compliance (if applicable)
- Accessibility (WCAG 2.1 AA)
- Copyright notices

### Phase 3: Scoring & Reporting

**Scoring System:**
- Each category scored 0-10
- Overall production readiness: 0-100
- **90-100**: Production Ready ✅
- **70-89**: Minor Improvements Needed ⚠️
- **50-69**: Significant Work Required ⚠️⚠️
- **<50**: Not Production Ready ❌

**Report Sections:**
1. **Executive Summary**: Overall score, critical blockers, timeline estimate
2. **Category Breakdown**: Score per dimension with specific findings
3. **Critical Issues**: Must-fix before production (security, data loss risks)
4. **High Priority**: Should-fix (performance, reliability)
5. **Medium Priority**: Nice-to-have (observability, documentation)
6. **Low Priority**: Polish items (comments, minor docs)
7. **Action Plan**: Prioritized checklist with effort estimates

### Phase 4: Generate Actionable TodoList

**Output Format:**
```markdown
## Production Readiness Report

**Overall Score**: 78/100 ⚠️ Minor Improvements Needed

### Critical Blockers (Must Fix) 🚨
- [ ] Remove hardcoded API keys from `src/config/api.ts:12`
- [ ] Add SSL certificate for production domain
- [ ] Fix SQL injection vulnerability in user search endpoint

### High Priority (Should Fix) ⚠️
- [ ] Implement error monitoring (Sentry)
- [ ] Add health check endpoint
- [ ] Configure database backups (daily)
- [ ] Set up CI/CD pipeline

### Medium Priority 📋
- [ ] Add E2E tests for checkout flow
- [ ] Document API endpoints (Swagger)
- [ ] Optimize bundle size (currently 2.1MB)

### Low Priority ✨
- [ ] Add code comments to complex algorithms
- [ ] Create architecture diagrams
```

## Tech Stack Detection Matrix

**Frontend:**
- React/Next.js/Vue/Angular: Check bundle size, lazy loading, SSR
- Mobile (React Native, Flutter): Check app size, performance profiling
- Static sites (Gatsby, Hugo): Check build optimization

**Backend:**
- Node.js (Express, Fastify, NestJS): Check clustering, error middleware
- Python (Django, Flask, FastAPI): Check WSGI/ASGI config, middleware
- Go: Check goroutine leaks, context cancellation
- Rust: Check unsafe blocks, panic handling
- Java/Spring: Check JVM tuning, connection pooling

**Database:**
- PostgreSQL/MySQL: Check indexes, query performance, replication
- MongoDB: Check indexes, sharding strategy
- Redis: Check persistence, memory limits

**Infrastructure:**
- Docker: Check multi-stage builds, security scanning
- Kubernetes: Check resource limits, HPA, readiness probes
- Serverless (AWS Lambda, Vercel): Check cold start, timeout config

## Smart Analysis Features

1. **Context-Aware Checks**: Different criteria for different project types
   - SaaS app: Emphasis on security, uptime, scalability
   - Internal tool: Emphasis on documentation, error handling
   - API service: Emphasis on rate limiting, versioning, docs

2. **Auto-Detect Critical Paths**: Identify most important user flows to prioritize

3. **Comparison Mode**: Compare against industry standards for similar projects

4. **Incremental Fixes**: Suggest what to fix first based on risk vs. effort

## Safety Guarantees

- ✅ **Read-only**: Never modifies code
- ✅ **Non-invasive**: No installations or npm installs
- ✅ **Universal**: Works on any tech stack
- ✅ **Fast**: Scans complete in <2 minutes for most repos
- ✅ **Offline-capable**: Core checks don't require internet

## Example Usage

```bash
# Quick scan (default)
/production-checklist

# Deep audit with detailed findings
/production-checklist --full-audit

# Generate exportable report
/production-checklist --export-report

# Both deep audit + export
/production-checklist --full-audit --export-report
```

## Output Example

```
🔍 Production Readiness Analysis
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📦 Detected: Next.js 15 + TypeScript + Supabase + Solana
🏗️  Infrastructure: Docker, Vercel-ready
📊 Overall Score: 82/100 ⚠️ Minor Improvements Needed

Category Scores:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Security             ████████░░ 8/10
Environment Config   ██████████ 10/10
Error Handling       ███████░░░ 7/10
Performance          █████████░ 9/10
Testing & Quality    ████████░░ 8/10
Infrastructure       ████████░░ 8/10
Database & Data      ██████░░░░ 6/10
Monitoring           █████░░░░░ 5/10
Documentation        ████████░░ 8/10
Legal & Compliance   ███████░░░ 7/10

🚨 Critical Issues (3):
  1. [SECURITY] Hardcoded API key in src/config/api.ts:23
  2. [DATA LOSS] No database backup strategy documented
  3. [SECURITY] CORS allows all origins in production

⚠️  High Priority (7):
  1. Add error monitoring (Sentry/LogRocket)
  2. Configure health check endpoint
  3. Set up uptime monitoring
  4. Add E2E tests for critical flows
  5. Document rollback procedure
  6. Configure rate limiting
  7. Add database indexes on foreign keys

📋 Action Plan:
  Estimated time to production ready: 2-3 days

  Day 1 (Critical):
    - Fix hardcoded secrets
    - Configure CORS properly
    - Set up database backups

  Day 2 (High Priority):
    - Add monitoring & alerts
    - Implement health checks
    - Add E2E tests

  Day 3 (Polish):
    - Complete documentation
    - Final security review
    - Load testing

✅ Production Ready When:
   - All critical issues resolved
   - Score reaches 85+
   - Manual QA passed
   - Load tested at 2x expected traffic
```

Now analyzing your repository with maximum thoroughness...

InshaAllah, this will provide a comprehensive, actionable roadmap to production! 🚀

## Analysis Starting...
