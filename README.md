# Code-Review-MasterSkill for Claude Code

A comprehensive, multi-lens code review skill for [Claude Code](https://claude.ai/code) that covers security, performance, architecture, style, and test quality — with confidence scoring to eliminate noise.

## Stack Coverage

| Language / Framework | What's covered |
|---|---|
| TypeScript / Next.js | App Router, API routes, SSR/SSG, React patterns |
| Java / Spring Boot | REST APIs, services, repositories, transactions |
| JavaScript | Node.js, async patterns, modern ES syntax |
| Python | Scripts, APIs, async, type hints |
| SQL / PostgreSQL | Queries, schema, migrations, indexes |

## Review Lenses

- **Security** — OWASP Top 10 (injection, auth, data exposure, CORS, …)
- **Performance** — N+1 queries, unbounded queries, blocking I/O, re-renders
- **Architecture** — SOLID principles, layer separation, coupling
- **Style** — Google Style Guides, naming, dead code, magic numbers
- **Tests** — Coverage gaps, over-mocking, flaky patterns
- **Git Context** — Blame, history, reverted logic, TODO drift
- **Stale Comments** — Outdated Javadoc/JSDoc, dead comment blocks
- **CLAUDE.md Compliance** — Project-specific guidelines (confidence ≥ 90)

## Confidence Scoring

Every issue is scored 0–100. Only issues scoring **≥ 80** are surfaced. This eliminates speculative findings and reduces noise to near zero.

## Output Formats

| Format | Description |
|---|---|
| `report` | Severity-grouped report with before/after code snippets (default) |
| `inline` | Findings grouped by file/function location |
| `summary` | Short paragraph + top-N issues list |

## Installation

### Via SkillsMP (recommended)
Search for `Ksawyoux/Code-Review-MasterSkill` on [SkillsMP](https://skillsmp.com).

### Manual
```bash
# Clone into your project's .claude/skills directory
git clone https://github.com/Ksawyoux/Code-Review-MasterSkill \
  .claude/skills/code-review
```

## Usage

```
/code-review
```

**Review a PR:**
```
/code-review 123
/code-review https://github.com/owner/repo/pull/123
```

**Target specific lenses:**
```
/code-review security performance
/code-review tests errors
```

**Parallel mode (faster for large PRs):**
```
/code-review all parallel
```

## Requirements

- [Claude Code](https://claude.ai/code)
- [`gh` CLI](https://cli.github.com/) — required for PR workflow features

## Author

[Ksawyoux](https://github.com/Ksawyoux)
)
ss-aboukad)
)
