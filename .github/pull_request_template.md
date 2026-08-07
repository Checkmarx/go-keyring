## Description

<!-- What does this change do, and why? Link the issue it addresses. -->

Closes #

## Type of change

- [ ] Bug fix (non-breaking change that fixes an issue)
- [ ] New feature (non-breaking change that adds functionality)
- [ ] Breaking change (fix or feature that changes existing behaviour)
- [ ] Documentation / repository maintenance
- [ ] Dependency update

## Platforms affected

- [ ] Linux / BSD (Secret Service via D-Bus)
- [ ] macOS (Keychain)
- [ ] Windows (Credential Manager)
- [ ] Mock provider / testing
- [ ] None (docs or CI only)

## Testing performed

<!--
Describe how you verified the change. Include the platform you tested on and
the commands you ran, e.g.:

    go build ./... && go vet ./... && go test ./...
-->

## Checklist

- [ ] All commits are signed off (`git commit -s`) — the DCO check must pass
- [ ] Tests added or updated for this change
- [ ] `go build ./...`, `go vet ./...`, and `go test ./...` pass locally
- [ ] Code style matches the surrounding code (`gofmt` applied)
- [ ] Documentation updated where relevant
- [ ] No secrets, credentials, or personal data included in the diff
- [ ] CI and the Checkmarx One scan are green
