# Changelog

All notable changes to the perf-lint GitHub Action are documented here.

## [1.0.0] — 2026-02-28

### Added
- Initial release of the perf-lint GitHub Action
- Composite action wrapping `perf-lint check`
- Inputs: `paths`, `version`, `severity`, `format`, `config`,
  `ignore_rule`, `upload_sarif`, `fail_on_violations`, `api_key`
- Outputs: `violations`, `score`, `grade`, `sarif_file`
- SARIF upload to GitHub Code Scanning via `github/codeql-action/upload-sarif`
- Silent POST of full results to dashboard when `api_key` is set
- Major version tag (`v1`) updated automatically on each release
