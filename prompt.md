# NPM Dependency Upgrade Strategy — Full Audit & Action Plan

## Role & Objective

You are an expert Node.js / Full Stack Developer tasked with planning a
comprehensive npm package upgrade strategy for this project. Your goal is to
produce a **complete, actionable audit** — not to upgrade anything yet.

---

## Phase 1 — Research & Audit

Perform each step sequentially. Be thorough — read configs, search the
codebase, and verify every claim before writing it down.

### Step 1: Read the Baseline

- Read `package.json` (root and workspace apps/packages if monorepo).
- Read build configs: Babel, Webpack/Vite/esbuild, tsconfig, ESLint, Nx/Turbo.
- Note the runtime (Node version), framework versions, and test frameworks.

### Step 2: Detect Unused Dependencies

For every dependency that looks suspicious (no obvious import, niche name,
possible transitive-only usage):

- Search the **entire codebase** for `require('pkg')`, `import ... from 'pkg'`,
  and any config-file references (babel plugins, eslint plugins, jest
  transforms, etc.).
- A package is "unused" only if it has **zero references** anywhere — code,
  configs, scripts, and build tooling.
- List each unused package with a one-line justification.

### Step 3: Identify Outdated & Vulnerable Packages

- For every pinned or significantly outdated dependency, check:
  - Current version vs. latest stable.
  - Known CVEs (include CVE ID, CVSS score, affected range, fix version).
  - Breaking-change highlights from changelogs.
- Flag packages that are **End-of-Life** (e.g., no security patches).

### Step 4: Map Dependency Ecosystems

Identify **tightly-coupled clusters** — groups of packages that share
peerDependencies or must be upgraded together. Common clusters:

- Build tooling (bundler + plugins + framework adapters)
- Monorepo orchestrator (Nx/Turbo + related plugins)
- Database layer (ORM + driver + plugins)
- UI framework (React/Vue + router + state management)
- Linting & formatting (ESLint + plugins + Prettier)

For each cluster, note the **current version range**, the **target version**,
and which packages are coupled.

### Step 5: Map Risks & Opportunities

- Flag any package with no maintained alternative or no CVE fix available.
- Note migrations that unlock other upgrades (e.g., "upgrading X unblocks Y").
- Highlight quick wins (major version bumps that are drop-in compatible).

### Phase 1 Output

Write the full audit to a markdown file. Organize by:

1. Project overview (runtime, framework, build tools, monorepo structure)
2. Unused packages (table: package, justification)
3. Upgrade recommendations by priority
4. Dependency ecosystem clusters (diagram or table)
5. CVE reference table (CVE ID, package, CVSS, affected range, fix version)
6. Recommended upgrade order

---

## Phase 2 — Actionable Ticket Plan

Translate Phase 1 findings into **Jira-style tickets**. Follow these rules:

### Rule 1: Ecosystem Batching

Group packages into a **single ticket** when:

- They share peerDependencies (e.g., `vite` + `@vitejs/plugin-react` + `vitest`)
- One upgrade is meaningless without the other
- They belong to the same ecosystem cluster from Phase 1

**Never** batch unrelated packages just because they share a priority level.

### Rule 2: Prioritization

| Priority | Criteria | Examples |
|----------|----------|---------|
| **P1 Critical** | Active CVEs (CVSS ≥ 7), broken builds, EOL with no patches | Security fixes, EOL database drivers |
| **P2 High** | Performance gains, significant bug fixes, compliance deadlines | Framework upgrades with deadlines |
| **P3 Medium** | Standard version bumps, DX improvements, new features | Build tool upgrades, linter updates |
| **P4 Low** | Nice-to-have, cosmetic, non-blocking | Removing unused deps, minor tooling |

### Ticket Format

For each ticket, include:

```
### DEPS-NNN · [P1/P2/P3/P4] — Title

**Epic/Theme:** <category>

**Packages:**

| Package | Current | Target |
|---------|---------|--------|
| pkg     | x.y.z   | a.b.c  |

**Why grouped:** <1-2 sentences on coupling rationale>

**Risk & Rollback:** <risk level> · <rollback strategy>

**Acceptance Criteria:**
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] All tests pass, no regressions
```

### Phase 2 Output

Write the full ticket plan to a markdown file. End with an **execution
summary table** mapping tickets to suggested sprint phases.

---

## Phase 3 — Visual Dashboard

Create a **single-file HTML dashboard** (embedded CSS + JS, no external
dependencies) that visualizes the entire audit:

- Stat cards for ticket counts by priority (P1–P4)
- Security health gauge (0–100 score based on CVE severity)
- Unused packages to remove (chip/tag visualization)
- CVE reference table (sortable or filterable)
- Dependency ecosystem cluster diagrams
- All tickets as expandable cards with priority-based filtering
- Execution roadmap / timeline
- Dark theme, modern typography, responsive layout

### Phase 3 Output

Write to an HTML file.

---

## General Rules

- **Do not upgrade anything.** This is a planning exercise only.
- **Verify before claiming.** Never mark a package as unused without searching
  the full codebase. Never claim a CVE without confirming the affected range.
- **Be specific.** Include exact version numbers, CVE IDs, and file paths.
- **Prefer accuracy over speed.** Use parallel research agents where possible,
  but validate findings before including them.
- Save each phase output to a file before proceeding to the next.
