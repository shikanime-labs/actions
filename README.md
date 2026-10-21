# Actions

Composite actions backing the fleet's Nix CI: environment setup, check and
package matrices, and workflow utilities. Invoke them directly from your
workflow jobs.

## Nix Setup Action

Use the composite action to install Nix, optionally configure Cachix, and (on
Linux) enable QEMU for additional platforms.

- `extra-platforms` accepts `amd64` and `arm64` (comma-separated). It configures
  both QEMU and Nix `extra-platforms` accordingly.

```yaml
steps:
  - uses: shikanime-studio/actions/nix/setup@main
    with:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      cachix-name: my-cache
      cachix-auth-token: ${{ secrets.CACHIX_AUTH_TOKEN }}
      extra-platforms: arm64
      extra-config: |
        experimental-features = nix-command flakes
```

## Nix Matrix Actions

Use the composite actions to generate matrices for checks and packages.

```yaml
jobs:
  setup-checks-jobs:
    runs-on: ubuntu-latest
    outputs:
      continue: ${{ steps.matrix.outputs.continue }}
      matrix: ${{ steps.matrix.outputs.matrix }}
    steps:
      - id: matrix
        uses: shikanime-studio/actions/nix/setup-checks-jobs@v9
        with:
          systems: >-
            {
              "ubuntu-latest": ["x86_64-linux"],
              "ubuntu-24.04-arm": [
                "aarch64-linux",
                "armv6l-linux",
                "armv7l-linux"
              ]
            }

  checks:
    needs: [setup-checks-jobs]
    if: needs.setup-checks-jobs.outputs.continue == 'true'
    runs-on: ${{ matrix.runner }}
    strategy:
      fail-fast: false
      matrix:
        include: >-
          ${{ fromJSON(needs.setup-checks-jobs.outputs.matrix) }}
    steps:
      - run:
          nix flake check \ --accept-flake-config \ --no-pure-eval \ --system
          "${{ matrix.system }}"

  setup-packages-jobs:
    runs-on: ubuntu-latest
    outputs:
      continue: ${{ steps.matrix.outputs.continue }}
      matrix: ${{ steps.matrix.outputs.matrix }}
    steps:
      - id: matrix
        uses: shikanime-studio/actions/nix/setup-packages-jobs@v9
        with:
          systems: >-
            {
              "ubuntu-latest": ["x86_64-linux"],
              "ubuntu-24.04-arm": ["aarch64-linux"]
            }
          excludes: '["devenv-up","devenv-test"]'

  packages:
    needs: [setup-packages-jobs]
    if: needs.setup-packages-jobs.outputs.continue == 'true'
    runs-on: ${{ matrix.runner }}
    strategy:
      fail-fast: false
      matrix:
        include: >-
          ${{ fromJSON(needs.setup-packages-jobs.outputs.matrix) }}
    steps:
      - run:
          nix build \ --accept-flake-config \ --no-pure-eval \ ".#packages.${{
          matrix.system }}.${{ matrix.name }}"
```
