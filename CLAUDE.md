# CLAUDE.md — vts-studio/.github

## Overview

This is the org-level `.github` repository for VTS Studio. It contains reusable GitHub Actions workflows shared across all repositories in the organization.

## Structure

```
.github/
  workflows/
    laravel.yml   — Reusable CI for Laravel backends (Pint, Tests, PHPStan)
    react.yml     — Reusable CI for React frontends (ESLint, TypeScript, Vitest, Build)
```

## How It Works

Repos call these workflows using `workflow_call` with custom inputs. Each workflow has sensible defaults that match our standard monorepo structure (`backend/` and `frontend/` directories).

Example caller in a repo's `.github/workflows/ci.yml`:

```yaml
jobs:
  laravel:
    uses: vts-studio/.github/.github/workflows/laravel.yml@main
    with:
      php-version: "8.3"
      test-runner: "vendor/bin/pest"
```

## Conventions

- All workflows must use `workflow_call` trigger to be reusable
- Every input must have a sensible default so repos only override what they need
- Jobs run in parallel when independent
- Services (MySQL, Redis) are only spun up in the test job, not in linting jobs
- Boolean inputs (`run-pint`, `run-tests`, etc.) allow repos to disable specific checks

## Tech Stack

- GitHub Actions (YAML)
- PHP tooling: Laravel Pint, PHPUnit/Pest, PHPStan
- Node tooling: ESLint, TypeScript, Vitest, npm/yarn

## Commands

No local commands — this repo is consumed by GitHub Actions only.

## Deployment workflows

- `deploy-static-site.yml` — builds a static front (Vite SPA or Nuxt generate) and deploys it to S3 + CloudFront. Per-environment settings come from the caller's GitHub environment variables, never from inputs, so one workflow serves every project. AWS access through GitHub OIDC when `AWS_DEPLOY_ROLE_ARN` is set, access-key secrets otherwise (transition only).
