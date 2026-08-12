# Usage

`go-keyring` is an OS-agnostic Go library for *setting*, *getting* and
*deleting* secrets in the system keyring. It supports **macOS** (Keychain),
**Linux/BSD** (Secret Service via D-Bus) and **Windows** (Credential Manager).

This repository is a Checkmarx-maintained fork of
[zalando/go-keyring](https://github.com/zalando/go-keyring). All issues and
pull requests are owned by Checkmarx — please do not file them upstream.

## Installation

```sh
go get github.com/zalando/go-keyring
```

The module path keeps the upstream `github.com/zalando/go-keyring` identity, so
existing imports continue to work. Requires Go 1.18 or newer.

## Quick start

```go
package main

import (
    "log"

    "github.com/zalando/go-keyring"
)

func main() {
    service := "my-app"
    user := "anon"
    password := "secret"

    // store a secret
    if err := keyring.Set(service, user, password); err != nil {
        log.Fatal(err)
    }

    // retrieve it
    secret, err := keyring.Get(service, user)
    if err != nil {
        log.Fatal(err)
    }
    log.Println(secret)
}
```

A secret is identified by the pair (`service`, `user`):

- `service` scopes the secret to your application, e.g. `"my-cli"`.
- `user` identifies the account within that service, e.g. a username or e-mail.

## API reference

All functions operate on the platform provider selected at build time.

| Function | Description |
| --- | --- |
| `Set(service, user, password string) error` | Store `password` for (`service`, `user`). Overwrites an existing entry. |
| `Get(service, user string) (string, error)` | Retrieve the secret. Returns `ErrNotFound` if there is no entry. |
| `Delete(service, user string) error` | Delete a single entry. Returns `ErrNotFound` if there is no entry. |
| `DeleteAll(service string) error` | Delete **all** entries for a service. An empty `service` is rejected with `ErrNotFound` to prevent wiping the keyring. |
| `ListUsers(service string) ([]string, error)` | Return all users that have an entry under `service`, sorted and de-duplicated. |

### Errors

| Error | Meaning |
| --- | --- |
| `keyring.ErrNotFound` | The requested secret does not exist in the keyring. |
| `keyring.ErrSetDataTooBig` | The value passed to `Set` exceeds the platform limit (see [Platform limits](#platform-limits)). |
| `keyring.ErrUnsupportedPlatform` | The library was built for an OS without a keyring provider. |

Handle the common "not found" case explicitly:

```go
secret, err := keyring.Get("my-app", "anon")
if errors.Is(err, keyring.ErrNotFound) {
    // prompt the user for credentials, then keyring.Set(...)
} else if err != nil {
    return err
}
```

## Platform notes

### macOS

- Uses the `/usr/bin/security` binary to talk to the default Keychain; no cgo
  required.
- Secrets are stored as *generic passwords* with the service name in `-s` and
  the user in `-a`.
- Passwords are base64-encoded by the library before storage (values carry the
  `go-keyring-base64:` prefix) so that multi-line and non-ASCII secrets survive
  the round trip. Decoding is transparent on `Get`.

### Linux and *BSD

- Uses the [Secret Service API](https://specifications.freedesktop.org/secret-service-spec/latest/)
  over D-Bus (pure Go, no cgo). A Secret Service provider must be running —
  typically **GNOME Keyring**; KeePassXC and KWallet (via integration) also work.
- Secrets are stored in the default `login` collection with attributes
  `service` and `username`, and labelled `Password for '<user>' on '<service>'`.
- On headless systems (CI, SSH sessions, containers) there is no D-Bus session
  or keyring daemon by default — see
  [troubleshooting](troubleshooting.md#linux--bsd) for how to run one.

### Windows

- Uses the Windows Credential Manager through the
  [wincred](https://github.com/danieljoos/wincred) bindings; no cgo required.
- The credential target name is `service:user`. Entries appear under
  **Credential Manager → Windows Credentials → Generic Credentials**.
- Credentials persist for the local user profile that created them.

### Platform limits

| Platform | Limit |
| --- | --- |
| macOS | Combined service, username and password should not exceed ~3000 bytes (the generated `security` command is capped at 4096 bytes). |
| Windows | Password ≤ 2560 bytes; service < 512 bytes. |
| Linux/BSD | No hard limit, but performance degrades with values larger than ~100 KiB. |

Exceeding a limit makes `Set` return `ErrSetDataTooBig`.

## Testing with the mock provider

Use the in-memory mock to unit-test code that depends on the keyring without
touching the real OS keyring:

```go
func TestLoginFlow(t *testing.T) {
    keyring.MockInit()
    defer keyring.MockRestore() // restore the platform provider

    if err := keyring.Set("my-app", "anon", "secret"); err != nil {
        t.Fatal(err)
    }
    // ... exercise your code ...
}
```

- `MockInit()` swaps in the in-memory provider.
- `MockRestore()` restores the platform provider (useful with `defer` when
  several tests share the process).
- `MockInitWithError(err)` returns a provider whose operations all fail with
  `err`, for testing error paths.

## Verifying secrets with OS tooling

For debugging you can inspect the same entries with native tools:

- **macOS:** `security find-generic-password -s "my-app" -wa "anon"`
- **Linux/BSD:** `secret-tool lookup service "my-app" username "anon"`
  (from the `libsecret-tools` package)
- **Windows:** entries are visible in Credential Manager; passwords can be read
  with the PowerShell `CredentialManager` module (`Get-StoredCredential`).

See the [README](../README.md#direct-cli-usage) for the full set of CLI
examples, including how to pre-seed secrets from scripts.

## Further reading

- [Troubleshooting](troubleshooting.md) — platform-specific problems and fixes.
- [Contributing](../CONTRIBUTING.md) — development workflow, DCO sign-off, tests.
