# perf-lint-action — Developer Guide

GitHub Action that runs `perf-lint` static analysis on performance test scripts (JMeter, k6, Gatling) in CI pipelines.

Published to [GitHub Marketplace](https://github.com/marketplace/actions/perf-lint).

---

## Overview

A composite action that:
1. Installs `perf-lint-tool` from PyPI (or a pinned version)
2. Runs `perf-lint check` against specified paths
3. Exports `violations`, `score`, and `grade` as step outputs

---

## Repository layout

```
perf-lint-action/
├── action.yml    ← composite action definition
├── CHANGELOG.md
├── LICENSE
└── README.md
```

---

## Versioning and tags

Releases use semver. The floating `v1` tag always points to the latest `v1.x.x` release.

**After merging a fix or feature:**

```bash
# Tag the new release
git tag v1.x.x
git push origin v1.x.x

# Move the floating v1 tag (required — users pin to @v1)
gh api --method PATCH /repos/markslilley/perf-lint-action/git/refs/tags/v1 \
  -f sha="$(git rev-parse HEAD)" \
  -F force=true
```

**IMPORTANT**: Use `-F force=true` (capital F), not `-f force=true`. Capital F is for typed values (boolean); lowercase -f sends strings.

---

## Known issues / past fixes

### violations=0 when violations are found (fixed in v1.0.1)

The original action used:
```bash
JSON=$(perf-lint check ... --format json 2>/dev/null || echo '{}')
```

When `perf-lint` exits 1 (violations found), the `|| echo '{}'` replaces the JSON stdout with `{}`, making violations=0. Fixed by:
```bash
set +e
JSON=$(perf-lint check ... --format json 2>/dev/null)
set -e
```

---

## Testing

The `perf-lint` public repo's CI (`ci.yml`) self-tests this action:

```yaml
- name: Run perf-lint against samples (report-only)
  id: lint
  uses: markslilley/perf-lint-action@v1
  with:
    paths: samples/
    fail_on_violations: 'false'

- name: Verify violations were found
  run: |
    if [ "${{ steps.lint.outputs.violations }}" -eq 0 ]; then
      echo "Expected violations from bad sample scripts but got 0"
      exit 1
    fi
```

After pushing a fix to this repo, move the `v1` floating tag before triggering the CI run.

---

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `paths` | `.` | Space-separated paths/globs to analyse |
| `version` | `latest` | perf-lint-tool version to install |
| `severity` | `warning` | Minimum severity for non-zero exit |
| `fail_on_violations` | `true` | Set `false` to always exit 0 (report-only) |
| `format` | `text` | Output format: `text`, `json`, `sarif` |
| `config` | `` | Path to `.perf-lint.yml` config file |

## Outputs

| Output | Description |
|--------|-------------|
| `violations` | Total number of violations found |
| `score` | Quality score (0–100) |
| `grade` | Quality grade (A–F) |

---

## Commands

```bash
# Validate action.yml syntax
python3 -c "import yaml; yaml.safe_load(open('action.yml'))" && echo "OK"

# Test locally with act (https://github.com/nektos/act)
act -j perf-lint-action
```
