# PR-CI-Pipeline — Developer README
## Author: Shubham Tripathi
Repository: ecom-frontend (Next.js 14 · App Router · TypeScript)
Last Updated: 2026-09-18
Audience: Frontend Engineers, Reviewers, Tech Leads, QA, DevOps

# Table of Contents
The Trigger — Ticket → PR → Pipeline

The Four Parallel Jobs

Build Stage

Lint Stage

Security Scans

Unit & Integration Tests

Quality Gate

Merge & Official CI Build

Promotion Ladder — DEV → QA → STAGING

Canary Release to Production

Reference Tables

Troubleshooting

Governance

Repository Structure

The flow, at a glance
Ticket → Branch → Code → Push → PR → Pipeline fires → Gates pass → Merge → Deploy → Live.

That's it. Let me walk through each part.

1. Before you push
Do this every time. It saves CI runs and saves you waiting.

bash
git checkout main
git pull origin main
git checkout -b feature/coupon-checkout

# write your code

npm ci
npm run lint
npx tsc --noEmit
npm test
npm run build

# only push when everything is green
git add .
git commit -m "feat(coupon): add coupon input to checkout"
git push origin feature/coupon-checkout
If it fails locally, it'll fail in CI too. Fix it before pushing.

2. Opening the PR
When you open the PR, two things happen in parallel:

A human reviewer looks at your code

The PR-CI-Pipeline starts automatically

You don't click anything. GitHub fires a webhook, Azure DevOps picks it up, and the pipeline begins in under 5 seconds.

3. The pipeline runs four jobs in parallel
Build — compiles the app (~90 sec)

Lint — checks code style (~45 sec)

Scans — six security layers (~2 min)

Tests — unit and integration (~3 min)

Total: about 7 minutes. If they ran sequentially it'd be closer to 20. Each job runs on its own fresh agent, so nothing leaks between them.

If any job fails, the PR is blocked. Fix it, push again, pipeline re-runs.

4. Build stage
Runs npm ci then npm run build. Output depends on the framework:

React (CRA) → build/

React (Vite) → dist/

Next.js (SSR) → .next/

Next.js (static export) → out/

Vue → dist/

Nuxt → .nuxt/

For the coupon feature, the build takes about 90 seconds and produces a ~48 MB .next/ folder. That gets uploaded as an artifact.

If the build fails, the pipeline stops. Usually it's a TypeScript error, a missing dep, or a bad import path.

5. Lint stage
Every file type gets its own linter:

.js .ts .tsx → ESLint + Biome

.py → pylint + flake8

.yml → yamllint

.json → jsonlint

.css .scss → csslint

.md → markdownlint

.java → checkstyle

Takes about 45 seconds. Any error blocks the merge. Warnings are logged but don't block.

Run npm run lint --fix locally to auto-fix most issues.

6. Security scans
Six layers run in parallel. Each one targets something different:

Secret scan (gitleaks) — catches hardcoded API keys

SCA (BlackDuck, Snyk) — checks dependencies for CVEs

SAST (SonarQube, Checkmarx) — finds insecure code patterns

Container scan (trivy) — checks the Docker base image

IAC scan (checkov) — checks infrastructure config

DAST (ZED Proxy) — runs against staging, finds runtime exploits

Severity gate:

Critical or High → blocks merge, must fix

Medium → warning, logged

Low → logged to dashboard

Real example from a past PR: SAST caught a SQL injection in the coupon lookup query. That's the kind of thing these scans exist for.

7. Tests
Framework-specific runners:

React → Jest + React Testing Library

Next.js → Jest + RTL + next-router-mock

Vue → Vitest + Vue Test Utils

Nuxt → Vitest + @nuxt/test-utils

E2E tests run via Cypress or Playwright.

Coverage thresholds:

Line: 80%

Branch: 70%

Function: 85%

Critical path: 100% (blocking)

Takes about 3 minutes.

8. Quality Gate
This is the decision point. All four jobs must pass:

Build ✅

Lint ✅

Scans ✅ (no Critical/High)

Tests ✅

If everything's green, the PR can merge. If not, you get a comment on the PR telling you exactly what failed.

Do not try to bypass the gate. Branch protection will stop you anyway.

