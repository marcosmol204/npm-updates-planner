# Express.js 5.2.1 — Dependency Audit Report

> Generated: 2026-03-03 | Audit scope: all production & dev dependencies

---

## 1. Project Overview

| Property | Value |
|----------|-------|
| **Package** | `express` v5.2.1 |
| **License** | MIT |
| **Runtime** | Node.js >= 18 (local: v20.19.1) |
| **Package manager** | npm 10.8.2 |
| **Monorepo** | No — single-package repository |
| **Language** | JavaScript (CommonJS, no TypeScript) |
| **Framework** | Express.js (this IS the framework) |
| **Test runner** | Mocha ^10.7.3 + Supertest ^6.3.0 |
| **Coverage** | nyc ^17.1.0 (Istanbul) |
| **Linting** | ESLint 8.47.0 (.eslintrc.yml, no plugins) |
| **Bundler** | None — library ships raw CJS |
| **CI** | GitHub Actions (CodeQL, artifact upload/download) |

### Source structure

```
lib/
  application.js    — app factory & settings
  express.js        — main exports
  request.js        — req prototype extensions
  response.js       — res prototype extensions
  utils.js          — internal helpers
  view.js           — template rendering
test/               — 85 Mocha test files
test/acceptance/    — integration tests against examples/
examples/           — 22 example apps
```

### Dependency counts

| Category | Count |
|----------|-------|
| Production (`dependencies`) | 28 |
| Development (`devDependencies`) | 16 |
| Total installed (incl. transitive) | 385 |

---

## 2. Unused Packages

After exhaustive codebase search (all `require()` calls, config references, script invocations, and implicit engine resolution across `lib/`, `test/`, `examples/`, and root configs):

| Package | Status | Justification |
|---------|--------|---------------|
| — | — | **No unused packages found** |

All 16 devDependencies are actively referenced. Several packages are only used in example apps (`connect-redis`, `cookie-session`, `hbs`, `pbkdf2-password`, `vhost`), but examples are part of the shipped repository and validated by acceptance tests.

---

## 3. Upgrade Recommendations by Priority

### P1 — Critical (Security / EOL)

| Package | Type | Current | Target | Reason |
|---------|------|---------|--------|--------|
| **eslint** | dev | 8.47.0 | 9.x+ | **EOL since Oct 5 2024** — no security patches |
| **mocha** | dev | ^10.7.3 (10.8.2) | 11.7.5 | Transitive CVEs: serialize-javascript RCE (CVSS 8.1), minimatch ReDoS (CVSS 7.5), diff DoS |

### P2 — High (Patch-level security + significant bug fixes)

| Package | Type | Current | Target | Reason |
|---------|------|---------|--------|--------|
| **body-parser** | prod | ^2.2.1 (2.2.1) | 2.2.2 | Patch release with fixes post-CVE-2025-13466 |
| **send** | prod | ^1.1.0 (1.2.0) | 1.2.1 | Patch update, historical CVE-2024-43800 remediated |
| **serve-static** | prod | ^2.2.0 (2.2.0) | 2.2.1 | Patch update, historical CVE-2024-43800 remediated |
| **qs** | prod | ^6.14.2 (6.14.0) | 6.15.0 | Installed version below declared range; update available |

### P3 — Medium (Major version bumps, DX improvements)

| Package | Type | Current | Target | Reason |
|---------|------|---------|--------|--------|
| **supertest** | dev | ^6.3.0 (6.3.4) | 7.2.2 | Major rewrite with ESM support, better types |
| **connect-redis** | dev | ^8.0.1 (8.1.0) | 9.0.0 | Major update, API improvements |
| **nyc** | dev | ^17.1.0 (17.1.0) | 18.0.0 | Major update with Node 20+ improvements |
| **marked** | dev | ^15.0.3 (15.0.12) | 17.0.3 | Two major versions behind, DX/perf improvements |
| **ejs** | dev | ^3.1.10 (3.1.10) | 4.0.1 | Major update; transitive brace-expansion ReDoS in 3.x |
| **express-session** | dev | ^1.18.1 (1.18.2) | 1.19.0 | Minor update with bug fixes |

