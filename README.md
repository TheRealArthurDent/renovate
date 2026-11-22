# renovate

A repo just to streamline renovate executions and config

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Renovate enabled](https://img.shields.io/badge/renovate-enabled-brightgreen.svg)](https://renovatebot.com/)
[![Config Validator](https://github.com/TheRealArthurDent/renovate/actions/workflows/renovate-config-validator.yaml/badge.svg)](https://github.com/TheRealArthurDent/renovate/actions/workflows/renovate-config-validator.yaml)
[![Self-test](https://github.com/TheRealArthurDent/renovate/actions/workflows/renovate-selftest.yaml/badge.svg)](https://github.com/TheRealArthurDent/renovate/actions/workflows/renovate-selftest.yaml)
[![Reviewdog](https://github.com/TheRealArthurDent/renovate/actions/workflows/reviewdog.yaml/badge.svg)](https://github.com/TheRealArthurDent/renovate/actions/workflows/reviewdog.yaml)

[![GitHub pull-requests](https://img.shields.io/github/issues-pr/TheRealArthurDent/renovate.svg)](https://GitHub.com/TheRealArthurDent/renovate/pull/)

## Runs

Each workflow runs self-hosted Renovate with its own global config. It runs as
the repo owner's GitHub App: before Renovate starts, the workflow creates an
installation token (valid for an hour) from the repository variable
`RENOVATE_APP_CLIENT_ID` and the secret `RENOVATE_APP_PRIVATE_KEY`, and Renovate
takes its username and commit author from the app. Commits are made through the
API, so GitHub shows them as verified. Branches the former bot account
`P4ranoidAndroid` created still count as Renovate's own (`gitIgnoredAuthors`),
and its PRs and Dependency Dashboards are still found (`ignorePrAuthor`, which
makes Renovate read all PRs and issues instead of only its own). Both can go
once no open Renovate PR or branch of the former account is left.

Each run limits its token to the repositories of its config plus this repo (for
the shared presets) and to the permissions Renovate uses; the self-test only
reads. The app needs these repository permissions and must be installed on every
repo the runs list: Checks (read), Commit statuses, Contents, Issues, Pull
requests and Workflows (read and write), Dependabot alerts (read), Metadata
(read).

The runs:

- `renovate.yaml` with `config.json`: renovate, iracing-settings
- `renovate-smarthome.yaml` with `config-smarthome.json`: homelab-engineering,
  infrastructure, smart-home, private-cloud, node-red-flows,
  ikea-firmware-downloader
- `renovate-arduino.yaml` with `config-arduino.json`: the ESP8266 libraries
  and leakage-sensor
- `renovate-selftest.yaml` with `config-selftest.json`: this repo, as a dry
  run on every PR

The global configs only hold what the runs need (repositories, ignored authors,
labels, the regex manager for `# renovate:` comments in
`.pre-commit-config.yaml`). They are copies of each other; keep them in sync.

## Shared preset for the homelab repos

`homelab.json` holds the rules that infrastructure, smart-home, private-cloud
and homelab-engineering share. Their `renovate.json` extends it and adds only
its own rules:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>TheRealArthurDent/renovate:homelab"]
}
```

What it sets:

- `config:recommended` and the pre-commit manager.
- A `minimumReleaseAge` of 14 days for every update, so a broken or hijacked
  release has time to be pulled before it lands. Updates whose registry has
  no release timestamp aren't held back. Security PRs (OSV and GitHub
  alerts) ignore it: Renovate's `vulnerabilityAlerts` default sets
  `minimumReleaseAge: null`.
- Security PRs from the OSV database (`osvVulnerabilityAlerts`).
- GitHub Actions pinned to a commit SHA, with the version as a comment
  (`helpers:pinGitHubActionDigests`). Their updates, digest and pin updates
  included, always wait the 14 days: the rule requires the release timestamp,
  which GitHub tags and releases have.
- GitHub Actions: minor, patch, pin and digest updates in one PR with GitHub
  auto-merge, majors as single PRs.
- pre-commit hooks: minor, patch, pin and digest updates in one weekly PR
  (Monday morning) with GitHub auto-merge.
- Docker image patches with GitHub auto-merge, but only once the release is
  14 days old. This rule requires the release timestamp, which only Docker
  Hub provides; images from ghcr.io, lscr.io and quay.io are left out and
  stay PRs merged by hand.

Auto-merge uses GitHub's auto-merge on the PR (`automergeType: pr`): the
repos' rulesets require a PR and green checks for `main`, so a branch
automerge can't push there. Minor and major image updates are never
auto-merged: `main` is what the next Ansible run rolls out to the Pis.

Renovate reads the preset from `main`, so a change to it applies to all repos
at their next run.

The `Config Validator` workflow checks all of these files with
`renovate-config-validator --strict` from the same Renovate image the runs
use: the global configs as global config, `renovate.json` and `homelab.json`
as repo config.

## This repo's own config

`renovate.json` covers this repo only and doesn't extend the shared preset (it
updates its pre-commit hooks through the regex manager of the global configs).
It pins GitHub Actions to a commit SHA as well and holds every update back for
14 days. Auto-merged updates get a PR, and Renovate merges it once its checks
pass: the workflows run on pull requests, so a branch without a PR would never
get checks.

The workflows give `GITHUB_TOKEN` read access to the contents only; Reviewdog
adds write access to pull requests for its review comments.

## Workflow lint

[zizmor](https://docs.zizmor.sh/) checks the workflows, as a pre-commit hook
and in the `zizmor` workflow on every PR. The workflow runs the version of the
hook in `.pre-commit-config.yaml`, which Renovate updates.
