# Troubleshooting

Common problems when using `go-keyring`, grouped by platform. Each entry lists
the symptom, the cause, and the fix.

Before debugging, make sure you understand which platform provider is in use —
see [usage.md](usage.md#platform-notes).

## Cross-platform

### `secret not found in keyring` (`ErrNotFound`)

- `Get`/`Delete` were called for a (`service`, `user`) pair that was never
  stored, was stored under a different pair, or was already deleted.
- On **Windows**, remember the credential target is stored as `service:user` —
  check the exact name in Credential Manager.
- On **Linux/BSD**, verify the entry exists with
  `secret-tool lookup service "<service>" username "<user>"`.
- On **macOS**, verify with
  `security find-generic-password -s "<service>" -wa "<user>"`.

### `data passed to Set was too big` (`ErrSetDataTooBig`)

The value exceeds the platform limit. Keep secrets small (tokens, passwords);
for larger payloads, store a reference in the keyring and the data elsewhere.
Limits per platform are documented in
[usage.md](usage.md#platform-limits).

### `unsupported platform: <GOOS>` (`ErrUnsupportedPlatform`)

The binary was built for an OS without a keyring provider (anything other than
macOS, Windows, Linux, or the supported BSDs). There is no file-based fallback —
use environment variables or a secrets manager on such platforms, or gate the
keyring calls behind a runtime check.

## Linux / *BSD

All Linux/BSD errors ultimately come from the Secret Service D-Bus API, so the
exact message varies. The most common ones:

### `The name org.freedesktop.secrets was not provided by any .service files`

No Secret Service provider is running. Install and start one:

```sh
# Debian/Ubuntu
sudo apt-get install gnome-keyring

# Fedora/RHEL
sudo dnf install gnome-keyring
```

On minimal/headless systems you must also start a D-Bus session and unlock the
daemon yourself:

```sh
dbus-run-session -- sh -c "echo 'somecredstorepass' | gnome-keyring-daemon --unlock; ./your-app"
```

### `Cannot autolaunch D-Bus without X11 $DISPLAY` / `unable to connect to D-Bus`

Typical in SSH sessions, containers, and CI. Wrap your command (or your tests)
in `dbus-run-session` as shown above. In CI, the full recipe is:

```sh
sudo apt-get install -y gnome-keyring dbus-x11 libsecret-tools
dbus-run-session -- sh -c "echo 'somecredstorepass' | gnome-keyring-daemon --unlock; go test ./..."
```

### `Set`/`Get` hangs or prompts interactively

The `login` collection is locked. Unlock it once (e.g. via the Seahorse GUI, or
by piping the keyring password to `gnome-keyring-daemon --unlock` as above) so
the Secret Service can serve requests non-interactively.

### Entries not visible between tools

The library always writes to the **default `login` collection** with attributes
`service` and `username`. If another tool stores the secret in a different
collection or with different attribute names, `go-keyring` will not find it
(and vice versa). Inspect entries with Seahorse or
`secret-tool search service "<service>"`.

### Poor performance with large secrets

There is no hard size limit on Linux, but values over ~100 KiB become slow.
Store large payloads elsewhere and keep only a reference in the keyring.

## macOS

### `security: SecKeychainSearchCopyNext: The specified item could not be found in the keychain.`

This is the raw `/usr/bin/security` output for a missing entry — the library
converts it to `ErrNotFound`. If you see it as an *unexpected* error, check
that the service and user strings match what was stored (they are
case-sensitive).

### Repeated "allow access" prompts

macOS ties keychain item access to the calling binary. Rebuilt or unsigned
binaries may trigger permission prompts for items created by an older build.
Options:

- click **Always Allow** when prompted,
- sign your binary (a stable identity stops the prompts),
- or delete and recreate the item from the final binary.

### `The user name or passphrase you entered is not correct` / keychain locked

The login keychain is locked (e.g. after sleep on some configurations). Unlock
it with `security unlock-keychain` or via Keychain Access, then retry.

### `ListUsers` returns nothing although entries exist

`ListUsers` inspects the **default keychain** only. Entries stored in a
non-default keychain are not listed (though `Get` can still find them). Check
your default keychain with `security default-keychain`.

## Windows

### Password rejected on `Set`

Windows Credential Manager limits generic credential passwords to **2560 bytes**
and the library additionally caps the service name at **< 512 bytes**. Larger
inputs fail with `ErrSetDataTooBig`.

### Cannot find the stored credential in Credential Manager

Look under **Windows Credentials → Generic Credentials** for a target named
`service:user` — the library joins the two parameters with a colon. `cmdkey`
does not display stored passwords; use the PowerShell `CredentialManager`
module instead:

```powershell
Install-Module -Name CredentialManager -Force
(Get-StoredCredential -Target "service:user").GetNetworkCredential().Password
```

### Credentials missing for other users/services

Generic credentials are stored per Windows user profile. A secret written by
one user (or by a scheduled task/service running as a different account) is not
visible to another account.

## Still stuck?

- Reproduce the operation with the native CLI tooling listed in
  [usage.md](usage.md#verifying-secrets-with-os-tooling) to tell a library
  problem apart from a keyring/environment problem.
- Run the library tests on the target machine: `go test ./...` — they exercise
  the real platform provider.
- If the problem persists, open an issue in this repository with your OS, Go
  version, the failing call, and the full error output. Do **not** include any
  actual secrets.