### P4 — Low (Nice-to-have / monitoring)

| Package | Type | Current | Target | Reason |
|---------|------|---------|--------|--------|
| **cookie** | prod | ^0.7.1 (0.7.2) | 1.1.1 | v1.x available but outside caret range; would require range bump |
| **after** | dev | 0.8.2 | 0.8.2 | Stale (2016) but stable; consider replacing with native Promise patterns |
| **method-override** | dev | 3.0.0 | 3.0.0 | Stale (2018) but stable; example-only usage |
| **hbs** | dev | 4.2.0 | 4.2.0 | Stale (2021); example-only usage |
| **pbkdf2-password** | dev | 1.2.1 | 1.2.1 | Stale (2016); example-only usage; consider native `crypto.pbkdf2` |
| **escape-html** | prod | ^1.0.3 (1.0.3) | 1.0.3 | Last updated 2015; feature-complete, no CVEs |
| **once** | prod | ^1.4.0 (1.4.0) | 1.4.0 | Last updated 2016; feature-complete, no CVEs |
| **parseurl** | prod | ^1.3.3 (1.3.3) | 1.3.3 | Last updated 2019; feature-complete, no CVEs |
| **vary** | prod | ^1.1.2 (1.1.2) | 1.1.2 | Last updated 2017; feature-complete, no CVEs |

---

## 4. Dependency Ecosystem Clusters

Packages that share peer dependencies, must move together, or belong to the same functional domain.

### Cluster A: Express Core HTTP Stack (Production)

```
body-parser ^2.2.1 ──┐
qs ^6.14.2            │
content-type ^1.0.5   ├── Request parsing
type-is ^2.0.1        │
accepts ^2.0.0        │
content-disposition ^1.0.0 ┘

send ^1.1.0 ──────────┐
serve-static ^2.2.0   ├── Static file serving
mime-types ^3.0.0      │
range-parser ^1.2.1   ┘

router ^2.2.0 ────────┐
parseurl ^1.3.3       ├── Routing
proxy-addr ^2.0.7     ┘

cookie ^0.7.1 ────────┐
cookie-signature ^1.2.1 ── Cookie handling
                       ┘
```

**Coupling note:** `body-parser`, `send`, `serve-static`, and `router` are all expressjs-org packages designed to work together. Patch updates within semver ranges are safe independently, but major bumps should be coordinated.

### Cluster B: Test Infrastructure (Dev)

```
mocha ^10.7.3 ────────┐
supertest ^6.3.0      ├── Test execution & assertions
after 0.8.2           │
nyc ^17.1.0           ┘── Coverage
```

**Coupling note:** Mocha 11.x drops support for some reporters; nyc 18.x may change Istanbul instrumentation. Test together before merging.

### Cluster C: Linting (Dev)

```
eslint 8.47.0 ────────── Standalone (no plugins)
```

**Coupling note:** ESLint 9.x introduces flat config format (eslint.config.js) and drops .eslintrc support. Migration required. No ESLint plugins are used, simplifying the upgrade.

### Cluster D: Example App Ecosystem (Dev)

```
express-session ^1.18.1 ─┐
connect-redis ^8.0.1     ├── Session examples
cookie-session 2.1.1     ┘

ejs ^3.1.10 ─────────────┐
hbs 4.2.0                ├── Template engine examples
marked ^15.0.3           ┘

morgan 1.10.1 ────────────── Logging examples
method-override 3.0.0 ────── REST examples
cookie-parser 1.4.7 ──────── Cookie examples
pbkdf2-password 1.2.1 ────── Auth example
vhost ~3.0.2 ──────────────── Virtual host example
```

**Coupling note:** These are loosely coupled. Each can be updated independently. `connect-redis` 9.x changes the constructor API and requires `express-session` compat testing.

---

## 5. CVE Reference Table

### Active / Applicable Vulnerabilities (in installed tree)

