# AGENTS.md — ActionsCI/compute-semver

This file is for AI agents working in a repository that consumes the
`ActionsCI/compute-semver` GitHub Action. Read it before generating workflow
YAML that calls this action.

## Purpose

`ActionsCI/compute-semver` is a composite GitHub Action (action name
`Compute Version`, defined in `action.yaml` at the repo root) that computes a
single output string `version` from the current branch name and the
repository's git tags. It is intended for trunk-based CI/CD pipelines that
need a deterministic SemVer-shaped string to pass to downstream build, image
tag, and release steps. It is NOT a tag pusher — it never creates, pushes, or
mutates git refs in the caller repo. It is also NOT a changelog generator,
release-notes builder, or version-bump committer; callers needing those must
use a separate action.

## Calling this action — minimal example

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0          # required: action reads git tags

      - id: compute_version
        uses: ActionsCI/compute-semver@main

      - run: echo "version=${{ steps.compute_version.outputs.version }}"
```

Pin to a tag (for example `@v1.0.0`) rather than `@main` in production
callers; `main` will float.

## Inputs reference

| Input              | Required | Default | Purpose |
|--------------------|----------|---------|---------|
| `major_version`    | No       | `''`    | Pins the major component. Must be set together with `minor_version`; if only one is set the override is ignored and the action falls back to repo-wide tag scan. Typically sourced from `cicd.yaml` `config.version.major`. |
| `minor_version`    | No       | `''`    | Pins the minor component. Must be set together with `major_version`. Typically sourced from `cicd.yaml` `config.version.minor`. |
| `release_branch`   | No       | `main`  | The branch name that produces a clean SemVer with no postfix. Set this if your repo's trunk is not `main` (for example `master` or `trunk`); otherwise builds on your trunk will receive a `-<branch>-<sha>` postfix. |
| `test_version_tag` | No       | `''`    | Test-only override that forces the "latest version" lookup to a literal value. Setting this in production yields wrong versions. |
| `test_branch_name` | No       | `''`    | Test-only override that forces the branch name detection. Setting this in production yields wrong versions. |

## Outputs reference

| Output    | Description |
|-----------|-------------|
| `version` | Computed SemVer-shaped string. On the configured `release_branch` and on branches matching `release*` it is a clean `MAJOR.MINOR.PATCH`. On `feature/*`, `bugfix/*`, `alpha/*`, `beta/*` it is `MAJOR.MINOR.PATCH-<prefix>[JIRAID]-<shortSHA>`. On `develop*` it is `MAJOR.MINOR.PATCH-develop-<shortSHA>`. On any other branch it is `MAJOR.MINOR.PATCH-<branch-segment-before-slash>-<shortSHA>`. |

## Secrets and permissions required

| Item | Required by | Why |
|------|-------------|-----|
| `GITHUB_TOKEN` with `contents: read` | The `actions/checkout` step in the caller job | The action reads tags via `git fetch --tags` and `git tag --list` against the local clone; checkout must populate that clone with full history (`fetch-depth: 0`). |

The action itself declares no `secrets:` and performs no network calls beyond
`git fetch --tags` against the origin already configured by `actions/checkout`.
It does not log into ECR, push to gitops, or call external APIs. Callers do
not need to declare any secrets specifically for this action and do not need
to use `secrets: inherit`.

## What the action does internally

The composite action runs a single bash step (`id: compute-version`) that:

- Resolves the branch name from `test_branch_name` if set, otherwise from
  `git rev-parse --abbrev-ref HEAD`.
- Runs `git fetch --tags` against the caller's checkout.
- Resolves a "latest version" baseline:
  - If `test_version_tag` is set, uses it verbatim (with leading `v` stripped).
  - Else if both `major_version` and `minor_version` are set, picks the
    highest patch tag matching `^v?MAJOR.MINOR.[0-9]+$`, or starts the series
    at `MAJOR.MINOR.0` if no such tag exists.
  - Else picks the highest tag matching `^v?[0-9]+\.[0-9]+\.[0-9]+$` repo-wide,
    or falls back to `0.1.0` for an empty repo.
- Increments the patch component by 1 to form `nextBaseVersion`, except on
  the initial-version path where it keeps `MAJOR.MINOR.0`.
- Appends a postfix template based on branch name (see the `version` output
  row above for the exact rules).
- Writes `version=<string>` to `$GITHUB_OUTPUT`. No file is mutated, no tag is
  pushed, no commit is made, no remote API call is performed.

## Constraints callers must not violate

- Do not skip `actions/checkout` before this action — the script calls
  `git rev-parse` and `git tag --list` against the working directory; without
  a checkout the step exits non-zero and the job fails.
- Do not use `actions/checkout` with the default `fetch-depth: 1` if your
  repo has more than one tag — shallow clones may omit tag history, causing
  the action to compute against `0.1.0` instead of your real latest tag and
  silently produce a regressed version string.
- Do not set only one of `major_version` / `minor_version` — the override is
  applied only when both are non-empty; setting one alone is silently
  ignored and the action falls back to repo-wide tag scan, which will not
  match the cicd.yaml-declared series.
- Do not set `test_version_tag` or `test_branch_name` in a production
  workflow — these are simulation hooks; production callers will get a
  hardcoded version that does not reflect repo state.
- Do not assume the trunk branch is `main` — if your trunk is `master` or
  another name and you do not pass `release_branch`, the action will treat
  trunk builds as ad-hoc branches and emit `-<branch>-<sha>` postfixes,
  breaking downstream "is this a release?" checks.
- Do not parse the `version` output as strict SemVer 2.0.0 in every case —
  the postfix uses `-` separators and the leading prerelease identifier
  starts with a letter that is allowed by SemVer, but downstream consumers
  that reject SemVer prerelease segments (some Docker registries) require
  callers to strip or transform the postfix themselves.
- Do not call this action expecting it to push a tag for the computed
  version — it only emits the string. Tag creation is handled separately
  (in this repo, by `.github/workflows/version-incrementor.yaml`, which is
  this action's own release tagger and is not invoked by callers).
- Do not rename the step that calls the action away from a stable `id:` if a
  later step in the same job references `steps.<id>.outputs.version` — the
  action's own output name is fixed to `version`, but the step `id:` is
  caller-defined and renaming it breaks every downstream reference.

## Escalation

Stop and ask a human before:

- Changing `action.yaml` `inputs:` keys, defaults, or removing existing
  inputs — these form the public contract; consumer workflows in unrelated
  repos pin against them.
- Renaming or removing the `compute-version` step `id` inside `action.yaml`
  — the `outputs.version.value` expression
  `${{ steps.compute-version.outputs.version }}` references that id by
  name, and any other workflow that polls the step by id will break.
- Renaming or removing the `version` output — every consumer reads
  `steps.<id>.outputs.version`.
- Changing the postfix grammar (the `case "$branchName"` block) — downstream
  Docker tag, ECR repository, and gitops promotion logic in caller repos
  parses the postfix shape; silent changes cause mis-routed deploys.
- Changing the default of `release_branch` away from `main` — every caller
  that omits the input today will start producing postfixed versions on
  trunk.
