# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Checkmarx governance layer in preparation for public release:
  `CODEOWNERS`, GitHub issue templates, pull request template, DCO sign-off
  workflow, Checkmarx One scan workflow, and a pre-commit configuration
  (`gofmt`, `go vet`, `gitleaks`).
- Project documentation: [docs/usage.md](docs/usage.md),
  [docs/troubleshooting.md](docs/troubleshooting.md), and this changelog.

### Changed

- Rewrote `CONTRIBUTING.md` and `MAINTAINERS` for the Checkmarx-maintained
  fork.

### Removed

- Upstream Zalando governance documents (`CONTRIBUTING.md`, `MAINTAINERS`,
  `SECURITY.md`) — superseded by the Checkmarx files at the repository root and
  the Checkmarx organization-level security policy. Upstream attribution
  remains in `LICENSE` and the git history.
- Zalando-era repository automation configuration (`.zappr.yml`,
  `.catwatch.yml`) and Dependabot configuration (`.github/dependabot.yml`).
- Legacy upstream `Go` CI workflow (`.github/workflows/go.yml`).

## [v0.2.8] - 2026-03-23

Code baseline of this fork, synchronized with upstream
[zalando/go-keyring](https://github.com/zalando/go-keyring) release **v0.2.8**
(module path `github.com/zalando/go-keyring`), including subsequent upstream
maintenance merges.

Feature set at this baseline:

- OS keyring providers: macOS Keychain (via `/usr/bin/security`), Linux/BSD
  Secret Service (via D-Bus), and Windows Credential Manager (via `wincred`).
- Package-level API: `Set`, `Get`, `Delete`, `DeleteAll`, and `ListUsers`.
- Sentinel errors: `ErrNotFound`, `ErrSetDataTooBig`, and
  `ErrUnsupportedPlatform`.
- In-memory mock provider for tests: `MockInit`, `MockInitWithError`, and
  `MockRestore`.

For upstream changes included in this baseline, see the
[upstream release history](https://github.com/zalando/go-keyring/releases).

[Unreleased]: https://github.com/CheckmarxDev/go-keyring/compare/master...HEAD