| CVE / Advisory | Package | Severity | CVSS | Affected Range | Installed | Fix Version | Direct? |
|---------------|---------|----------|------|----------------|-----------|-------------|---------|
| [GHSA-5c6j-r48x-rmvq](https://github.com/advisories/GHSA-5c6j-r48x-rmvq) | serialize-javascript | **High** | 8.1 | <=7.0.2 | 7.0.2 (via mocha) | 7.0.3+ | No (transitive via mocha) |
| [GHSA-3ppc-4f35-3m26](https://github.com/advisories/GHSA-3ppc-4f35-3m26) | minimatch | **High** | 7.5 | <3.1.3, >=5.0.0 <5.1.7 | 3.1.2 (via glob, mocha) | 3.1.3+ / 5.1.7+ | No (transitive) |
| [GHSA-7r86-cg39-jmmj](https://github.com/advisories/GHSA-7r86-cg39-jmmj) | minimatch | **High** | 7.5 | <3.1.3, >=5.0.0 <5.1.8 | 3.1.2 | 3.1.3+ / 5.1.8+ | No (transitive) |
| [GHSA-23c5-xmqv-rm74](https://github.com/advisories/GHSA-23c5-xmqv-rm74) | minimatch | **High** | 7.5 | <3.1.4, >=5.0.0 <5.1.8 | 3.1.2 | 3.1.4+ / 5.1.8+ | No (transitive) |
| [GHSA-2g4f-4pwh-qvx6](https://github.com/advisories/GHSA-2g4f-4pwh-qvx6) | ajv | **Moderate** | — | <6.14.0 | 6.12.6 (via eslint) | 6.14.0+ | No (transitive via eslint) |
| [GHSA-73rr-hh4g-fpgx](https://github.com/advisories/GHSA-73rr-hh4g-fpgx) | diff | **Low** | — | >=5.0.0 <5.2.2 | 5.2.0 (via mocha) | 5.2.2+ | No (transitive via mocha) |

### Historical / Already Patched (for reference)

| CVE / Advisory | Package | Status |
|---------------|---------|--------|
| CVE-2025-13466 | body-parser | Fixed in >=2.2.1; installed 2.2.1 is safe |
| CVE-2024-47764 | cookie | Fixed in >=0.7.0; installed 0.7.2 is safe |
| CVE-2024-43800 | send / serve-static | Fixed in send >=1.1.0 / serve-static >=2.1.0; installed versions safe |
| CVE-2022-24999 | qs | Fixed in >=6.10.3; installed 6.14.0 is safe |

**Key finding:** All active vulnerabilities are in **dev-dependency transitive trees** (mocha, eslint). Zero active CVEs affect the production dependency tree.

---

## 6. Recommended Upgrade Order

Execute in this sequence to minimize risk and maximize unblocking:

```
Phase 1 (Sprint 1) — Security & EOL
  ├── 1a. Patch prod deps: body-parser, send, serve-static, qs  [low risk, semver-compatible]
  ├── 1b. Upgrade mocha 10.x → 11.x  [resolves 4 CVEs]
  └── 1c. Migrate ESLint 8.x → 9.x  [EOL remediation, config format change]

Phase 2 (Sprint 2) — Dev tooling modernization
  ├── 2a. Upgrade supertest 6.x → 7.x
  ├── 2b. Upgrade nyc 17.x → 18.x
  └── 2c. Upgrade connect-redis 8.x → 9.x + express-session → 1.19.0

Phase 3 (Sprint 3) — Example & template ecosystem
  ├── 3a. Upgrade marked 15.x → 17.x
  ├── 3b. Upgrade ejs 3.x → 4.x
  └── 3c. Evaluate stale packages (after, hbs, pbkdf2-password, method-override)

Phase 4 (Backlog) — Opportunistic
  └── 4a. Bump cookie ^0.7.x → ^1.x (requires range change in package.json)
```

### Risk Assessment Summary

| Risk Level | Count | Details |
|------------|-------|---------|
| **No risk** (semver patch) | 4 | body-parser, send, serve-static, qs |
| **Low risk** (minor bump) | 3 | express-session, connect-redis (within range), marked (within range) |
| **Medium risk** (major bump) | 5 | mocha, supertest, nyc, ejs, connect-redis (to 9.x) |
| **High risk** (config migration) | 1 | ESLint 8→9 (flat config migration) |
| **Monitoring only** | 6 | escape-html, once, parseurl, vary, after, hbs |
