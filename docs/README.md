<!-- owner: Automata | zone: technical | purpose: Canonical technical doc index for shikanime-labs/actions -->
# Actions — technical index

Composite GitHub Actions backing the fleet's Nix CI and PR workflow automation. Every top-level directory (and the `nix/`, `pnpm/`, `skaffold/`, `flux/` subtrees) holds one `action.yaml` consumed by caller workflows.

## Canonical sources

- [Repository README](https://github.com/shikanime-labs/actions#readme)
- [Technical wiki](https://github.com/shikanime-labs/actions/wiki)

## Subsystems

### Nix environment setup

- `nix/setup` — installs Nix, configures Cachix, and optionally enables QEMU via `extra-platforms` (`amd64`, `arm64`). Cachix pushes run in store-scan mode so multi-GB paths are not silently dropped at job teardown. No system feature is hardcoded: builds needing KVM pass `extra-system-features = kvm` through `extra-config`.

### Matrix generation

- `nix/setup-checks-jobs` — emits a `{ system, runner }` matrix for `nix flake check` jobs.
- `nix/setup-packages-jobs` — emits a `{ system, runner, name }` matrix for `nix build` jobs; `excludes` drops named packages.

### PR operations

Comment-driven actions for stack/VCS operations, built around ghstack, slpr (Sapling), and jujutsu:

- `land` — lands the current PR stack through the ghstack, slpr, or ghpr backend.
- `rebase` — rebases a PR; accepts `gpg-passphrase`, `gpg-private-key`, `sign-commits`, `email`, `fullname`, `username` so rebased commits stay signed.
- `close`, `backport`, `checkout`, `checks`, `release`, `run` — standalone wrappers reusable outside the comment flow.

`release` creates the GitHub release as a draft, uploads assets, then publishes it (`gh release edit --draft=false`): GitHub treats published releases as immutable, so uploading an asset after publishing 422s. Required permissions are declared per operation (`contents`, `pull-requests`, `issues`: write) and requested through a GitHub App token via `actions/create-github-app-token`.

### Build and deploy helpers

`nix/integration`, `pnpm/integration`, `skaffold/integration`, `skaffold/setup-profiles-jobs`, and `flux/flux-push` integrate builds and deployments.

### Repository CI

`.github/workflows/` holds the `integration`, `nix`, and `release` workflows; the flake and pnpm lockfile drive the toolchain.

## Documentation rule

In-repo `docs/` is canonical for technical detail. The [wiki](https://github.com/shikanime-labs/actions/wiki) is a rendered human index that links here.
