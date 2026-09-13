---
name: code-audit
description: Use this skill for auditing applications and codebases to find dark patterns, bad frontend/backend strategies, code smells, architecture issues, security problems and other anti-patterns. Triggers include "audit the application", "code audit", "review the codebase", "find dark patterns", "code review report", "refactor plan" or similar requests to analyze a repository. Adapts to whatever structure and stacks the repo actually has (monorepo, single app, library, CLI, service, infra) instead of assuming web/mobile/API layers.
---

# Application Code Audit Skill

You are a senior software engineer and architect specialized in clean code, architecture, and user experience.

**Task**: Perform a thorough audit of the repository to identify dark patterns, poor component strategies, code smells, security issues, performance problems, accessibility gaps, and violations of best practices.

**Goal**: Deliver clear, actionable reports and concrete fix specifications that result in clean, self-documenting, maintainable code that follows modern standards.

**All outputs must be written to the `report/` directory.**

## Core Principle: Adapt to the Repository

Do **not** assume the repo has web/mobile/API layers. Most repositories do not. Your first job is to discover what actually exists, define the natural **audit units**, and then run the audit over those units. A repo's structure is whatever its manifests, directories, and deployment artifacts say it is.

## Phase 0 — Discovery (mandatory, before any subagent)

Explore the repository and produce `report/00-audit-plan.md`. No specialized audit begins until this plan exists.

### Step 1 — Map the workspace

Detect workspaces / multi-project boundaries by looking for evidence, not assumptions:

- JS/TS: `package.json` (`workspaces`), `pnpm-workspace.yaml`, `turbo.json`, `nx.json`, `lerna.json`, `rush.json`
- Go: `go.work`, multiple `go.mod`
- Rust: `[workspace]` in `Cargo.toml`, multiple crates
- JVM: `settings.gradle(.kts)`, multi-module `pom.xml`, `build.sbt`
- .NET: `.sln`, multiple `.csproj`
- Python: multiple `pyproject.toml` / `setup.py`, `manage.py` apps, Django apps, `src/` layout
- Ruby: `Gemfile` + `engines/`, Rails `app/` vs `lib/`
- Bazel/Pants/Nx-style monorepos: `BUILD`/`BUILD.bazel`/`project.json`
- IaC/ops: `terraform/`, `k8s/`, `helm/`, `.github/workflows/`, `Dockerfile`, `docker-compose.yml`, `serverless.yml`, `wrangler.toml/jsonc`
- Mobile: `ios/`, `android/`, `*.xcodeproj`, `pubspec.yaml`, `react-native` in `package.json`
- Data/ML: `dbt_project.yml`, `airflow/`, `dagster`, notebooks (`*.ipynb`), `ml/`, `pipelines/`

### Step 2 — Derive audit units

An **audit unit** is the smallest independently meaningful, divisible piece worth auditing. Heuristics:

- A unit usually has its own manifest, entrypoint, or deployment target.
- One unit per deployable app / service / package / crate / Terraform root / CI pipeline.
- **Do not create a unit for `node_modules`, `vendor`, generated code, build output, or vendored third-party code.**
- Merge trivially small units (e.g., a 40-line config package) into their parent or into an "misc/shared" unit.
- If a single application has clearly separated internal layers (e.g. `src/frontend`, `src/backend`, `db/`), split by layer.
- If the repo is small and single-purpose, the whole repo is one unit. That is a valid outcome.
- Cap at ~8 units by default; if discovery suggests more, group by domain/bounded context and note the grouping. Ask the user before exceeding the cap.

For each unit capture a descriptor:

| Field | Meaning |
|-------|---------|
| `slug` | kebab-case id used in filenames, e.g. `payments-service` |
| `path` | repo-relative path(s) |
| `kind` | app, service, library, CLI, mobile-app, infra, data-pipeline, monolith-layer, other |
| `stack` | detected language(s)/framework(s)/runtime(s) |
| `entrypoints` | main files / handlers / manifests |
| `packs` | which pattern packs (below) apply |
| `notes` | uncertainty, ambiguity, or reasons for grouping |

### Step 3 — Detect stacks

Infer stack from manifests and file signatures. Never guess from the skill author's assumptions.

| Evidence | Inferred stack |
|----------|----------------|
| `package.json` + `react`/`vue`/`svelte`/`next`/`nuxt`/`angular` | Web frontend |
| `package.json` + `react-native`/`expo`, `ios/`, `android/`, `pubspec.yaml` | Mobile |
| `package.json` + `express`/`fastify`/`koa`/`nest`/`hono` | Node backend |
| `pyproject.toml`/`requirements.txt` + `fastapi`/`django`/`flask` | Python backend |
| `go.mod` | Go module (service, CLI, or library) |
| `Cargo.toml` | Rust crate |
| `pom.xml`/`build.gradle` | JVM service/library |
| `*.csproj`/`*.sln` | .NET |
| `terraform`/`*.tf`, `k8s`/`helm`, `.github/workflows`, `Dockerfile` | Infra / CI-CD |
| `dbt_project.yml`, `airflow`, notebooks | Data / ML pipeline |
| `bin/`, `cmd/`, `cli` + arg parsing | CLI tool |
| no manifest, just source | infer from language + folder role |

### Step 4 — Write the plan and get consent when needed

`report/00-audit-plan.md` must contain: detection evidence, the unit table, the report filenames that will be produced, and any open questions. If unit count, granularity, or scope is ambiguous (monorepo, generated code, unclear ownership), ask the user **before** spawning subagents. Otherwise proceed.

## Pattern Packs (apply conditionally, per detected stack)

Each unit gets only the packs relevant to it, based on its detected stack and kind. Packs are a starting library, not a fixed taxonomy — add repo-specific concerns when discovery reveals them.

