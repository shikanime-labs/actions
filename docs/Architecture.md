<!-- owner: shikanime | zone: internal | purpose: module and data-flow design -->

# Architecture

Actions is a library of reusable GitHub Actions, not an application. Two kinds
of actions live side by side:

- `command/<name>/` — thin comment handlers. Each contains an `action.yaml`
  whose only job is to parse the PR-comment body and dispatch the matching
  composite action.
- `<name>/action.yaml` (e.g. `land/`, `rebase/`, `close/`, `backport/`,
  `cleanup/`) — composite actions that do the real work: call
  `nix run ...#ghstack`, `sapling`, or `gh` to land/rebase/close/backport a PR.

A workflow in `.github/workflows/` (typically `commands.yaml`) listens on
`issue_comment` and selects a job by matching the comment body (`.land`,
`.rebase`, `.close`, `.backport <branch>`, `.run <wf>`). Each job requests a
GitHub App token via `actions/create-github-app-token`, checks the repo out with
`fetch-depth: 0`, then calls the composite action.

Nix glue lives under `nix/`: `setup` installs Nix + Cachix (optional QEMU),
`setup-checks-jobs` and `setup-packages-jobs` emit CI matrices.

## Design intent

Keep the comment surface tiny and stable; put all behaviour in composite actions
so the same logic can run from CI or from a manual `nix run`.
