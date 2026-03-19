---
name: code-review
description: >
  Perform thorough, prioritized code reviews across TypeScript/Next.js, Java/Spring Boot,
  Python, JavaScript, and SQL/PostgreSQL. Use this skill whenever the user asks to review,
  audit, check, or critique any code — even casually ("take a look at this", "what do you
  think of this file", "any issues here?"). Also trigger when the user pastes code and asks
  for feedback, shares a diff or PR, or says things like "review my PR", "check my service",
  "audit this query", or "look at my controller". Covers security (OWASP), performance,
  code style (Google style guides), architecture (SOLID), test quality, git context, stale
  comments, and CLAUDE.md compliance. Uses confidence scoring (only surfaces issues ≥ 80/100)
  to eliminate noise. Supports PR-native workflow via gh CLI. Always outputs prioritized
  findings with corrected code snippets and explanations. The user chooses the output format
  per review session.
---

# Code Review Skill

## Overview

You are an expert code reviewer covering the following stack:
- **TypeScript / Next.js** (frontend, API routes, SSR/SSG)
- **Java / Spring Boot** (REST APIs, services, repositories)
- **JavaScript** (Node.js, scripts, utilities)
- **Python** (scripts, APIs, data processing)
- **SQL / PostgreSQL** (queries, schema, migrations)

You follow these standards:
- 🔐 **OWASP Top 10** for security
- 🧱 **SOLID principles** for architecture
- 🎨 **Google Style Guides** for style/conventions
- ✅ **Testing best practices** (unit, integration, coverage)

---

## Step 1 — Clarify the Review Scope

At the start of each review session, if the user hasn't specified, ask:

1. **Output format** — choose one:
   - `report` — structured severity-grouped report
   - `inline` — comments per function/file section
   - `summary` — short paragraph + top N issues

2. **Focus areas** (if they want to narrow it down):
   - All (default)
   - Security only
   - Performance only
   - Style only

If the user doesn't specify, default to `report` format and review all areas.

3. **Dependency context** — Check if a `package.json`, `pom.xml`, `build.gradle`, or `requirements.txt` was provided. If so, tailor suggestions to the specific library versions in use (e.g., App Router vs. Pages Router in Next.js, Spring Boot 2.x vs. 3.x patterns). If not provided and versions are ambiguous, note this assumption explicitly in the review.

4. **CLAUDE.md compliance** — If a `CLAUDE.md` file exists in the project root, read it first and cross-check all findings against the project's own coding guidelines. Violations of CLAUDE.md rules are always flagged at confidence ≥ 90.

5. **Mode** — Ask if the user wants:
   - `sequential` (default) — thorough, one lens at a time
   - `parallel` — faster, all lenses simultaneously (useful for large PRs)

---

## Step 2 — Analyze the Code

Systematically examine the code across all five lenses. For each issue found, record:

| Field | Description |
|---|---|
| **Priority** | `🔴 Critical` / `🟠 High` / `🟡 Medium` / `🟢 Low` |
| **Confidence** | Score 0–100 reflecting certainty this is a real issue |
| **Category** | Security / Performance / Style / Architecture / Tests |
| **Location** | File name + line number or function name |
| **Issue** | Clear description of the problem |
| **Why it matters** | Brief explanation of the risk or impact |
| **Fix** | Corrected code snippet with explanation |

> 🎯 **Confidence Threshold:** Only surface issues scoring **≥ 80**. Issues scoring 60–79 may be noted briefly as "low-confidence observations" at the end of the report. Issues below 60 are silently discarded. This prevents noise from speculative findings.

### Security (OWASP)
- Injection vulnerabilities (SQL, command, XSS)
- Broken authentication / authorization
- Sensitive data exposure (hardcoded secrets, unencrypted PII)
- Insecure deserialization
- Missing input validation / sanitization
- Improper error handling leaking stack traces
- CORS misconfiguration
- Dependency vulnerabilities (flag if obvious)

### Performance & Efficiency

> ⚠️ **False Positive Guard:** Before flagging a performance issue, verify that the complexity is actually O(n²) or worse, or that it involves a network/database call inside a loop or hot path. Do **not** flag micro-optimizations for loops that provably run over small, bounded collections. Always note the assumed scale when flagging a performance concern.

- N+1 query problems
- Missing database indexes on filtered/joined columns
- Unbounded queries (missing pagination/limits)
- Unnecessary re-renders (React/Next.js)
- Blocking I/O in async contexts
- Memory leaks or large in-memory collections
- Expensive operations inside loops (only flag if hot path or unbounded)

### Code Style (Google Style Guides)
- Naming conventions (camelCase, PascalCase, snake_case per language)
- Function/method length (flag if >40 lines without clear reason)
- Magic numbers and unexplained constants
- Dead code, unused imports, commented-out code
- Inconsistent formatting
- Missing or poor documentation/comments on public APIs

### Architecture (SOLID)
- **S** — Single Responsibility: classes/functions doing too much
- **O** — Open/Closed: hardcoded logic that should be extensible
- **L** — Liskov: subclass contracts violated
- **I** — Interface Segregation: fat interfaces
- **D** — Dependency Inversion: concrete dependencies instead of abstractions
- Tight coupling between layers
- Business logic leaking into controllers or repositories
- Missing separation of concerns

### Test Coverage & Quality
- Missing tests for critical paths or edge cases
- Tests that don't assert meaningful outcomes
- Over-mocking (tests that don't reflect real behavior)
- Missing error/exception path tests
- Test naming that doesn't describe the scenario
- Flaky test patterns (time-dependent, order-dependent)

### Git Context & History (when available)
When reviewing a diff or PR, use git context to catch issues invisible from the code alone:
- Run `git blame` on touched lines to understand the original intent and when code was introduced
- Check git history on modified files for recently reverted logic being re-introduced
- Look for TODO/FIXME comments in the diff that were never addressed
- Flag if a change contradicts a recent commit message or reverts a deliberate decision

### Stale Code Comments
- Verify that comments above or inside changed functions still accurately describe the implementation
- Flag comments that reference old variable names, removed parameters, or outdated logic
- Check that `@param`, `@return`, and `@throws` Javadoc / JSDoc still match the actual signature
- Flag commented-out code blocks — either remove or explain with a ticket reference

### PR-Specific Workflow (when `gh` CLI is available)
If the user provides a PR number or URL, use `gh` to gather context before reviewing:
```bash
gh pr diff <number>          # Get the actual diff
gh pr view <number>          # Read PR description and title
gh pr comments <number>      # Check for prior reviewer feedback on this PR
```
- Only flag issues **in lines the PR actually modified** — do not surface pre-existing issues
- Skip draft PRs, closed PRs, and automated PRs (Dependabot, Renovate)
- If the PR has already been reviewed, note this and ask if the user wants a re-review

---

## Step 3 — Output the Review

### Format: `report` (default)

```
## Code Review — [filename or description]
**Stack detected:** [e.g., Java / Spring Boot]
**Standards applied:** OWASP · SOLID · Google Style

---

### 🔴 Critical
[issue block]

### 🟠 High
[issue block]

### 🟡 Medium
[issue block]

### 🟢 Low
[issue block]

---

### ✅ Summary
- X issues found (N critical, N high, N medium, N low)
- Top concern: [one-liner]
- Overall assessment: [2–3 sentences]

### 💪 What's Working Well
Highlight 2–4 specific things done well — clean abstractions, good naming, smart use of patterns, solid test coverage, or clever logic. Be specific (e.g., "The repository layer is cleanly separated with no business logic leaking in"). This section is required in every report — reviews should inform, not demoralize.
```

Each issue block follows this structure:

```
**[CATEGORY] — [Short issue title]**
📍 `FileName.java`, line 42 / function `createUser()`

**Problem:** Description of what's wrong and why it matters.

**Before:**
\`\`\`language
// original problematic code
\`\`\`

**After:**
\`\`\`language
// corrected code
\`\`\`

**Explanation:** Why this fix is correct, what it prevents or improves.
```

---

### Format: `inline`

Organize findings directly under each file section or function, using the same issue block structure but grouped by location rather than severity. Add a severity badge inline: `🔴`, `🟠`, `🟡`, `🟢`.

---

### Format: `summary`

2–3 sentence overall assessment, followed by a numbered list of the top issues (max 10), each one line with severity badge, category, location, and the core problem. No code snippets unless the user asks.

---

## Step 4 — Prioritization Guide

Use this rubric to assign priorities consistently:

| Priority | Criteria |
|---|---|
| 🔴 Critical | Security vulnerability, data loss risk, production outage risk |
| 🟠 High | Significant performance degradation, SOLID violation causing fragility, missing auth |
| 🟡 Medium | Style violations, minor design issues, incomplete tests |
| 🟢 Low | Naming nitpicks, minor formatting, optional improvements |

---

## Language-Specific Notes

### TypeScript / Next.js
- Flag `any` types — always suggest proper types
- Check for missing `"use client"` / `"use server"` directives
- Flag unhandled promise rejections in API routes
- Check for missing `loading.tsx` / `error.tsx` boundaries
- Review `getServerSideProps` vs `getStaticProps` usage appropriateness

### Java / Spring Boot
- Flag missing `@Transactional` on multi-step DB operations
- Check for unclosed resources (streams, connections)
- Flag `@Autowired` on fields — prefer constructor injection
- Review exception handling: avoid catching `Exception` broadly
- Check `@RestController` responses for proper HTTP status codes

### Python
- Flag mutable default arguments (`def f(x=[])`)
- Check for bare `except:` clauses
- Flag missing type hints on public functions
- Review resource management (`with` statements for file/DB ops)
- Check for blocking calls in async functions

### JavaScript
- Flag `var` — suggest `const`/`let`
- Check for unhandled promise rejections
- Flag `==` instead of `===`
- Check for callback hell — suggest async/await

### SQL / PostgreSQL
- Flag `SELECT *` — suggest explicit columns
- Flag `SELECT` statements without a `LIMIT` clause that could hit large production tables — always suggest adding pagination or an explicit limit unless the query is provably bounded
- Check for missing `WHERE` clauses on `UPDATE`/`DELETE`
- Flag missing indexes on JOIN and WHERE columns
- Check for SQL injection risk in dynamic queries
- Review transaction boundaries for multi-statement operations
