# MultiversX Smart Contract GitHub Actions

This repository provides reusable GitHub Actions workflows for MultiversX smart contract development and testing.

## Available Individual Workflows

### 1. Setup Environment (`setup.yml`)
Sets up the build environment with Rust, sc-meta, mx-scenario-go, and other required tools.

**Usage:**
```yaml
jobs:
  setup:
    uses: ./.github/workflows/setup.yml
    with:
      rust-toolchain: "stable"  # optional, default: "stable"
      runs-on: "ubuntu-latest"  # optional, default: "ubuntu-latest"
      rust-target: "wasm32-unknown-unknown"  # optional, default: "wasm32-unknown-unknown"
      sc-meta-version: ""  # optional, default: latest
      mx-scenario-go-version: ""  # optional, default: latest
      wasm-opt-version: "108"  # optional, default: "108"
      path-to-sc-meta: ""  # optional, path to local sc-meta
```

### 2. Wasm Tests (`wasm-tests.yml`)
Builds and tests WASM contracts, generates size reports.

**Usage:**
```yaml
jobs:
  wasm-tests:
    needs: setup  # Requires setup job to run first
    uses: ./.github/workflows/wasm-tests.yml
    with:
      rust-toolchain: "stable"  # optional
      runs-on: "ubuntu-latest"  # optional
      rust-target: "wasm32-unknown-unknown"  # optional
      enable-contracts-size-report: true  # optional, default: true
```

### 3. Interactor Tests (`interactor-tests.yml`)
Runs interactor tests with chain simulator.

**Usage:**
```yaml
jobs:
  interactor-tests:
    needs: setup  # Requires setup job to run first
    uses: ./.github/workflows/interactor-tests.yml
    with:
      rust-toolchain: "stable"  # optional
      runs-on: "ubuntu-latest"  # optional
      rust-target: "wasm32-unknown-unknown"  # optional
```

### 4. Test Coverage (`coverage.yml`)
Generates and reports test coverage.

**Usage:**
```yaml
jobs:
  test-coverage:
    needs: setup  # Requires setup job to run first
    uses: ./.github/workflows/coverage.yml
    with:
      rust-toolchain: "stable"  # optional
      runs-on: "ubuntu-latest"  # optional
      rust-target: "wasm32-unknown-unknown"  # optional
      coverage-args: "--output ./coverage.md"  # optional
```

### 5. Rust Tests (`rust-tests.yml`)
Runs standard Rust unit tests.

**Usage:**
```yaml
jobs:
  rust-tests:
    uses: ./.github/workflows/rust-tests.yml
    with:
      rust-toolchain: "stable"  # optional
      runs-on: "ubuntu-latest"  # optional
```

### 6. Clippy Check (`clippy-check.yml`)
Runs Rust linting with Clippy.

**Usage:**
```yaml
jobs:
  clippy:
    uses: ./.github/workflows/clippy-check.yml
    with:
      rust-toolchain: "stable"  # optional
      runs-on: "ubuntu-latest"  # optional
      clippy-args: "--all-targets --all-features"  # optional
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```

## Complete Workflow (`contracts.yml`)

For backwards compatibility, the complete workflow is still available and includes all jobs:

**Usage:**
```yaml
jobs:
  contracts:
    uses: ./.github/workflows/contracts.yml
    with:
      # All inputs from individual workflows are supported
      rust-toolchain: "stable"
      enable-interactor-tests: true
      enable-contracts-size-report: true
      # ... other inputs
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```

## Example: Custom Workflow with Individual Jobs

```yaml
name: Custom Smart Contract CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  setup:
    uses: ./.github/workflows/setup.yml
    with:
      rust-toolchain: "1.75.0"
      wasm-opt-version: "108"

  rust-tests:
    uses: ./.github/workflows/rust-tests.yml
    with:
      rust-toolchain: "1.75.0"

  wasm-tests:
    needs: setup
    uses: ./.github/workflows/wasm-tests.yml
    with:
      rust-toolchain: "1.75.0"
      enable-contracts-size-report: true

  clippy:
    uses: ./.github/workflows/clippy-check.yml
    with:
      rust-toolchain: "1.75.0"
      clippy-args: "--all-targets --all-features -- -D warnings"
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}

  # Only run coverage on main branch
  coverage:
    if: github.ref == 'refs/heads/main'
    needs: setup
    uses: ./.github/workflows/coverage.yml
    with:
      rust-toolchain: "1.75.0"
```

## Dependencies Between Jobs

- **Setup job** must run before: `wasm-tests`, `interactor-tests`, `test-coverage`
- **Independent jobs**: `rust-tests`, `clippy-check`

This modular approach allows you to:
- Mix and match only the jobs you need
- Use different runner types for different jobs
- Customize inputs per job
- Maintain backwards compatibility with existing workflows