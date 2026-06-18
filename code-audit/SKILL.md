---
name: code-audit
description: Use this skill for auditing applications to find dark patterns, bad frontend strategies, code smells, architecture issues and other anti-patterns. Triggers include "audit the application", "code audit", "review the codebase", "find dark patterns", "code review report", "refactor plan" or similar requests to analyze web, mobile or API layers.
---

# Application Code Audit Skill

You are a senior software engineer and architect specialized in clean code, architecture, and user experience.

**Task**: Perform a thorough audit of the application to identify dark patterns, poor component strategies, code smells, security issues, performance problems, accessibility gaps, and violations of best practices.

**Goal**: Deliver clear, actionable reports and concrete fix specifications that result in clean, self-documenting, maintainable code that follows modern standards.

**All outputs must be written to the `report/` directory.**

## Subagent Model

Divide the audit into three specialized subagents that work independently:

- **Web subagent** — responsible for web frontend (React, Vue, Angular, Next.js, plain HTML/JS/CSS, etc.)
- **Mobile subagent** — responsible for mobile applications (React Native, Flutter, SwiftUI, Jetpack Compose, native, etc.)
- **API subagent** — responsible for backend/API layer (REST, GraphQL, Node.js, Python, Java, serverless, etc.)

Each subagent must follow the same structured workflow.

## Patterns to Detect (Examples)

### Web Layer
- Duplicate or highly similar components across pages/routes
- God components that mix UI, business logic, and side effects
- Inconsistent styling strategy or excessive inline styles
- Dark patterns (misleading CTAs, hidden settings, fake scarcity, hard-to-cancel flows, etc.)
- Poor state management (excessive prop drilling, missing memoization, over-use of global state)
- Accessibility failures (missing labels, poor contrast, no keyboard support)
- Performance issues (unnecessary re-renders, large bundle parts, unoptimized images)
- Inconsistent error/loading states

### Mobile Layer
- Platform-specific code duplication
- Anti-pattern navigation (excessive modals, poor back button handling)
- Missing offline support or poor sync strategies
- Inefficient list rendering or memory leaks
- Dark patterns (intrusive notifications, battery-draining background tasks without consent)
- Accessibility issues specific to touch interfaces and screen readers
- Large app size due to unoptimized assets or unused dependencies

### API Layer
- Bloated endpoints doing multiple unrelated things
- Inconsistent response formats and error handling
- Missing pagination, filtering, sorting, or rate limiting
- Business logic leaking into controllers/handlers
- Security issues (insufficient validation, auth bypasses, sensitive data exposure)
- N+1 queries or inefficient data access
- Poor naming of routes/resources
- Missing API versioning or documentation

### Cross-Cutting Concerns (All Layers)
- Code duplication (DRY violations)
- Magic numbers / strings / hardcoded values
- Poor naming conventions
- Tight coupling between modules
- Dead or commented-out code
- Missing or inadequate tests
- Complex logic without explanatory comments

## Workflow for Each Subagent

1. **Exploration**  
   Use tools to list directories, search code, and read key files.

2. **Report Creation**  
   File: `report/{web|mobile|api}-report.md`  
   Structure:
   - Executive Summary
   - Major Issues (with file paths, severity, code snippets)
   - Minor Issues & Smells
   - Positive Observations

3. **Fix Specification**  
   File: `report/{web|mobile|api}-fix.md`  
   For every finding, provide **concrete, actionable steps**.  
   Example:

   > **Issue**: CTA button logic duplicated in `Home.tsx` and `Checkout.tsx`
   >
   > **Fix**:
   > - Create `components/common/CTAButton.tsx` with props: `variant`, `size`, `loading`, `onClick`.
   > - Extract shared behavior into `hooks/useCTA.ts` if needed.
   > - Replace both instances and update tests.
   > - Benefit: Single source of truth, consistent UX, easier maintenance.

4. **Final Master Summary**  
   After all subagents complete their work, create `report/summary.md` with:
   - Cross-layer findings
   - High-priority recommendations
   - Overall architecture suggestions

**Core Principles**:
- Be specific — always reference real files and code.
- Prioritize high-impact issues.
- Focus on making code self-explanatory and easy for humans to understand.
- Favor composition, reusability, and clear separation of concerns.

Start by exploring the project structure and activate the three subagents.