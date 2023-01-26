# Github Actions for smart contracts

A Github Action for smart contracts which:
- builds the wasm files
- runs mandos-rs and mandos-go tests
- does a clippy check
- provides a report containing details about the smart contracts

## Usage of `contracts.yml`

### Standard build

See [contracts.yml](.github/workflows/contracts.yml)
This uses fixed versions of rust and vmtools.
Ignores `eei` checks which allows the contracts to use features which are not live on the elrond mainnet yet.

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
    uses: ElrondNetwork/elrond-actions/.github/workflows/contracts.yml@v1
    with:
      rust-toolchain: nightly-2022-01-17
      vmtools-version: v1.4.43
      extra-build-args: --ignore-eei-checks
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```

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

### Private repositories permissions

For private repositories, read-only `contents` access has to be granted as well, so the permissions would be:
```yml
permissions:
  checks: write
  contents: read
  pull-requests: write
```

## Additional options

### Using a custom erdpy version

The erdpy version can be specified by providing:
```yml
pip-erdpy-args: erdpy==1.2.3
```

### Installing libtinfo5

When building smart contracts written in C, on ubuntu, the libtinfo5 has to be installed as clang requires this.
This can be optionally enabled by specifying:
```yml
install-libtinfo5: true
```
Note: if using a matrix build with multiple operating systems, enable this only for ubuntu.

## Usage of `reproducible-build.yml`

See [contracts.yml](.github/workflows/reproducible-build.yml).

## Configuration entries

The following configuration entries are available:

 - `image_tag`: the desired Docker image tag to be used for the reproducible contract build. The available tags are listed [here](https://hub.docker.com/r/multiversx/sdk-rust-contract-builder/tags).
 - `contract_name`: a specific contract to be built. If not specified, all contracts in the workspace (repository) are built.
 - `create_release`: whether to create a new release (and upload the build artifacts as assets).
 - `attach_to_existing_release`: whether to upload the build artifacts on an existing release. This only works if the current `github.ref_name` (of the executing workflow) is associated with an existing release.

Note that `create_release` and `attach_to_existing_release` are mutually exclusive.

## Creating a release from scratch

At times, you might want to create a release directly from a Github Workflow. In order to do so, follow this example:

```
name: Create release, build contracts, upload assets

on:
  release:
    types: [published]

jobs:
  build:
    uses: multiversx/mx-sc-actions/.github/workflows/reproducible-build.yml@v2.1.0
    with:
      image_tag: v4.1.0
      create_release: true
```

### Building on an existing release

In order to configure your workflow for building the contracts and uploading the output on a newly created (published) release, do as follows:

```
name: On new release, build contracts, upload assets

on:
  release:
    types: [published]

jobs:
  build:
    uses: multiversx/mx-sc-actions/.github/workflows/reproducible-build.yml@v2.1.0
    with:
      image_tag: v4.1.0
      attach_to_release: true
```

### Running reproducible builds on pull requests

In order to run the reproducible builds on a pull request, without creating or editing a GitHub release, do as follows:

```
name: Build contracts

on:
  release:
    types: [published]

jobs:
  build:
    uses: multiversx/mx-sc-actions/.github/workflows/reproducible-build.yml@v2.1.0
    with:
      image_tag: v4.1.0
```

Once the workflow finishes, the build artifacts will be found as workflow artifacts.

Now, let's select a single contract to be built:

```
name: Build contracts

on:
  release:
    types: [published]

jobs:
  build:
    uses: multiversx/mx-sc-actions/.github/workflows/reproducible-build.yml@v2.1.0
    with:
      image_tag: v4.1.0
      contract_name: adder
```
