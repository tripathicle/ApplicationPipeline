# ApplicationPipeline
This repository contains the Application Pipeline — a unified CI/CD workflow designed to build, lint, scan, and test frontend applications across multiple frameworks including React, VueJS, Angular, Svelte, Solid.js, Qwik, and Astro.

Supported Frameworks:
React · VueJS · Angular · Svelte · Solid.js · Qwik · Astro

Pipeline Stages:

Build → npm install → npm run build → Upload artifact

Lint → jsLint / pylint / YAML linter / Java linter

Scans → Security & dependency checks

Unit Testing → Test runner + coverage report

Built for teams who want a single, reusable pipeline across multiple frontend stacks without rewriting CI config for every project.


