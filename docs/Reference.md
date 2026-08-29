<!-- owner: shikanime | zone: internal | purpose: command + input surface -->

# Reference

## Comment commands

| Command     | Effect                          | Args                          |
| ----------- | ------------------------------- | ----------------------------- |
| `.land`     | Land the current PR stack       | `ghstack` \| `slpr` \| `ghpr` |
| `.rebase`   | Rebase PR onto its base         | —                             |
| `.close`    | Close PR, clean remote branches | —                             |
| `.backport` | Backport onto a target branch   | `<target-branch>`             |
| `.run`      | Trigger a workflow dispatch     | `<workflow-name-or-path>`     |

## Key inputs (composite actions)

- `base` — base branch to rebase/land onto (default `main`).
- `github-token` — token for API calls (default `github.token`).
- `method` — `ghstack` \| `sapling-pull-request` \| `github-pull-request`.
- `pull-request-url` — the PR to act on.
- `registry` — Nix registry for `nix run` (default `nixpkgs`).

## Nix utilities

- `nix/setup` — install Nix + Cachix (optional QEMU).
- `nix/setup-checks-jobs` — `{ system, runner }` matrix.
- `nix/setup-packages-jobs` — `{ system, runner, name }` matrix.
