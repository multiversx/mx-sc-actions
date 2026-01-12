# Github Actions for smart contracts

A Github Action for smart contracts which:

- builds the wasm files
- runs both the rust and go testing scenarios
- if enabled, runs interactor tests on the Chain Simulator
- does a clippy check
- provides a report containing details about the smart contracts
- if enabled, provides a coverage report

## Usage of `contracts.yml`

### Standard build

Create a new file under `.github/workflows/actions.yml` with the following contents:

```yml
name: CI

on:
  push:
    branches:
      - main
  pull_request:

permissions:
  checks: write
  pull-requests: write

jobs:
  contracts:
    name: Contracts
    uses: multiversx/mx-sc-actions/.github/workflows/contracts.yml@vMajor.Minor.Patch
    with:
      rust-toolchain: 1.86
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```

Preferably use the latest version available from here: <https://github.com/multiversx/mx-sc-actions/releases>.

This uses a fixed version of rust.

See [contracts.yml](.github/workflows/contracts.yml) in `inputs` section for all available configuration options.

### Main branch notes

When using the action, pay attention to the branch naming under the push event and use either `main` or `master` accordingly. Using the wrong main branch name will cause the github actions build to be skipped, without displaying an error message.

### Using more than one base branch

As an alternative, when more than one branch is used as a base branch for pull requests, the following can be used instead:

```yml
on:
  push:
  pull_request:
```

Note, however, that this runs the build multiple times for each commit.

## Additional options

### Private repositories permissions

For private repositories, read-only `contents` access has to be granted as well, so the permissions would be:

```yml
permissions:
  checks: write
  contents: read
  pull-requests: write
```

### Complete workflow options

See [contracts.yml](.github/workflows/contracts.yml) in `inputs` section for all available configuration options.

**Included Jobs:**
- Setup Environment
- Build Contracts
- Wasm Tests
- Interactor Tests (if enabled)
- Contract Report (if enabled)
- Test Coverage
- Rust Tests
- Clippy Check

## Using Individual Workflows

Instead of using the complete `contracts.yml` workflow, you can compose your own CI pipeline using individual reusable workflows. This gives you more control over which jobs run and their dependencies.

### Available Individual Workflows

#### 1. Setup Environment (`setup.yml`)
Sets up the build environment with Rust, sc-meta, mx-scenario-go, and other required tools. Creates artifacts that can be used by subsequent jobs.

**Usage:**
```yaml
jobs:
  setup:
    uses: multiversx/mx-sc-actions/.github/workflows/setup.yml@vMajor.Minor.Patch
    with:
      rust-toolchain: "stable"  # optional, default: "stable"
      runs-on: "ubuntu-latest"  # optional, default: "ubuntu-latest"
      rust-target: "wasm32-unknown-unknown"  # optional, default: "wasm32-unknown-unknown"
      sc-meta-version: ""  # optional, default: latest
      mx-scenario-go-version: ""  # optional, default: latest
      wasm-opt-version: "108"  # optional, default: "108"
      path-to-sc-meta: ""  # optional, path to local sc-meta
```

#### 2. Build Contracts (`build-contracts.yml`)
Builds all WASM contracts and uploads them as artifacts.

**Usage:**
```yaml
jobs:
  build-contracts:
    needs: setup  # Requires setup job to run first
    uses: multiversx/mx-sc-actions/.github/workflows/build-contracts.yml@vMajor.Minor.Patch
    with:
      rust-toolchain: "stable"  # optional, default: "stable"
      runs-on: "ubuntu-latest"  # optional, default: "ubuntu-latest"
      rust-target: "wasm32-unknown-unknown"  # optional, default: "wasm32-unknown-unknown"
```

#### 3. Wasm Tests (`wasm-tests.yml`)
Runs WASM tests using mx-scenario-go.

**Usage:**
```yaml
jobs:
  wasm-tests:
    needs: build-contracts  # Requires build-contracts job to run first
    uses: multiversx/mx-sc-actions/.github/workflows/wasm-tests.yml@vMajor.Minor.Patch
    with:
      rust-toolchain: "stable"  # optional, default: "stable"
      runs-on: "ubuntu-latest"  # optional, default: "ubuntu-latest"
```

#### 4. Interactor Tests (`interactor-tests.yml`)
Runs interactor tests with chain simulator.

