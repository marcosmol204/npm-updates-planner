# Express.js 5.2.1 — Dependency Upgrade Ticket Plan

> Generated: 2026-03-03 | Derived from: DEPENDENCY_AUDIT.md

---

## Ticket Index

| Ticket | Priority | Title | Sprint |
|--------|----------|-------|--------|
| DEPS-001 | P1 | Patch production HTTP stack dependencies | Sprint 1 |
| DEPS-002 | P1 | Upgrade Mocha 10.x to 11.x (resolve transitive CVEs) | Sprint 1 |
| DEPS-003 | P1 | Migrate ESLint 8.x to 9.x (EOL remediation) | Sprint 1 |
| DEPS-004 | P2 | Upgrade Supertest 6.x to 7.x | Sprint 2 |
| DEPS-005 | P2 | Upgrade nyc 17.x to 18.x | Sprint 2 |
| DEPS-006 | P2 | Upgrade session ecosystem (connect-redis + express-session) | Sprint 2 |
| DEPS-007 | P3 | Upgrade marked 15.x to 17.x | Sprint 3 |
| DEPS-008 | P3 | Upgrade ejs 3.x to 4.x | Sprint 3 |
| DEPS-009 | P4 | Evaluate and modernize stale example-only dependencies | Backlog |
| DEPS-010 | P4 | Bump cookie range from ^0.7.x to ^1.x | Backlog |

---

## Tickets

### DEPS-001 · [P1] — Patch production HTTP stack dependencies

**Epic/Theme:** Security & Maintenance

**Packages:**

| Package | Current | Target |
|---------|---------|--------|
| body-parser | 2.2.1 | 2.2.2 |
| send | 1.2.0 | 1.2.1 |
| serve-static | 2.2.0 | 2.2.1 |
| qs | 6.14.0 | 6.15.0 |

**Why grouped:** All four are expressjs-org production dependencies in the HTTP request/response pipeline. These are semver-compatible patch updates that can be applied together with a single `npm update`. The `qs` update also corrects a mismatch where the installed version (6.14.0) falls below the declared range (^6.14.2).

**Risk & Rollback:** Very Low · `git revert` + `npm ci` restores previous lock file. All updates are within declared semver ranges (except qs which is being brought into compliance).

**Acceptance Criteria:**
- [ ] `npm ls body-parser` shows >=2.2.2
- [ ] `npm ls send` shows >=1.2.1
- [ ] `npm ls serve-static` shows >=2.2.1
- [ ] `npm ls qs` shows >=6.14.2
- [ ] `npm test` passes with zero failures
- [ ] `npm audit` shows no new vulnerabilities in production tree
- [ ] All tests pass, no regressions

---

### DEPS-002 · [P1] — Upgrade Mocha 10.x to 11.x (resolve transitive CVEs)

**Epic/Theme:** Security — Dev Tooling

**Packages:**

| Package | Current | Target |
|---------|---------|--------|
| mocha | 10.8.2 | 11.7.5 |

**Transitively resolves:**

| Vulnerable Package | CVE / Advisory | Severity | CVSS |
|-------------------|----------------|----------|------|
| serialize-javascript <=7.0.2 | GHSA-5c6j-r48x-rmvq (RCE) | High | 8.1 |
| minimatch <3.1.4 | GHSA-23c5-xmqv-rm74 (ReDoS) | High | 7.5 |
| minimatch <3.1.3 | GHSA-3ppc-4f35-3m26 (ReDoS) | High | 7.5 |
| minimatch <3.1.3 | GHSA-7r86-cg39-jmmj (ReDoS) | High | 7.5 |
| diff >=5.0.0 <5.2.2 | GHSA-73rr-hh4g-fpgx (DoS) | Low | — |

**Why grouped:** All five CVEs exist in Mocha's transitive dependency tree. Upgrading Mocha to 11.x pulls in fixed versions of all transitive deps. Upgrading them individually is not possible (they are not direct deps).

**Risk & Rollback:** Medium · Mocha 11.x drops Node 14/16 support (not relevant — project requires Node >=18). Breaking changes include: removal of `--grep` short flag `-g`, changes to parallel mode defaults, and updated reporter APIs. Rollback: revert `package.json` + `package-lock.json` and run `npm ci`.

**Acceptance Criteria:**
- [ ] `mocha --version` shows 11.x
- [ ] `npm test` passes — all 800+ tests green
- [ ] `npm run test-tap` passes
- [ ] `npm run test-cov` generates coverage report without errors
- [ ] `npm audit` shows serialize-javascript, minimatch, and diff vulnerabilities resolved
- [ ] CI pipeline (GitHub Actions) passes
- [ ] All tests pass, no regressions

---

### DEPS-003 · [P1] — Migrate ESLint 8.x to 9.x (EOL remediation)

**Epic/Theme:** Security — EOL Remediation

**Packages:**

| Package | Current | Target |
|---------|---------|--------|
| eslint | 8.47.0 | 9.x (latest stable) |

**Transitively resolves:**

