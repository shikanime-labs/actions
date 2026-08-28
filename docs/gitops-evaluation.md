# GitOps-style CI evaluation

Evaluation of pull-based, self-hosted CI systems to move orchestration logic
out of GitHub Actions (issue #270).

## Criteria

- Truly OSS, no commercial incentive, no vendor lead-gen.
- Self-hosted.
- Pull-based (agents/controllers poll or reconcile, not a cloud push model).

License and hosting model below are verified from primary source (repo LICENSE
or forge docs), dated 2026-08-28.

## Logic currently embedded in GitHub Actions

| Path | Logic |
| ---- | ----- |
| `command/land` | `.land`/`.force-land` dispatch, permission gate, reaction |
| `command/rebase` | `.rebase` dispatch, permission gate, reaction |
| `command/close` | `.close` dispatch, permission gate, reaction |
| `command/backport` | `.backport` dispatch, target-branch param, reaction |
| `command/run` | `.run` dispatch, workflow ref param |
| command/* (shared) | PR-URL parse, failure comment with run-log link |
| `cleanup/` | Post-close branch deletion; ghstack 3-branch cleanup |
| `nix/setup-checks-jobs` | `nix flake show` → system matrix, `continue` |
| `nix/setup-packages-jobs` | Same, for packages, with excludes |
| `skaffold/setup-profiles-jobs` | `yq` read of skaffold profiles → matrix |

Two distinct slices: (1) matrix computation (portable, pure logic) and (2)
comment-command orchestration (GitHub-forge-specific dispatch).

## Candidates

### Woodpecker CI — recommended

- License: Apache-2.0 (github.com/woodpecker-ci/woodpecker, LICENSE via API).
- Activity: pushed 2026-08-27, ~7.8k stars, not archived.
- Hosting: self-hosted server; agents poll the server for jobs (pull model).
- Governance: community-run, no commercial company (unlike its Drone lineage).
- Pipelines live in repo `.woodpecker/*.yml`, owned by the repo — fits moving
  logic out of the Actions runtime.

### Gitea Actions

- License: MIT (github.com/go-gitea/gitea, LICENSE via API); runner at
  gitea.com/gitea/runner (off GitHub).
- Activity: pushed 2026-08-28, ~57.6k stars.
- Hosting: self-hosted `act_runner` agents poll for jobs; GitHub-Actions-
  compatible YAML.
- Cost: requires migrating the forge to Gitea. Large lift for a GitHub-homed
  org; kept only if a forge move is already on the table.

### Zuul

- License: Apache-2.0 (opendev.org/zuul/zuul, LICENSE).
- Hosting: self-hosted gating CI; genuinely pull/event-driven with shared
  queues; reports back to GitHub. Operated by OpenInfra.
- Strongest philosophical fit for a reconciling controller, but heavier to run
  (ZooKeeper, scheduler, merger components). The comment-command layer maps
  most cleanly here.

### Buildbot

- License: GPL-2.0 (github.com/buildbot/buildbot, LICENSE via API).
- Hosting: self-hosted; pull-based ChangeSources; Python framework.
- Mature and capable, but GPL conflicts with the preferred Apache-2.0/MIT
  posture. Lower fit.

### Disqualified: Drone

- Apache-2.0, but the repo now reads "Harness Open Source ... Source Control
  Management, CI" — a commercial company owns it. Violates the no-commercial-
  incentive criterion despite the permissive license. The Woodpecker fork
  preserves the original community model.

### Also self-hosted (Kubernetes-native, not evaluated in depth)

Argo Workflows, Tekton, Concourse — all Apache-2.0 and active, but assume a
Kubernetes control plane the current fleet does not require for this slice.

## Recommendation

Woodpecker CI. It is the direct, Apache-2.0, genuinely community-owned
successor to Drone, agents pull jobs (a true pull model), and repo-owned
`.woodpecker` pipelines move the matrix logic out of GitHub Actions.

Scope honesty: the matrix slice (nix checks/packages, skaffold) ports cleanly
to Woodpecker. The comment-command slice (land, rebase, close, backport) is
GitHub-forge-specific — Woodpecker does not dispatch on PR comments, so that
layer either stays on GitHub Actions or becomes a small controller; Zuul is the
stronger model for that slice if a fuller migration is wanted.
