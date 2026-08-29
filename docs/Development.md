<!-- owner: shikanime | zone: internal | purpose: local setup and build/format loop -->

# Development

## Prerequisites

- Nix (with flakes) and `direnv` — `direnv allow` to load `.envrc`.
- A GitHub App or PAT with the permissions listed in `README.md` if you test
  against a real repo.

## Local loop

```bash
direnv allow
nix fmt            # treefmt: format YAML/Markdown/Nix
nix flake check    # lint + any module checks
```

## How to add a command

1. Add `<name>/action.yaml` as a composite action that does the work.
2. Add `command/<name>/action.yaml` that reads the comment and dispatches it.
3. Add a job to `.github/workflows/commands.yaml` matching the trigger phrase
   (e.g. `.name`).
4. Run `nix fmt`, then open a PR.

Commit style (see `AGENTS.md`): plain-text capitalized title, no
conventional-commit prefix; body labels `Design:`, `Related:`, `Closes #`.
