# famly/zpm

Famly's fork of [yarnpkg/zpm](https://github.com/yarnpkg/zpm) (Yarn 6). It exists so famly/app can
run Yarn 6 with fixes that upstream has not merged yet, from binaries built here instead of on every
developer's machine.

## Branches

- `famly` is the default branch: upstream `main` plus our patches and the two `famly-*` workflows.
  Upstream's own workflows are deleted on it: they benchmark, publish to npm and run on every push.
- `main` is upstream's branch as it was when the fork was made. Nothing updates it, and nothing
  builds it.

## Patches

Every patch we carry should be an open pull request on yarnpkg/zpm. Land it on `famly` as its own
commit. Upstream squash-merges, so once it merges the pull request, revert our commit on `famly`;
the sync would otherwise conflict on it or carry it twice.

## Workflows

- `famly-sync.yml` merges upstream `main` into `famly` every Monday at 05:00 UTC, or when you run it
  from the Actions tab. When the merge conflicts, Claude resolves it, editing only the conflicted
  files. The result lands only if no conflict is left, no other file changed and
  `cargo check --workspace --locked` passes. Claude's resolutions say so in the merge commit and in
  the release notes.
- `famly-release.yml` builds every new `famly` commit for `aarch64-apple-darwin`,
  `x86_64-unknown-linux-musl` and `aarch64-unknown-linux-musl`. It publishes a release named
  `famly-<commit date>-<sha12>` with the archives and `SHA256SUMS`.

The merge and `cargo check` run upstream's code and crates, so they run in a job with a read-only
token. A second job, which runs nothing from the repository, pushes the result.

To try the sync without touching `famly`, run it from the Actions tab with `branch` set to a branch
cut from `famly`. It pushes the merge there and releases nothing.

A failed sync shows up as a failed run in the Actions tab, and GitHub emails whoever last changed
the schedule.

## Secrets

Two repository secrets, neither exposed to pull requests from forks:

- `ANTHROPIC_API_KEY_ZPM_SYNC`: a key for this sync alone, with a spending limit. Used only when a
  merge conflicts.
- `ZPM_SYNC_TOKEN`: a fine-grained token for famly/zpm alone, with Contents and Workflows write.
  `GITHUB_TOKEN` cannot push upstream commits that change a workflow file; without this token, those
  weeks fail at the push.

## famly/app

`scripts/yarn6-trial.sh` downloads the release pinned in `scripts/yarn6-build.txt` and checks it
against the checksums pinned there. `scripts/yarn6-trial.sh pin <tag>` moves the pin and warns when
Claude resolved the release's last upstream merge. A new release reaches nobody until a famly/app
pull request moves that pin.