**Usage:**
```yaml
jobs:
  interactor-tests:
    needs: build-contracts  # Requires build-contracts job to run first
    uses: multiversx/mx-sc-actions/.github/workflows/interactor-tests.yml@vMajor.Minor.Patch
    with:
      rust-toolchain: "stable"  # optional, default: "stable"
      runs-on: "ubuntu-latest"  # optional, default: "ubuntu-latest"
```

#### 5. Test Coverage (`coverage.yml`)
Generates and reports test coverage.

**Usage:**
```yaml
jobs:
  test-coverage:
    needs: setup  # Requires setup job to run first
    uses: multiversx/mx-sc-actions/.github/workflows/coverage.yml@vMajor.Minor.Patch
    with:
      rust-toolchain: "stable"  # optional, default: "stable"
      runs-on: "ubuntu-latest"  # optional, default: "ubuntu-latest"
      rust-target: "wasm32-unknown-unknown"  # optional, default: "wasm32-unknown-unknown"
      coverage-args: "--output ./coverage.md"  # optional, default: "--output ./coverage.md"
```

#### 6. Rust Tests (`rust-tests.yml`)
Runs standard Rust unit tests.

**Usage:**
```yaml
jobs:
  rust-tests:
    uses: multiversx/mx-sc-actions/.github/workflows/rust-tests.yml@vMajor.Minor.Patch
    with:
      rust-toolchain: "stable"  # optional, default: "stable"
      runs-on: "ubuntu-latest"  # optional, default: "ubuntu-latest"
```

#### 7. Contract Report (`report.yml`)
Generates contract size and comparison reports.

**Usage:**
```yaml
jobs:
  generate-report:
    needs: build-contracts  # Requires build-contracts job to run first
    uses: multiversx/mx-sc-actions/.github/workflows/report.yml@vMajor.Minor.Patch
    with:
      rust-toolchain: "stable"  # optional, default: "stable"
      runs-on: "ubuntu-latest"  # optional, default: "ubuntu-latest"
```

#### 8. Proxy Compare (`proxy-compare.yml`)
Compares newly generated proxies with existing ones in the file tree.

**Usage:**
```yaml
jobs:
  proxy-compare:
    needs: setup  # Requires setup job to run first
    uses: multiversx/mx-sc-actions/.github/workflows/proxy-compare.yml@vMajor.Minor.Patch
    with:
      rust-toolchain: "stable"  # optional, default: "stable"
      runs-on: "ubuntu-latest"  # optional, default: "ubuntu-latest"
```

### Example: Custom Workflow with Individual Jobs

```yaml
name: Custom Smart Contract CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  setup:
    uses: multiversx/mx-sc-actions/.github/workflows/setup.yml@vMajor.Minor.Patch
    with:
      rust-toolchain: "1.86"
      wasm-opt-version: "108"

  build-contracts:
    needs: setup
    uses: multiversx/mx-sc-actions/.github/workflows/build-contracts.yml@vMajor.Minor.Patch
    with:
      rust-toolchain: "1.86"

  rust-tests:
    uses: multiversx/mx-sc-actions/.github/workflows/rust-tests.yml@vMajor.Minor.Patch
    with:
      rust-toolchain: "1.86"

  wasm-tests:
    needs: build-contracts
    uses: multiversx/mx-sc-actions/.github/workflows/wasm-tests.yml@vMajor.Minor.Patch
    with:
      rust-toolchain: "1.86"

  generate-report:
    needs: build-contracts
    uses: multiversx/mx-sc-actions/.github/workflows/report.yml@vMajor.Minor.Patch
    with:
      rust-toolchain: "1.86"

  # Only run coverage on main branch
  coverage:
    if: github.ref == 'refs/heads/main'
    needs: setup
    uses: multiversx/mx-sc-actions/.github/workflows/coverage.yml@vMajor.Minor.Patch
    with:
      rust-toolchain: "1.86"
```

### Dependencies Between Jobs

The following dependencies exist between jobs:

**Setup artifacts required by:**
- `build-contracts.yml` - needs sc-meta artifact
- `wasm-tests.yml` - needs sc-meta and mx-scenario-go artifacts
- `interactor-tests.yml` - needs sc-meta artifact
- `coverage.yml` - needs sc-meta artifact
- `report.yml` - needs sc-meta artifact
- `proxy-compare.yml` - needs sc-meta artifact

**Built contracts required by:**
- `wasm-tests.yml` - needs built contracts from build-contracts
- `interactor-tests.yml` - needs built contracts from build-contracts
- `report.yml` - needs built contracts from build-contracts

**Independent jobs:**
- `rust-tests.yml` - no dependencies
- `reproducible-build.yml` - no dependencies (uses Docker)

