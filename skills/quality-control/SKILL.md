---
name: quality-control
description: Performs end-to-end quality control and verification audits across the entire project. Use when conducting a full project audit, validating codebase health, running comprehensive pre-release quality gates, or auditing code, tests, security, build, and runtime integrity before merging or deployment.
---

# Quality Control

## Overview

Execute systematic, end-to-end quality control (QC) audits across an entire project. Quality control verifies that every dimension of software health — syntax, type safety, test suites, dependencies, security vulnerabilities, asset integrity, and production build viability — satisfies strict production standards before release.

## When to Use

- Conducting a full pre-release or pre-merge quality audit across the entire repository.
- Onboarding, refactoring, or taking ownership of an existing codebase to baseline health.
- Running comprehensive automated quality gate evaluations before shipping (`/ship` or CI staging).
- Verifying regression resistance after major cross-cutting changes.

**When NOT to use:**
- Narrow single-file bugfixes (use `debugging-and-error-recovery` or `surgical-patch`).
- Standard pull request diff reviews (use `code-review-and-quality` or `caveman-review`).
- Isolated performance profiling (use `performance-optimization`).

## The End-to-End QC Pipeline

The quality control workflow executes across 6 sequential gates. A blocker failure at any gate halts the release verdict until remediated.

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│ 1. Discover │ ──▶ │ 2. Static &  │ ──▶ │ 3. Test     │
│  & Baseline │     │    Types     │     │    Suites   │
└─────────────┘     └──────────────┘     └─────────────┘
                                                │
