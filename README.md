# VTS Studio — Shared CI & Org Config

Reusable GitHub Actions workflows and org-level configuration for all VTS Studio repositories.

## Reusable Workflows

### Laravel CI

**Usage** — add this to your repo's `.github/workflows/ci.yml`:

```yaml
jobs:
  laravel:
    uses: vts-studio/.github/.github/workflows/laravel.yml@main
    with:
      php-version: "8.3"
      working-directory: "backend"
      test-runner: "vendor/bin/pest"
```

**Jobs**: Pint (code style), Tests (PHPUnit/Pest with MySQL + Redis), PHPStan (static analysis, opt-in)

| Input | Default | Description |
|---|---|---|
| `php-version` | `8.3` | PHP version |
| `working-directory` | `backend` | Path to Laravel project |
| `test-runner` | `vendor/bin/pest` | Test command (`vendor/bin/pest` or `vendor/bin/phpunit`) |
| `run-pint` | `true` | Run Laravel Pint check |
| `run-tests` | `true` | Run test suite |
| `run-phpstan` | `false` | Run PHPStan static analysis |
| `phpstan-level` | `5` | PHPStan analysis level (0-9) |

### React CI

**Usage**:

```yaml
jobs:
  react:
    uses: vts-studio/.github/.github/workflows/react.yml@main
    with:
      node-version: "20"
      working-directory: "frontend"
```

**Jobs**: ESLint, TypeScript check, Vitest, Production build

| Input | Default | Description |
|---|---|---|
| `node-version` | `20` | Node.js version |
| `working-directory` | `frontend` | Path to React project |
| `run-lint` | `true` | Run ESLint |
| `run-typecheck` | `true` | Run TypeScript check |
| `run-tests` | `true` | Run Vitest |
| `run-build` | `true` | Run production build |
| `package-manager` | `npm` | Package manager (`npm` or `yarn`) |

### React Native CI

**Usage**:

```yaml
jobs:
  mobile:
    uses: vts-studio/.github/.github/workflows/react-native.yml@main
    with:
      node-version: "20"
      working-directory: "mobile"
```

**Jobs**: ESLint, TypeScript check, Jest tests

| Input | Default | Description |
|---|---|---|
| `node-version` | `20` | Node.js version |
| `working-directory` | `mobile` | Path to React Native project |
| `run-lint` | `true` | Run ESLint |
| `run-typecheck` | `true` | Run TypeScript check |
| `run-tests` | `true` | Run Jest tests |
| `package-manager` | `npm` | Package manager (`npm` or `yarn`) |

## Full Example (Monorepo)

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  laravel:
    uses: vts-studio/.github/.github/workflows/laravel.yml@main
    with:
      php-version: "8.5"
      test-runner: "vendor/bin/pest"

  react:
    uses: vts-studio/.github/.github/workflows/react.yml@main
    with:
      node-version: "20"

  mobile:
    uses: vts-studio/.github/.github/workflows/react-native.yml@main
```

## Deployment Workflows

### Deploy static site (S3 + CloudFront)

Builds a static front — single-page app (Vite) or generated site (Nuxt generate) — and ships it to a private S3 bucket served by CloudFront. Files under the hashed-assets directory are cached for a year; every other file is revalidated on each visit, then CloudFront is invalidated.

**Usage** — `.github/workflows/deploy-frontend.yml` in the project:

```yaml
name: Deploy frontend

on:
  push:
    branches: [deploy-staging, deploy-production]
  workflow_dispatch:

concurrency:
  group: deploy-frontend-${{ github.ref_name }}
  cancel-in-progress: false

jobs:
  frontend:
    uses: vts-studio/.github/.github/workflows/deploy-static-site.yml@main
    permissions:
      contents: read
      id-token: write
    secrets: inherit # only while the project still uses AWS access keys
    with:
      environment: ${{ github.ref_name == 'deploy-production' && 'production' || 'staging' }}
```

| Input | Default | Description |
|---|---|---|
| `environment` | — (required) | GitHub environment holding the deployment variables |
| `working-directory` | `frontend` | Path to the front project (`.` at the repository root) |
| `node-version` | `24` | Node.js version |
| `package-manager` | `npm` | `npm` or `yarn` (dependency cache) |
| `install-command` | `npm ci` | Dependency installation |
| `build-command` | `npm run build` | Build (`yarn generate` for Nuxt) |
| `output-directory` | `dist` | Build output, relative to `working-directory` |
| `env-prefix` | `VITE_` | Environment variables with this prefix are passed to the build (empty = none) |
| `build-variables` | — | Space-separated names of other environment variables passed to the build |
| `hashed-assets-path` | `assets` | Directory of content-hashed files (`_nuxt` for Nuxt) |
| `well-known-content-type` | — | Content-Type forced on `.well-known/` files, e.g. `application/json` for iOS/Android app links |

**Variables of each GitHub environment** (Settings → Environments): `S3_BUCKET`, `CLOUDFRONT_DISTRIBUTION_ID`, optionally `AWS_REGION` (default `eu-west-3`) and `FRONTEND_URL`, plus the build variables.

**AWS access**:
- **Recommended — GitHub OIDC**: set `AWS_DEPLOY_ROLE_ARN` on the environment. The IAM role trusts `repo:vts-studio/<repo>:environment:<environment>` in its `token.actions.githubusercontent.com:sub` condition, and only needs `s3:ListBucket`, `s3:GetObject`, `s3:PutObject`, `s3:DeleteObject` on the bucket and `cloudfront:CreateInvalidation` on the distribution. No AWS key is stored in GitHub.
- **Transition — access keys**: without `AWS_DEPLOY_ROLE_ARN`, the `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` secrets are used (pass `secrets: inherit`). Moving to OIDC later = set the variable, then delete the secrets and the IAM user's keys.

## Private Packages

If your project uses private Composer packages (e.g. Laravel Nova), add a `COMPOSER_AUTH` secret to your repo with your credentials.
