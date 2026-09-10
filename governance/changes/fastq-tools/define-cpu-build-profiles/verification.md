# Verification: define-cpu-build-profiles

- Status: `Not started`
- Ready to archive: `no`

| Requirement | Evidence | Result |
|---|---|---|
| Portable default | `<compile commands/tests>` | not run |
| Explicit optimized profiles | `<configure/build tests>` | not run |
| Invalid/unsupported fail | `<negative tests>` | not run |
| Build identity | `<summary/artifact>` | not run |
| Legacy option migration | `<configure tests>` | not run |
| No runtime-dispatch claim | `<doc audit>` | not run |

必需门禁：lint check、build、tests、compile-command audit、`git diff --check`。