### Typical Workflow Structure

```yaml
jobs:
  # Phase 1: Setup (runs first)
  setup:
    uses: multiversx/mx-sc-actions/.github/workflows/setup.yml@vMajor.Minor.Patch

  # Phase 2: Build (depends on setup)
  build-contracts:
    needs: setup
    uses: multiversx/mx-sc-actions/.github/workflows/build-contracts.yml@vMajor.Minor.Patch

  # Phase 3: Tests and Reports (run in parallel, depend on build)
  wasm-tests:
    needs: build-contracts
    uses: multiversx/mx-sc-actions/.github/workflows/wasm-tests.yml@vMajor.Minor.Patch

  interactor-tests:
    needs: build-contracts
    uses: multiversx/mx-sc-actions/.github/workflows/interactor-tests.yml@vMajor.Minor.Patch

  generate-report:
    needs: build-contracts
    uses: multiversx/mx-sc-actions/.github/workflows/report.yml@vMajor.Minor.Patch

  # Phase 3b: Coverage (depends on setup only)
  coverage:
    needs: setup
    uses: multiversx/mx-sc-actions/.github/workflows/coverage.yml@vMajor.Minor.Patch

  # Phase 3c: Independent tests (no dependencies)
  rust-tests:
    uses: multiversx/mx-sc-actions/.github/workflows/rust-tests.yml@vMajor.Minor.Patch
```

This modular approach allows you to:
- Mix and match only the jobs you need
- Use different runner types for different jobs (e.g., self-hosted for some jobs)
- Customize inputs per job
- Run jobs in parallel when dependencies allow
- Maintain backwards compatibility with the complete `contracts.yml` workflow

## Usage of `reproducible-build.yml`

See [reproducible-build.yml](.github/workflows/reproducible-build.yml).

## Configuration entries

The following configuration entries are available:

- `image_tag`: the desired Docker image tag to be used for the reproducible contract build. The available tags are listed [here](https://hub.docker.com/r/multiversx/sdk-rust-contract-builder/tags).
- `project_path`: the path to the project (workspace) containing the contracts to build. If not specified, the repository root folder is used.
- `contract_name`: a specific contract to be built. If not specified, all contracts in the workspace (repository) are built.
- `create_release`: whether to create a new release (and upload the build artifacts as assets).
- `attach_to_existing_release`: whether to upload the build artifacts on an existing release. This only works if the current `github.ref_name` (of the executing workflow) is associated with an existing release.
- `skip_preliminary_checks`: whether to skip the preliminary checks. **Never set this in production!**
- `package_whole_project_src`: whether to include all project files in the packaged source (`*.source.json`).

Note that `create_release` and `attach_to_existing_release` are mutually exclusive.

## Creating a release from scratch

At times, you might want to create a release directly from a Github Workflow. In order to do so, follow this example:

```yml
name: Create release, build contracts, upload assets

on:
  workflow_dispatch:

permissions:
  contents: write

jobs:
  build:
    uses: multiversx/mx-sc-actions/.github/workflows/reproducible-build.yml@v2.2.1
    with:
      image_tag: v1.2.3 # this is an example; see above
      create_release: true
```

### Building on an existing release

In order to configure your workflow for building the contracts and uploading the output on a newly created (published) release, do as follows:

```yml
name: On new release, build contracts, upload assets

permissions:
  contents: write

on:
  release:
    types: [published]

jobs:
  build:
    uses: multiversx/mx-sc-actions/.github/workflows/reproducible-build.yml@v2.2.1
    with:
      image_tag: v1.2.3 # this is an example; see above
      attach_to_existing_release: true
```

### Running reproducible builds on pull requests

In order to run the reproducible builds on a pull request, without creating or editing a GitHub release, do as follows:

```yml
name: Build contracts

on:
  pull_request:

permissions:
  contents: write

jobs:
  build:
    uses: multiversx/mx-sc-actions/.github/workflows/reproducible-build.yml@v2.2.1
    with:
      image_tag: v1.2.3 # this is an example; see above
```

Once the workflow finishes, the build artifacts will be found as workflow artifacts.

Now, let's select a single contract to be built:

```yml
name: Build contracts

on:
  pull_request:

permissions:
  contents: write

jobs:
  build:
    uses: multiversx/mx-sc-actions/.github/workflows/reproducible-build.yml@v2.2.1
    with:
      image_tag: v1.2.3 # this is an example; see above
      contract_name: adder
```