┌─────────────┐     ┌──────────────┐            │
│ 6. Scorecard│ ◀── │ 5. Build &   │ ◀──────────┘
│  & Verdict  │     │    Integrity │
└─────────────┘     └──────────────┘
```

---

### Gate 1: Discovery & Baseline

Detect project ecosystem, package managers, tools, scripts, and runtime requirements.

1. **Inspect configuration manifests & test directories**:
   - Node.js (`package.json`, `pnpm-lock.yaml`, `package-lock.json`)
   - Python (`pyproject.toml`, `requirements.txt`, `poetry.lock`, `pytest.ini`)
   - PHP (`composer.json`, `phpunit.xml`)
   - Rust (`Cargo.toml`), Go (`go.mod`), Makefile, or Dockerfile.
   - Test directories: `tests/`, `test/`, `__tests__/`, `spec/`, `evals/`, `scripts/*test*.js`.
2. **Catalog available verification scripts**:
   - Lint commands (e.g., `npm run lint`, `ruff check .`, `phpstan analyse`)
   - Type check commands (e.g., `npx tsc --noEmit`, `mypy .`)
   - Test runners (e.g., `npm test`, `node --test`, `pytest`, `cargo test`, `composer test`, `go test ./...`, or standalone scripts like `node scripts/run-evals-test.js`)
   - Build scripts (e.g., `npm run build`, `cargo build --release`)
3. **Verify environment cleanliness**:
   - Check clean git status (`git status --porcelain`).
   - Ensure dependencies are properly installed and locked.

---

### Gate 2: Static Analysis & Type Safety

Ensure strict syntax validation, style conformity, and static typing integrity.

1. **Run linters and style checkers**:
   - Execute project linter with zero-warning or fail-on-error tolerance:
     ```bash
     npm run lint -- --max-warnings=0
     ```
2. **Execute type checking**:
   - Enforce complete type soundness without implicit any or broken contracts:
     ```bash
     npx tsc --noEmit
     ```
3. **Verify Dead Code & Unused Exports**:
   - Check for orphaned imports, unreferenced variables, or obsolete assets.

---

### Gate 3: Automated Test Verification

Validate functional correctness and regression resistance via the full test suite.

1. **Resolve and Execute Test Suites (Auto-Discovery)**:
   - **Configured Runner**: If manifest contains test scripts, execute directly:
     ```bash
     npm test -- --coverage # atau pytest, cargo test, composer test, go test ./...
     ```
   - **Native / Unpackaged Node**: If no `npm test` script is defined, check and run native runners or test files:
     ```bash
     # Built-in Node test runner
     node --test
     # Atau jalankan file uji mandiri di direktori scripts / tests
     node scripts/run-evals-test.js
     ```
   - **Evaluations / Verification Scripts**: For skill/prompt/doc repos, run validation & eval suites:
     ```bash
     node scripts/validate-skills.js
     node scripts/validate-reference-links.js
     ```
   - **No Test Suites Detected**: If the project genuinely has no test files or runners, flag as `BLOCKED` (or `WARN` for pure documentation repos) with explicit recommendation and minimal test setup template.
2. **Evaluate Coverage & Flakiness**:
   - Verify critical business logic paths meet minimum coverage standards (≥80% unit coverage recommended).
   - Check for flaky tests, race conditions, or unhandled promise rejections.
3. **Run End-to-End / Browser Tests (if present)**:
   - Execute integration/E2E test runners (Playwright, Cypress, Vitest browser mode, or `browser-testing-with-devtools`).
   - Validate HTTP status codes, API payload schemas, and UI render stability.

---

### Gate 4: Security & Dependency Hygiene

Identify vulnerabilities, supply chain risks, exposed secrets, and licensing blockers.

1. **Dependency Vulnerability Scan**:
   - Audit installed package trees for known CVEs:
     ```bash
     npm audit --production
     ```
2. **Secret & Sensitive Data Scan**:
   - Confirm no API keys, private credentials, or `.env` files are tracked in git:
     ```bash
     git grep -E "(BEGIN (RSA |OPENSSH )?PRIVATE KEY|AIzaSy|sk_live_[0-9a-zA-Z]{24})"
     ```
3. **Input Sanitization & Boundary Validation**:
   - Audit public boundaries (API endpoints, form inputs, database queries) against OWASP Top 10 vulnerabilities following `security-and-hardening`.

---

### Gate 5: Production Build & Asset Integrity

Prove that the code compiles, bundles, and packages cleanly for production without runtime regressions.

1. **Run Production Build**:
   - Verify build completes with return code `0`:
     ```bash
     npm run build
     ```
2. **Audit Bundle & Artifact Sizes**:
   - Inspect build outputs in `dist/` or `build/`.
   - Check for unexpected bundle bloat, oversized chunks, or duplicate dependencies.
3. **Validate Asset References**:
   - Confirm all generated asset URLs, favicons, fonts, and stylesheets resolve properly.

---

### Gate 6: QC Verdict & Scorecard

Synthesize all findings into an objective, actionable Quality Control Scorecard.

#### Scorecard Structure

```markdown
# Project Quality Control Scorecard

**Audit Date**: YYYY-MM-DD
**Repository**: [Repo Name / Path]
**Status**: [PASSED | WARN | BLOCKED]

## Gate Summary

| Gate | Category | Tool / Command | Status | Issues Found |
|---|---|---|---|---|
| 1 | Baseline & Config | Manifest inspection | PASS | 0 |
| 2 | Static & Types | `npm run lint`, `tsc` | PASS | 0 |
| 3 | Automated Tests | `npm test` | PASS | 0 failed, 92% coverage |
| 4 | Security & Hygiene | `npm audit`, Secret Scan | WARN | 1 moderate CVE |
| 5 | Build & Artifacts | `npm run build` | PASS | Bundle within budget |

## Detailed Findings

### [Gate Number] - [Category]
- **Severity**: [Blocker | Critical | Moderate | Minor]
- **Location**: `path/to/file:line`
- **Issue**: Precise description of defect or failure.
- **Remediation**: Exact step or command to resolve.

## Remediation Plan
1. [ ] Fix Blocker items immediately before merge/deployment.
2. [ ] Schedule Moderate/Minor items in follow-up tasks.
```

---

## Gate Enforcement Rules

| Level | Criteria | Action Required |
|---|---|---|
| **BLOCKER** | Failing tests, broken build, syntax/type errors, high/critical CVEs, leaked secrets | Release is **BLOCKED**. Immediate fix required. |
| **WARN** | Coverage below threshold, moderate CVEs in devDependencies, bundle size warning | Allowed for non-production merge if documented and ticketed. |
| **PASS** | All gates pass with 0 errors and zero blocker warnings | Codebase approved for release / merge. |

---

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "The tests failed, but it's only an unrelated test." | A broken test means broken truth. Fix or delete the invalid test; never ignore a red suite. |
| "Lint warnings are just formatting, not bugs." | Lint rules catch real runtime errors and type leaks. Zero-warning tolerance prevents gradual rot. |
| "I will run the build on CI instead of locally." | If it fails on CI, you waste developer time and pipeline resources. Always verify builds locally. |
| "This is an internal tool, security audit isn't needed." | Internal tools often have broader permissions. A credential leak is catastrophic anywhere. |
| "We don't have time for a full QC pass." | A structured 6-gate QC pass takes minutes and prevents hours of emergency rollbacks. |

---

## Red Flags

- Skipping Gates 4 (Security) or 5 (Build) because tests passed.
- Running tests with `--passWithNoTests` or disabling linter rules via inline comments during audit.
- Approving a project QC pass when `git status` contains unstaged or untracked changes.
- Marking a QC report as PASSED without pasting the actual command output evidence.

---

## Verification

Before declaring an End-to-End Quality Control audit complete, confirm:

- [ ] All 6 gates have been executed in sequence with real command outputs.
- [ ] Linter and type checker exited with status `0`.
- [ ] 100% of test suites passed without skipped failing assertions.
- [ ] Dependency security audit shows 0 high or critical vulnerabilities.
- [ ] Production build succeeds cleanly without broken paths.
- [ ] The QC Scorecard is fully populated with specific evidence and remediation steps.