| Vulnerable Package | CVE / Advisory | Severity |
|-------------------|----------------|----------|
| ajv <6.14.0 | GHSA-2g4f-4pwh-qvx6 (ReDoS) | Moderate |

**Why grouped:** Single package, but ESLint 9.x requires a config format migration from `.eslintrc.yml` to `eslint.config.js` (flat config). The ajv vulnerability in ESLint 8's transitive tree is also resolved.

**Risk & Rollback:** High (config migration) · ESLint 9 introduces the flat config system and deprecates the legacy `.eslintrc.*` format. Since Express uses no ESLint plugins (only built-in rules), the migration scope is small. Rollback: revert both `package.json` and `.eslintrc.yml` / `eslint.config.js`.

**Migration steps:**
1. Create `eslint.config.js` with equivalent rules from `.eslintrc.yml`
2. Remove `.eslintrc.yml`
3. Update `package.json` eslint version
4. Run `npm run lint` and fix any new findings

**Acceptance Criteria:**
- [ ] `npx eslint --version` shows 9.x
- [ ] `.eslintrc.yml` removed, `eslint.config.js` created
- [ ] `npm run lint` passes with zero errors
- [ ] All existing lint rules (eol-last, eqeqeq, indent, no-trailing-spaces, no-unused-vars, no-restricted-globals) are preserved
- [ ] `npm audit` no longer flags ajv vulnerability
- [ ] All tests pass, no regressions

---

### DEPS-004 · [P2] — Upgrade Supertest 6.x to 7.x

**Epic/Theme:** Dev Tooling Modernization

**Packages:**

| Package | Current | Target |
|---------|---------|--------|
| supertest | 6.3.4 | 7.2.2 |

**Why grouped:** Single package. Supertest is the HTTP testing library used in 83 test files. It must be upgraded independently to isolate any breaking changes from other test infrastructure upgrades.

**Risk & Rollback:** Medium · Supertest 7.x drops callback-based API in favor of Promises, changes assertion chaining behavior, and requires Node >=14. Express tests heavily use the callback pattern (`.end(done)`). Many test files may need updates. Rollback: revert package.json + lock file.

**Acceptance Criteria:**
- [ ] `npm ls supertest` shows 7.x
- [ ] All 83 test files using supertest pass
- [ ] `npm test` passes with zero failures
- [ ] No changes to production code
- [ ] All tests pass, no regressions

---

### DEPS-005 · [P2] — Upgrade nyc 17.x to 18.x

**Epic/Theme:** Dev Tooling Modernization

**Packages:**

| Package | Current | Target |
|---------|---------|--------|
| nyc | 17.1.0 | 18.0.0 |

**Why grouped:** Single package. nyc is the Istanbul coverage tool. It should be upgraded separately from mocha to isolate coverage-specific issues.

**Risk & Rollback:** Low-Medium · nyc 18.x updates Istanbul dependencies and may change default instrumentation behavior. Coverage thresholds and report formats should be verified. Rollback: revert package.json + lock file.

**Acceptance Criteria:**
- [ ] `npx nyc --version` shows 18.x
- [ ] `npm run test-cov` generates HTML coverage report
- [ ] `npm run test-ci` generates lcov report for CI
- [ ] Coverage numbers are consistent with pre-upgrade baseline
- [ ] All tests pass, no regressions

---

### DEPS-006 · [P2] — Upgrade session ecosystem (connect-redis + express-session)

**Epic/Theme:** Example App Maintenance

**Packages:**

| Package | Current | Target |
|---------|---------|--------|
| connect-redis | 8.1.0 | 9.0.0 |
| express-session | 1.18.2 | 1.19.0 |

**Why grouped:** `connect-redis` is a session store adapter for `express-session`. They share a peer dependency contract — the store must implement the `express-session` Store interface. `connect-redis` 9.x changes its constructor API and should be tested against the target `express-session` version.

**Risk & Rollback:** Medium · `connect-redis` 9.x changes the constructor signature from `new RedisStore({ client })` to `new RedisStore({ prefix, client })` with different defaults. Only affects `examples/session/redis.js`. Rollback: revert package.json + lock file.

**Acceptance Criteria:**
- [ ] `examples/session/redis.js` starts without errors (with Redis available)
- [ ] `examples/session/index.js` works with updated express-session
- [ ] `npm ls connect-redis` shows 9.x
- [ ] `npm ls express-session` shows 1.19.x
- [ ] All tests pass, no regressions

---

### DEPS-007 · [P3] — Upgrade marked 15.x to 17.x

**Epic/Theme:** Example App Maintenance

**Packages:**

| Package | Current | Target |
|---------|---------|--------|
| marked | 15.0.12 | 17.0.3 |

**Why grouped:** Single package. Used in `examples/markdown/` and `examples/view-constructor/`. Independent of all other dependencies.

**Risk & Rollback:** Low · Marked follows semver. Two major versions bring API simplification and performance improvements. The example usage is minimal (`marked.parse(str)`). Rollback: revert package.json + lock file.

**Acceptance Criteria:**
- [ ] `examples/markdown/` renders markdown correctly
- [ ] `examples/view-constructor/` renders markdown correctly
- [ ] Acceptance tests for markdown example pass
- [ ] All tests pass, no regressions

