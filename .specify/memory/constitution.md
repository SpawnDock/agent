# SpawnDock Constitution

## Core Principles

### I. Code Quality (NON-NEGOTIABLE)

Every line of code must be purposeful and maintainable:

- **Clarity over cleverness**: Code is read far more than it is written; optimize for the next reader
- **Single Responsibility**: Each module, function, and class has one well-defined reason to change
- **No dead code**: Unused exports, commented-out blocks, and unreachable branches are prohibited
- **Consistent naming**: Follow language-idiomatic conventions (camelCase for TS/JS vars, PascalCase for types/components, SCREAMING_SNAKE for constants)
- **Zero linter warnings**: ESLint/TypeScript strict mode must pass with zero warnings before merge; `@ts-ignore` and `eslint-disable` require a written justification comment
- **Dependency hygiene**: Prefer stdlib and existing dependencies; every new dependency requires explicit justification in the PR description

### II. Testing Standards (NON-NEGOTIABLE)

Tests are not optional — they are part of the definition of done:

- **TDD cycle enforced**: Write a failing test → get approval → implement → go green → refactor
- **Coverage floors**: Unit test coverage must not drop below **80%** for any module; critical paths (auth, payments, data mutations) require **100%** branch coverage
- **Test pyramid**: Unit tests are the base; integration tests verify service contracts; E2E tests cover critical user journeys only
- **Deterministic tests**: No flaky tests allowed; tests depending on time, randomness, or network must use explicit mocks/fakes
- **Test naming**: Follow `given_when_then` or `should_<behaviour>_when_<condition>` patterns; test names are documentation
- **CI gate**: All tests must pass in CI before any merge to `main`; no bypassing test gates

### III. User Experience Consistency

Every interface — UI or API — must feel like one product:

- **Design tokens first**: Colors, spacing, typography, and motion must come from the shared design token system; no hardcoded values
- **Accessible by default**: Components must meet WCAG 2.1 AA; interactive elements require keyboard navigation and ARIA labels
- **Error states are first-class**: Every user-facing operation must have explicit loading, error, and empty states — not afterthoughts
- **API contract stability**: Public API responses follow consistent envelope structure (`{ data, error, meta }`); breaking changes require a major version bump and migration guide
- **Feedback latency**: User-initiated actions must provide visual feedback within **100 ms**; perceived responsiveness trumps raw speed

### IV. Performance Requirements

Performance is a feature, not a phase:

- **Frontend budgets**: Initial bundle ≤ **150 KB** gzipped JS; Largest Contentful Paint ≤ **2.5 s** on a mid-range device on 4G; Cumulative Layout Shift < **0.1**
- **API response targets**: p50 ≤ **100 ms**, p95 ≤ **500 ms**, p99 ≤ **1 s** for all read endpoints under nominal load
- **Database queries**: No N+1 queries; all queries touching more than one table require an EXPLAIN ANALYZE review before merge
- **Resource efficiency**: Background jobs and scheduled tasks must declare expected CPU/memory bounds; unbounded loops are prohibited
- **Measurement before optimization**: All performance work begins with a profiling report; changes must include before/after benchmarks

## Quality Gates

Every pull request must satisfy all of the following before merge:

| Gate              | Requirement                                     |
|-------------------|-------------------------------------------------|
| Linting           | Zero ESLint warnings, zero TypeScript errors    |
| Unit tests        | All pass; coverage does not regress             |
| Integration tests | All service-contract tests pass                 |
| Bundle size       | No regression beyond +5 KB gzipped              |
| Accessibility     | No new axe-core violations                      |
| Code review       | At least one approval from a project maintainer |

## Development Workflow

1. **Branch from `main`** using `feat/<short-slug>`, `fix/<short-slug>`, or `chore/<short-slug>`
2. **Write tests first** for any functional change
3. **Open a draft PR early** to share intent and get early feedback
4. **Self-review the diff** before requesting review — no WIP code in review-ready PRs
5. **Squash merge to `main`** with a conventional commit message (`feat:`, `fix:`, `chore:`, `perf:`, `test:`, `docs:`)

## Governance

This Constitution supersedes all other documented practices. Amendments require:

1. A written proposal explaining the motivation
2. Discussion period of at least two working days
3. Agreement from all active maintainers
4. Update to this document with the new version and amendment date

All code reviews must verify compliance with these principles. Complexity that contradicts any principle must be justified with documented rationale in the PR description.

**Version**: 1.0.0 | **Ratified**: 2026-03-21 | **Last Amended**: 2026-03-21
