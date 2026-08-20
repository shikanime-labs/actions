# Actions

Comment-driven composite GitHub Actions that automate common PR operations. Add
them to workflows as shown in README, then trigger via PR comments.

**Language:** Nix + JS/TS

## Structure

- `command/` — Comment-driven actions (land, rebase, close, backport, run)
- `update/` — Flake input updates and publishing
- `cleanup/` — Post-merge branch cleanup
- `nix/` — Nix setup and matrix helper actions

## Comment Commands

- `.land` — Lands the current PR stack. Usage: `.land` or
  `.land | ghstack|slpr|ghpr`
- `.rebase` — Rebases the current PR on its base branch
- `.close` — Closes the current PR and optionally cleans up remote branches
- `.backport` — Backports the current PR onto a target branch. Usage:
  `.backport | <target-branch>`
- `.run` — Triggers a workflow dispatch. Usage: `.run | <workflow-name-or-path>`

## Nix Utilities

- `nix/setup` — Installs Nix and configures Cachix (optionally with QEMU)
- `nix/setup-checks-jobs` — Produces a matrix of `{ system, runner }` for checks
- `nix/setup-packages-jobs` — Produces a matrix of `{ system, runner, name }`
  for package builds

## Commit Style

- Plain-text capitalized title, no conventional-commit prefix
- Body with labels: `Design:`, `Related:`, `Closes #`
- Keep Markdown lines wrapped at 80 columns and run `nix fmt` before shipping

## Stack Workflow

- Install the official GitHub extension once:
  `gh extension install github/gh-stack` (requires GitHub CLI ≥ 2.0; `gh stack`
  is in public preview and may change).
- Keep one logical change per PR; split large work into a stack of PRs.
- Create a stack: `gh stack init`, then `gh stack add` for each new branch, and
  commit on the active branch. `gh stack view` lists the stack.
- Submit/update: `gh stack submit` (add `--open` to open PRs, `--auto` to skip
  prompts). Resubmit after each change to refresh titles, bodies, and branches.
- Pull down an existing stack: `gh stack checkout <PR_NUMBER>` (also accepts a
  stack number, PR URL, or branch name).
- Rebase onto updated trunk: `gh stack rebase` (cascading), then
  `gh stack submit`.
- Land a stack: `gh stack merge` (interactive) or
  `gh stack merge <PR_NUMBER> --yes --squash` to merge up to a PR.
- Never `gh pr merge` on a stacked PR — only `gh stack merge` lands stacks.
- Never force-push stack branches; `gh stack` owns the branch pointers.
