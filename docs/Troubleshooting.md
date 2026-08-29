<!-- owner: shikanime | zone: internal | purpose: known failure modes -->

# Troubleshooting

## Comment command does nothing

The workflow only fires on `issue_comment` of type `created` on a PR. A comment
edited later, or on an issue (not a PR), is ignored. Ensure the phrase (`.land`,
`.backport <branch>`, …) is an exact substring.

## Token lacks permission

The App token must carry `contents: write`, `pull-requests: write`, and
`issues: write` (see `README.md`). Missing scopes surface as 403 / "resource not
accessible".

## Shallow checkout

`actions/checkout` must use `fetch-depth: 0`; stacks need full history to
rebase. A default shallow checkout breaks `.rebase` / `.land`.

## Bookmark conflict in a worktree

If `jj bookmark list` shows a conflicted bookmark, run
`jj bookmark set <name> -r <rev>` to resolve before pushing.
