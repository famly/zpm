# famly/zpm

Famly's fork of [yarnpkg/zpm](https://github.com/yarnpkg/zpm) (Yarn 6). It exists so famly/app can
run Yarn 6 with fixes that upstream has not merged yet, from binaries built here instead of on every
developer's machine.

## Branches

- `famly` is the default branch: upstream `main` plus our patches and the two `famly-*` workflows.
- `main` is upstream's branch as it was when the fork was made. Nothing updates it, and nothing
  builds it.

## Patches

Every patch we carry should be an open pull request on yarnpkg/zpm. Land it on `famly` as its own
commit, and drop nothing by hand: once upstream merges the pull request, the next sync merges the
same change, and the patch stops showing up in the release notes.

## Workflows

- `famly-sync.yml` merges upstream `main` into `famly` every Monday at 05:00 UTC, or when you run it
  from the Actions tab. When the merge conflicts, Claude resolves it. The result lands only if no
  conflict is left, famly's own files are unchanged and `cargo check --workspace --locked` passes.
  Claude's resolutions say so in the merge commit and in the release notes.
- `famly-release.yml` builds every new `famly` commit for `aarch64-apple-darwin`,
  `x86_64-unknown-linux-musl` and `aarch64-unknown-linux-musl`. It publishes a release named
  `famly-<commit date>-<sha12>` with the archives and `SHA256SUMS`.
- The sync turns off every other workflow on each run. Upstream's workflows benchmark, publish to
  npm and run on every push, and none of that belongs in the fork.

A failed sync shows up as a failed run in the Actions tab, and GitHub emails whoever last changed
the schedule.

## Secrets

The sync needs two org secrets granted to this repository:

- `ANTHROPIC_API_KEY`, used only when a merge conflicts.
- `FAMLYDEV_GITHUB_TOKEN`, used only when upstream changed a workflow file, which `GITHUB_TOKEN` may
  not push.

Neither is exposed to pull requests from forks.

## famly/app

`scripts/yarn6-trial.sh` downloads the release pinned in `scripts/yarn6-build.txt` and checks it
against the checksums pinned there. `scripts/yarn6-trial.sh pin <tag>` moves the pin. A new release
reaches nobody until a famly/app pull request moves that pin.