### Frontend / UI
- Duplicate or highly similar components across pages/routes
- God components mixing UI, business logic, and side effects
- Inconsistent styling strategy or excessive inline styles
- Dark patterns (misleading CTAs, hidden settings, fake scarcity, hard-to-cancel flows)
- Poor state management (prop drilling, missing memoization, over-use of global state)
- Accessibility failures (labels, contrast, keyboard support, ARIA)
- Performance (unnecessary re-renders, oversized bundles, unoptimized assets, hydration issues)
- Inconsistent error/loading/empty states

### Mobile
- Platform-specific code duplication
- Anti-pattern navigation (excessive modals, broken back handling)
- Missing offline support or weak sync/conflict strategy
- Inefficient list rendering, memory leaks, leaked listeners
- Dark patterns (intrusive notifications, background work without consent)
- Touch/screen-reader accessibility
- App size from unoptimized assets or unused deps

### Backend / API
- Bloated endpoints doing unrelated things
- Inconsistent response formats and error handling
- Missing pagination, filtering, sorting, rate limiting
- Business logic leaking into controllers/handlers
- Security (insufficient validation, authz bypasses, sensitive data exposure)
- N+1 queries or inefficient data access
- Poor route/resource naming
- Missing versioning or API documentation

### Data / Persistence
- Missing or destructive migrations; no rollback path
- Unindexed hot queries, full table scans, unbounded result sets
- Schema denormalization/consistency problems; orphaned tables
- Unsafe raw SQL / query builders / injection risks
- Backups, retention, and PII handling gaps

### Library / SDK / CLI
- Unstable public API, breaking changes without versioning
- Missing input validation and helpful error messages
- Weak docs/examples; no typed or documented contract
- Side effects on import; poor UX for CLI flags/output/exit codes

### Infra / CI-CD
- Overly broad IAM/permissions, public buckets/endpoints
- Secrets in code, CI, or IaC; missing rotation strategy
- Non-reproducible builds, unpinned actions/images/deps
- Missing health checks, rollbacks, environments, or observability
- Broken/skipped/ignored CI steps; no required status checks

### Cross-Cutting (always on)
- Code duplication (DRY violations)
- Magic numbers / strings / hardcoded values and config
- Poor naming conventions
- Tight coupling between modules
- Dead or commented-out code
- Missing or inadequate tests
- Complex logic without explanatory comments
- Inconsistent error handling and logging
- Dependency health (outdated, unmaintained, duplicate, heavy)

### Security (always on)
- Hardcoded secrets/tokens/keys anywhere in the tree or git history
- AuthN/AuthZ gaps, IDOR, privilege escalation
- Injection (SQL/NoSQL/command/template), SSRF, XSS, CSRF
- Unsafe deserialization, path traversal, unsafe file uploads
- Weak crypto, insecure randomness, missing TLS verification
- Dependency vulnerabilities and supply-chain risk (typosquatting, unpinned deps)

## Subagent Model

Spawn one focused subagent per audit unit from the plan, plus always-on cross-cutting subagents:

- **Unit subagents** — one per unit, auditing it against its assigned packs.
- **Security & Secrets subagent** — scans the whole repo for the Security pack, including git history and CI config.
- **Dependencies & Supply Chain subagent** — lockfiles, outdated/vulnerable/duplicate deps, license conflicts, unpinned CI actions.
- **Architecture & Cross-Unit subagent** — boundaries, coupling, shared code, consistency of patterns across units.

Run independent subagents in parallel; keep concurrency reasonable and let the user know how many you are launching. Each subagent must follow the workflow below and cite real files/lines.

## Workflow for Each Subagent

1. **Exploration**
   Read the audit plan first, then list directories, search code, and read key files. Stay within the unit's scope; flag cross-unit findings to the architecture subagent.

2. **Report Creation**
   File: `report/<slug>-report.md`
   Structure:
   - Executive Summary
   - Major Issues (with `path:line`, severity, code snippets)
   - Minor Issues & Smells
   - Positive Observations

3. **Fix Specification**
   File: `report/<slug>-fix.md`
   For every finding, provide **concrete, actionable steps**.

   > **Issue**: CTA button logic duplicated in `Home.tsx` and `Checkout.tsx`
   >
   > **Fix**:
   > - Create `components/common/CTAButton.tsx` with props: `variant`, `size`, `loading`, `onClick`.
   > - Extract shared behavior into `hooks/useCTA.ts` if needed.
   > - Replace both instances and update tests.
   > - Benefit: Single source of truth, consistent UX, easier maintenance.

4. **Final Master Summary**
   After all subagents complete, create `report/summary.md` with:
   - Detected units and stacks (link the plan)
   - Cross-unit findings
   - High-priority recommendations (ranked)
   - Overall architecture suggestions
   - Known blind spots / areas not audited

## Quality Bar

- **Evidence over opinion**: every finding cites a real `path:line`. Verify before reporting; if you cannot confirm it, drop it or mark it as a question.
- **Severity**: use Critical / High / Medium / Low and say why.
- **No false positives**: read enough context to understand intent before calling something a bug or dark pattern.
- **Scoped effort**: don't dump every nit; prioritize high-impact issues, but list minor smells briefly.
- **Self-documenting code**: prefer clarity, composition, reusability, and clear separation of concerns.

## Non-Goals

- Do not audit `node_modules`, vendored deps, generated code, or build artifacts.
- Do not assume any layer exists; absence of web/mobile/API is expected and fine.
- Do not rewrite the code as part of the audit; produce reports and fix specs only.

Start by exploring the project structure, writing `report/00-audit-plan.md`, and confirming scope when it is ambiguous. Then activate the subagents implied by the plan.
