# Github Actions for smart contracts

A Github Action for smart contracts which:
- builds the wasm files
- runs both the rust and go testing scenarios
- does a clippy check
- provides a report containing details about the smart contracts

## Usage

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
    uses: multiversx/mx-sc-actions/.github/workflows/contracts.yml@v2
    with:
      rust-toolchain: nightly-2022-12-08
      vmtools-version: v1.4.60
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```

This uses fixed versions of rust and vmtools.
See [contracts.yml](.github/workflows/contracts.yml) for more details on which other arguments are supported.

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

### Using a custom mxpy version

The mxpy version can be specified by providing:
```yml
pip-mxpy-args: multiversx-sdk-cli==1.2.3
```

### Installing libtinfo5

When building smart contracts written in C, on ubuntu, the libtinfo5 has to be installed as clang requires this.
This can be optionally enabled by specifying:
```yml
install-libtinfo5: true
```
Note: if using a matrix build with multiple operating systems, enable this only for ubuntu.
