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

## Private Packages

If your project uses private Composer packages (e.g. Laravel Nova), add a `COMPOSER_AUTH` secret to your repo with your credentials.