---

### DEPS-008 · [P3] — Upgrade ejs 3.x to 4.x

**Epic/Theme:** Example App Maintenance

**Packages:**

| Package | Current | Target |
|---------|---------|--------|
| ejs | 3.1.10 | 4.0.1 |

**Why grouped:** Single package. EJS is used as a view engine in `examples/ejs/` and `examples/mvc/`. Independent of other template engines. Also resolves transitive brace-expansion ReDoS (CVE-2025-5889) present in ejs 3.x tree.

**Risk & Rollback:** Low-Medium · EJS 4.x may change template compilation options and async rendering behavior. Express's `app.engine()` / `app.set('view engine', 'ejs')` integration should be verified. Rollback: revert package.json + lock file.

**Acceptance Criteria:**
- [ ] `examples/ejs/` renders templates correctly
- [ ] `examples/mvc/` works with EJS engine
- [ ] Acceptance tests for ejs and mvc examples pass
- [ ] `npm audit` no longer flags brace-expansion transitive CVE
- [ ] All tests pass, no regressions

---

### DEPS-009 · [P4] — Evaluate and modernize stale example-only dependencies

**Epic/Theme:** Tech Debt Reduction

**Packages:**

| Package | Current | Latest | Last Published | Usage |
|---------|---------|--------|----------------|-------|
| after | 0.8.2 | 0.8.2 | 2016 | Test utility (10 files) |
| hbs | 4.2.0 | 4.2.0 | 2021 | MVC example view engine |
| pbkdf2-password | 1.2.1 | 1.2.1 | 2016 | Auth example |
| method-override | 3.0.0 | 3.0.0 | 2018 | 2 example apps |

**Why grouped:** All four packages are stale (no updates in 2+ years), pinned to their latest version, and used only in test utilities or examples. They share the common risk profile of unmaintained code.

**Risk & Rollback:** Very Low · These packages work fine as-is. This ticket is for evaluation only — determine whether to keep, replace, or remove each one.

**Evaluation criteria:**
- `after`: Could be replaced with native `Promise.all` / counter pattern
- `hbs`: Could be replaced with a maintained Handlebars adapter
- `pbkdf2-password`: Could be replaced with native `crypto.pbkdf2`
- `method-override`: Stable, no security concerns; keep unless removing examples

**Acceptance Criteria:**
- [ ] Each package evaluated for replacement feasibility
- [ ] Decision documented (keep / replace / remove) for each
- [ ] If replacements made: affected test/example files updated and passing
- [ ] All tests pass, no regressions

---

### DEPS-010 · [P4] — Bump cookie range from ^0.7.x to ^1.x

**Epic/Theme:** Dependency Modernization

**Packages:**

| Package | Current | Target |
|---------|---------|--------|
| cookie | 0.7.2 | 1.1.1 |

**Why grouped:** Single package. The `cookie` module is a production dependency used for cookie parsing/serialization. The current caret range `^0.7.1` cannot resolve to 1.x — a `package.json` range change is required.

**Risk & Rollback:** Medium · Cookie 1.x changes the `serialize()` API to be stricter about cookie attribute validation. This affects `res.cookie()` in `lib/response.js`. Thorough testing of cookie-related functionality required. Rollback: revert package.json + lock file.

**Acceptance Criteria:**
- [ ] `package.json` updated: `"cookie": "^1.1.0"`
- [ ] `lib/response.js` cookie handling tested
- [ ] `test/res.cookie.js` passes
- [ ] `test/res.clearCookie.js` passes
- [ ] `test/req.signedCookies.js` passes
- [ ] `examples/cookies/` works correctly
- [ ] All tests pass, no regressions

---

## Execution Summary

| Sprint | Tickets | Focus | Est. Risk |
|--------|---------|-------|-----------|
| **Sprint 1** | DEPS-001, DEPS-002, DEPS-003 | Security & EOL — clear all active CVEs and EOL status | Medium-High |
| **Sprint 2** | DEPS-004, DEPS-005, DEPS-006 | Dev tooling modernization — update test/coverage/example infra | Medium |
| **Sprint 3** | DEPS-007, DEPS-008 | Example app template engines — update rendering libraries | Low |
| **Backlog** | DEPS-009, DEPS-010 | Tech debt & opportunistic — stale deps and range bumps | Low |

### Dependency between tickets

```
DEPS-001 (prod patches) ─── independent, do first
DEPS-002 (mocha 11.x) ───── do before DEPS-004 (supertest) and DEPS-005 (nyc)
DEPS-003 (eslint 9.x) ───── independent of test tooling
DEPS-004 (supertest 7.x) ── after DEPS-002 (both affect test infrastructure)
DEPS-005 (nyc 18.x) ─────── after DEPS-002 (both affect test/coverage pipeline)
DEPS-006 (session) ──────── independent of test tooling
DEPS-007 (marked) ───────── independent
DEPS-008 (ejs) ──────────── independent
DEPS-009 (stale deps) ───── after all other upgrades settled
DEPS-010 (cookie ^1.x) ──── independent but requires careful testing
```
