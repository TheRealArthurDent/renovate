# renovate

A repo just to streamline renovate executions and config

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Renovate enabled](https://img.shields.io/badge/renovate-enabled-brightgreen.svg)](https://renovatebot.com/)
[![Config Validator](https://github.com/TheRealArthurDent/renovate/actions/workflows/renovate-config-validator.yaml/badge.svg)](https://github.com/TheRealArthurDent/renovate/actions/workflows/renovate-config-validator.yaml)
[![Self-test](https://github.com/TheRealArthurDent/renovate/actions/workflows/renovate-selftest.yaml/badge.svg)](https://github.com/TheRealArthurDent/renovate/actions/workflows/renovate-selftest.yaml)
[![Reviewdog](https://github.com/TheRealArthurDent/renovate/actions/workflows/reviewdog.yaml/badge.svg)](https://github.com/TheRealArthurDent/renovate/actions/workflows/reviewdog.yaml)

[![GitHub pull-requests](https://img.shields.io/github/issues-pr/TheRealArthurDent/renovate.svg)](https://GitHub.com/TheRealArthurDent/renovate/pull/)

## Runs

Each workflow runs self-hosted Renovate as `P4ranoidAndroid` with its own
global config:

- `renovate.yaml` with `config.json`: renovate, iracing-settings
- `renovate-smarthome.yaml` with `config-smarthome.json`: homelab-engineering,
  infrastructure, smart-home, private-cloud, node-red-flows,
  ikea-firmware-downloader
- `renovate-arduino.yaml` with `config-arduino.json`: the ESP8266 libraries
  and leakage-sensor
- `renovate-selftest.yaml` with `config-selftest.json`: this repo, as a dry
  run on every PR

The global configs only hold what the runs need (bot account, repositories,
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
- A `minimumReleaseAge` of 3 days for every update, so a broken or hijacked
  release has time to be pulled before it lands. Updates whose registry has
  no release timestamp aren't held back.
- Security PRs from the OSV database (`osvVulnerabilityAlerts`).
- GitHub Actions: minor, patch, pin and digest updates in one PR with GitHub
  auto-merge, majors as single PRs.
- pre-commit hooks: minor, patch, pin and digest updates in one weekly PR
  (Monday morning) with GitHub auto-merge.

Auto-merge uses GitHub's auto-merge on the PR (`automergeType: pr`): the
repos' rulesets require a PR and green checks for `main`, so a branch
automerge can't push there. Docker images are never auto-merged: `main` is
what the next Ansible run rolls out to the Pis.

Renovate reads the preset from `main`, so a change to it applies to all repos
at their next run.

The `Config Validator` workflow checks all of these files with
`renovate-config-validator --strict` from the same Renovate image the runs
use: the global configs as global config, `renovate.json` and `homelab.json`
as repo config.
