# perf-lint action

> Static analysis for performance test scripts — in your CI pipeline.

Runs [perf-lint](https://github.com/markslilley/perf-lint) against JMeter `.jmx`,
k6 `.js`/`.ts`, and Gatling `.scala`/`.kt` files. Catches missing think times,
hardcoded values, absent assertions, unrealistic ramp patterns, and 15+ other
quality issues before they reach your load environment.

---

## Quick start

```yaml
# .github/workflows/perf-lint.yml
name: perf-lint

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: markslilley/perf-lint-action@v1
        with:
          paths: tests/performance/
```

## With dashboard (Pro / Team violations)

```yaml
- uses: markslilley/perf-lint-action@v1
  with:
    paths: tests/performance/
    api_key: ${{ secrets.PERF_LINT_API_KEY }}
```

Get your API key at [perflint.martkos-it.co.uk](https://perflint.martkos-it.co.uk).
Full results (all tiers) are sent silently to your dashboard — the CLI output
always shows only free-tier violations.

## With GitHub Code Scanning (SARIF)

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
      contents: read

    steps:
      - uses: actions/checkout@v4

      - uses: markslilley/perf-lint-action@v1
        with:
          paths: tests/performance/
          upload_sarif: 'true'
          api_key: ${{ secrets.PERF_LINT_API_KEY }}
```

Violations appear inline on pull requests under the **Security** tab.

---

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `paths` | No | `.` | Files or directories to analyse (space-separated) |
| `version` | No | `latest` | perf-lint version to install |
| `severity` | No | `warning` | Minimum severity for non-zero exit: `error` \| `warning` \| `info` |
| `format` | No | `text` | Primary output format: `text` \| `json` \| `sarif` |
| `config` | No | — | Path to `.perf-lint.yml` config file |
| `ignore_rule` | No | — | Comma-separated rule IDs to ignore (e.g. `JMX001,K6003`) |
| `upload_sarif` | No | `false` | Upload SARIF to GitHub Code Scanning |
| `fail_on_violations` | No | `true` | Fail the step when violations are found |
| `api_key` | No | — | perf-lint API key — store as a repository secret |

## Outputs

| Output | Description |
|--------|-------------|
| `violations` | Total free-tier violations found |
| `score` | Overall quality score (0–100) |
| `grade` | Overall quality grade (A–F) |
| `sarif_file` | Path to SARIF file (when `upload_sarif: true`) |

---

## Tier model

perf-lint runs **all 53 rules** locally — scripts never leave your machine.
The terminal output shows **18 free-tier rules**. Pro (22) and Team (13) violations
are counted and hinted at; the full picture is visible in your dashboard.

| Tier | Rules | What it covers |
|------|-------|---------------|
| Free | 18 | Basic hygiene: missing managers, think time, thresholds |
| Pro | 22 | All ERRORs + production-critical warnings |
| Team | 13 | Advanced patterns: correlation, memory, architecture |

## Examples

### Fail only on errors

```yaml
- uses: markslilley/perf-lint-action@v1
  with:
    paths: tests/
    severity: error
```

### Pin a specific version

```yaml
- uses: markslilley/perf-lint-action@v1.0.0
  with:
    paths: tests/
```

### Report-only mode (never fail CI)

```yaml
- uses: markslilley/perf-lint-action@v1
  with:
    paths: tests/
    fail_on_violations: 'false'
```

### Ignore specific rules

```yaml
- uses: markslilley/perf-lint-action@v1
  with:
    paths: tests/
    ignore_rule: 'JMX013,GAT005'
```

### Use a config file

```yaml
- uses: markslilley/perf-lint-action@v1
  with:
    paths: tests/
    config: .perf-lint.yml
```

---

## Permissions

| Feature | Permission needed |
|---------|------------------|
| Basic lint | None (default) |
| SARIF upload | `security-events: write` |

---

## License

MIT — see [LICENSE](LICENSE).
