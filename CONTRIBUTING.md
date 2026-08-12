# Contributing

Thanks for your interest in contributing. This repository is a Checkmarx-maintained
fork of [zalando/go-keyring](https://github.com/zalando/go-keyring). All issues and
pull requests for this fork are owned by Checkmarx — please do not file them upstream.

## Before you start

- Open an issue first for bugs and feature requests so the approach can be discussed
  before you invest time in an implementation.
- Check existing issues and pull requests to avoid duplicate work.

## Development workflow

1. Fork the repository and create a feature branch off `master`.
2. Make your change, keeping commits focused and readable.
3. Build and test locally:

   ```sh
   go build ./...
   go vet ./...
   go test ./...
   ```

   The keyring backend is platform-specific, so tests exercise the implementation
   for the OS you run them on (`keyring_darwin.go` on macOS, `keyring_unix.go` on
   Linux/BSD, `keyring_windows.go` on Windows). On headless Linux you need a
   running Secret Service:

   ```sh
   dbus-run-session -- sh -c "echo 'somecredstorepass' | gnome-keyring-daemon --unlock; go test ./..."
   ```

4. Open a pull request against `master`.

## Requirements for a pull request

- **Every code change should have a test.** New behaviour needs new coverage;
  bug fixes need a regression test.
- **Keep the current code style.** Match the surrounding code; run `gofmt` before
  committing.
- **Sign off every commit (DCO).** All commits must carry a `Signed-off-by` line
  certifying the [Developer Certificate of Origin](https://developercertificate.org/):

  ```sh
  git commit -s -m "your message"
  ```

  The DCO check runs on every pull request and must pass. To fix an existing
  branch, use `git rebase --signoff <base>` and force-push.
- **CI must be green.** The build and test matrix (Linux, macOS, Windows, FreeBSD)
  and the Checkmarx One security scan both run on pull requests and must pass.
- **Review.** Pull requests require approval from the code owners listed in
  [.github/CODEOWNERS](.github/CODEOWNERS) before merging. See
  [MAINTAINERS](MAINTAINERS).

## Optional: pre-commit hooks

This repository ships a [pre-commit](https://pre-commit.com/) configuration that
runs `gofmt`, `go vet`, and a `gitleaks` secret scan locally:

```sh
pip install pre-commit
pre-commit install
```

## Security issues

Do not report security vulnerabilities through public GitHub issues. Follow the
Checkmarx organization security policy linked from this repository's **Security**
tab.
