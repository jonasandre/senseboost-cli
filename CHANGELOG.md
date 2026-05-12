# Changelog

All notable changes to the senseboost CLI are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Each released version below corresponds to a Git tag of the form `senseboost-cli-vX.Y.Z` in the private SenseBoost monorepo and a Git tag `vX.Y.Z` plus GitHub Release in the public distribution repo [`jonasandre/senseboost-cli`](https://github.com/jonasandre/senseboost-cli/releases).

## [Unreleased]

<!--
Add new entries here as work lands on main. Move them under a new
"## [vX.Y.Z] — YYYY-MM-DD" heading when cutting a release.

Use the following categories (omit empty ones):
- Added       — new user-visible features
- Changed     — non-breaking changes in existing behavior
- Deprecated  — features still present but slated for removal
- Removed     — features removed in this release
- Fixed       — bug fixes
- Security    — security-relevant fixes
- Output      — changes to the JSON envelope or HTML report shape
-->

## [v0.2.0] — 2026-05-12

### Added

- Shell completion generation via `senseboost completion` for bash, zsh, fish, and PowerShell.
- Dynamic completion of saved context names for `senseboost context use`, `senseboost context delete`, and `senseboost analyze --context`.

### Changed

- `senseboost completion` now tries to detect the current shell automatically when no shell is passed, while keeping explicit shell selection as a fallback.
- Public CLI docs now explain shell completion setup with simpler usage examples.

## [v0.1.1] — 2026-05-12

### Fixed

- Bookmark-backed objects such as `alertbookmark` and `subscriptionbookmark` now use the correct Qlik bookmark handle during analysis, removing recurring invalid-handle warnings from app scans.
- Dimension validation no longer sends plain field names such as `CPF/CNPJ Fornecedor` or `Mês/Ano` to `CheckExpression`, which removes false-positive JSON parse errors for direct field definitions.

### Changed

- CLI progress and completion lines now use a consistent timestamped format with an explicit log level (`INFO` / `WARN`).

### Output

- JSON analysis items now include explicit `used: "yes" | "no"` fields for fields, variables, dimensions, and measures while keeping the existing usage counters.
- JSON dimensions and measures now include explicit `broken: "yes" | "no"` fields alongside the existing boolean/status fields.
- HTML reports now render yes/no badges for usage, highlight unused items more consistently, and show broken status explicitly for measures.

## [v0.1.0] — 2026-05-11

Initial public release.

### Added

- New `senseboost` binary for running the SenseBoost Qlik Cloud app analyzer locally without sending data to SenseBoost.
- `senseboost context create|list|use|delete` for managing named tenant + API-key pairs. API keys are stored in the OS-native credential vault (macOS Keychain, Windows Credential Manager, Linux Secret Service). Non-sensitive metadata (tenant URL, auth method, active context) is stored in a config file under `os.UserConfigDir()`.
- `senseboost analyze --app-id <id> --out <path> [--html <path>] [--context <name>] [--timeout <duration>] [--force]` for one-shot app analysis.
- Pre-check of output paths before opening the Qlik QIX WebSocket, so an existing file or unwritable directory fails immediately rather than after a multi-minute analysis.
- `--timeout` flag accepting any `time.ParseDuration` value (default `10m`, matching the SaaS worker handler).
- `--force` flag to overwrite existing output files.
- CI / non-interactive usage via environment variables: when `SENSEBOOST_QLIK_BASE_URL` is set, the CLI reads credentials from `SENSEBOOST_QLIK_BASE_URL` + `SENSEBOOST_QLIK_API_KEY` and ignores stored contexts entirely (no partial merge).
- `senseboost version` prints the build version, short commit SHA, and build timestamp.
- Static, self-contained HTML report with summary cards plus tables for fields, variables, dimensions, and measures. No external resources, no network requests on open.
- macOS (Intel + Apple Silicon), Linux (x86_64 + arm64), and Windows (x86_64) prebuilt binaries published to [`jonasandre/senseboost-cli` Releases](https://github.com/jonasandre/senseboost-cli/releases) alongside a `checksums.txt` file.

### Output

- JSON envelope `schemaVersion: 1` with `generatedAt`, `source` (`tenantUrl`, `appId`, `appName`, `spaceName`), and `analysis` (`summary`, `fields`, `variables`, `dimensions`, `measures`, `objects`).

### Security

- API keys are written only to the OS keychain; the CLI refuses to fall back to a plaintext file when the keychain is unavailable and instructs the user to use environment variables instead.
- Output files are written with `0600` permissions on Unix.
- A notice is printed when the generated report contains variable definitions or measure expressions, because those can encode business logic.
- No telemetry, no analytics, no opt-in pings.

### Known limitations

- Only API-key authentication is supported. OAuth client-credentials (M2M) is on the roadmap for a later release.
- HTML report is intentionally simple (summary cards + four tables). Richer visualizations are not planned for the free CLI — the SenseBoost SaaS provides those.
- No `--redact-expressions` flag yet; if you need to share a report without business-logic expressions, post-process the JSON envelope.

[Unreleased]: https://github.com/jonasandre/senseboost-cli/compare/v0.2.0...HEAD
[v0.2.0]: https://github.com/jonasandre/senseboost-cli/releases/tag/v0.2.0
[v0.1.1]: https://github.com/jonasandre/senseboost-cli/releases/tag/v0.1.1
[v0.1.0]: https://github.com/jonasandre/senseboost-cli/releases/tag/v0.1.0
