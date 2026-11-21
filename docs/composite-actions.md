# Composite actions

Every directory in this repo is one composite action (`action.yaml`) invoked
directly from caller workflows — nothing here listens for PR comments; the
calling workflow decides when an action runs and passes its inputs.

## Stack methods

The PR-operation actions (`land`, `rebase`, `close`, `backport`, `checkout`)
share a `method` input selecting the stack toolchain:

| `method` | Toolchain |
| -------- | --------- |
| `ghstack` | `ghstack` from the Nix registry (`nix run $registry#ghstack`) |
| `sapling-pull-request` | Sapling (`nix run $registry#sapling`) |
| `github-pull-request` | Plain `git` + `gh`; `backport` drives this path through jujutsu (`nix run $registry#jujutsu`) |

The default is `github-pull-request` and the input is required when invoking an
action directly. All methods accept a `registry` input (default `nixpkgs`) for
the tool source and a `github-token` input.

## PR operations

| Action | Purpose | Notable inputs |
| ------ | ------- | -------------- |
| `land` | Lands the PR stack | `base`, `pull-request-url` |
| `rebase` | Rebases the PR onto its base | `base`, `pull-request-url` |
| `close` | Closes the PR | `pull-request-url` |
| `backport` | Cherry-picks the PR onto a target branch | `target-branch`, `username` |
| `checkout` | Checks out a PR with method-appropriate defaults | `base-ref`, `head-ref`, `parent-pull-request-url`, plus commit-signing inputs (`sign-commits`, `gpg-private-key`, `gpg-passphrase`, `email`, `fullname`, `username`) |
| `checks` | Reads check-rollup state for a PR | `label`, `continue` |
| `run` | Dispatches a workflow | `workflow`, `ref` |
| `direnv` | Allows `.envrc` and exports its environment to the job | `cache-key`, `path` |
| `release` | Creates or updates a GitHub release and uploads artifacts | `ref`, `repo` |

`release` is draft-first: it creates the release as a draft, uploads assets,
then publishes (`gh release edit --draft=false`). GitHub treats published
releases as immutable — an asset upload after a published create 422s — so
draft-first keeps the release invisible until every asset landed.

`checkout` is the only action that configures commit signing: `sign-commits`
plus `gpg-private-key`/`gpg-passphrase` import a GPG key (via
`crazy-max/ghaction-import-gpg`) so commits it produces are signed. The other
PR-operation actions carry no signing inputs.

## Nix helpers

| Action | Purpose |
| ------ | ------- |
| `nix/setup` | Installs Nix (`cachix/install-nix-action`), configures Cachix, optionally sets up QEMU. `extra-platforms` accepts `amd64`/`arm64` and configures QEMU plus Nix `extra-platforms`; `extra-config` passes raw Nix config (e.g. `extra-system-features = kvm`). Cachix pushes run in store-scan mode (`useDaemon: false`): the daemon's post-stop upload races job teardown and silently drops multi-GB paths, so the step scans the store and waits for every path to land before it ends. |
| `nix/setup-checks-jobs` | Builds a `{ system, runner }` matrix for checks from a `systems` map |
| `nix/setup-packages-jobs` | Builds a `{ system, runner, name }` matrix for package builds |
| `nix/integration` | Builds a flake target; uploads NixOS image artifacts when present |

## Build-tool integrations

| Action | Purpose |
| ------ | ------- |
| `pnpm/integration` | Runs package-manager stages via corepack |
| `skaffold/integration` | Runs `skaffold build`/`render`, optionally with a profile |
| `skaffold/setup-profiles-jobs` | Builds matrices for Skaffold profiles (render/build) |
| `flux/flux-push` | Pushes Flux gitops updates |

## The repo's own CI

`.github/workflows/` ships three reusable workflows: `integration.yaml`
(workflow_call / workflow_dispatch / non-draft PRs), `nix.yaml`
(workflow_call, optional Cachix inputs), and `release.yaml` (push on `main`,
`release-*.*` branches, and semver tags; also workflow_call).

## Consuming

Actions are referenced by org and ref, e.g.:

```yaml
- uses: shikanime-labs/actions/nix/setup@v9
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    cachix-name: my-cache
```
