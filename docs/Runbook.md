<!-- owner: shikanime | zone: internal | purpose: consume/deploy/release -->

# Runbook

## Consume

Reference an action by path in a workflow:

```yaml
- uses: shikanime-labs/actions/land@<ref>
```

Or run a command action locally with `nix run`:

```bash
nix run "nixpkgs#ghstack" -- land "<pr-url>"
```

## Release

There is no published version; consumers pin a git ref (`@main` or a tag). Bump
the ref in the consuming repo's workflow when you need a change.

## Branch protection

`main` is protected (required review, linear history, signed commits). See
`AGENTS.md`. Land stacks with `gh stack merge`, never `gh pr merge`.
