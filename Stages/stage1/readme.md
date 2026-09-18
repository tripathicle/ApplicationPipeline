PR-CI-Pipeline — Developer README
Author: Shubham Tripathi
Repository: ecom-frontend (Next.js 14 · App Router · TypeScript)
Last Updated: 2026-09-18
Audience: Frontend Engineers, Reviewers, Tech Leads, QA, DevOps

📑 Table of Contents
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

🔄 The Flow, at a Glance
Ticket → Branch → Code → Push → PR → Pipeline fires → Gates pass → Merge → Deploy → Live.

That's it. Let me walk through each part.

🚦 Before You Push
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
⚠️ If it fails locally, it'll fail in CI too. Fix it before pushing.

📬 Opening the PR
When you open the PR, two things happen in parallel:

A human reviewer looks at your code.

The PR-CI-Pipeline starts automatically.

You don't click anything. GitHub fires a webhook, Azure DevOps picks it up, and the pipeline begins in under 5 seconds.

🧩 The Four Parallel Jobs
The pipeline runs four jobs in parallel:

Job	Purpose	Duration
Build	Compiles the app	~90 sec
Lint	Checks code style	~45 sec
Scans	Six security layers	~2 min
Tests	Unit and integration	~3 min
⏱️ Total: about 7 minutes.
If they ran sequentially, it'd be closer to 20 minutes.

Each job runs on its own fresh agent, so nothing leaks between them.

❌ If any job fails, the PR is blocked. Fix it, push again, pipeline re-runs.

🏗️ Build Stage
Runs npm ci then npm run build. Output depends on the framework:

Framework	Output Directory
React (CRA)	build/
React (Vite)	dist/
Next.js (SSR)	.next/
Next.js (static export)	out/
Vue	dist/
Nuxt	.nuxt/
For the coupon feature, the build takes about 90 seconds and produces a ~48 MB .next/ folder. That gets uploaded as an artifact.

⚠️ If the build fails, the pipeline stops. Usually it's a TypeScript error, a missing dep, or a bad import path.

🧹 Lint Stage
Every file type gets its own linter:

File Type	Linter
.js .ts .tsx	ESLint + Biome
.py	pylint + flake8
.yml	yamllint
.json	jsonlint
.css .scss	csslint
.md	markdownlint
.java	checkstyle
⏱️ Takes about 45 seconds.
❌ Any error blocks the merge.
⚠️ Warnings are logged but don't block.

Tip: Run npm run lint --fix locally to auto-fix most issues.

🔐 Security Scans
Six layers run in parallel. Each one targets something different:

#	Scan	Purpose
1	Secret scan (gitleaks)	Catches hardcoded API keys
2	SCA (BlackDuck, Snyk)	Checks dependencies for CVEs
3	SAST (SonarQube, Checkmarx)	Finds insecure code patterns
4	Container scan (trivy)	Checks the Docker base image
5	IAC scan (checkov)	Checks infrastructure config
6	DAST (ZED Proxy)	Runs against staging, finds runtime exploits
Severity gate:

Severity	Action
🔴 Critical / High	Blocks merge — must fix
🟡 Medium	Warning, logged
🟢 Low	Logged to dashboard
💡 Real example from a past PR: SAST caught a SQL injection in the coupon lookup query. That's the kind of thing these scans exist for.

🧪 Unit & Integration Tests
Framework-specific runners:

Framework	Test Runner
React	Jest + React Testing Library
Next.js	Jest + RTL + next-router-mock
Vue	Vitest + Vue Test Utils
Nuxt	Vitest + @nuxt/test-utils
E2E tests run via Cypress or Playwright.

Coverage thresholds:

Metric	Threshold
Line	80%
Branch	70%
Function	85%
Critical path	100% (blocking)
⏱️ Takes about 3 minutes.

🚧 Quality Gate
This is the decision point. All four jobs must pass:

✅ Build

✅ Lint

✅ Scans (no Critical/High)

✅ Tests

If everything's green, the PR can merge.
If not, you get a comment on the PR telling you exactly what failed.

🛑 Do not try to bypass the gate. Branch protection will stop you anyway.

🔀 Merge & Official CI Build
(Section reserved — content to be added)

🪜 Promotion Ladder — DEV → QA → STAGING
(Section reserved — content to be added)

🐤 Canary Release to Production
(Section reserved — content to be added)

📊 Reference Tables
(Section reserved — content to be added)

🛠️ Troubleshooting
(Section reserved — content to be added)

🏛️ Governance
(Section reserved — content to be added)

📁 Repository Structure
(Section reserved — content to be added)