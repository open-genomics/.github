# Verification: <change-id>

## Metadata

- Verification status: `Not started | Failed | Passed`
- Implementation HEAD: `<full SHA or working tree base>`
- Verifier: `<handle/model>`
- Verified at: `<YYYY-MM-DD>`
- Ready to archive: `no`

## Scope audit

- Expected files/modules: `<...>`
- Actual changed files: `<...>`
- Unexpected changes: `<none or explanation>`
- Existing user changes preserved: `yes/no + evidence`

## Requirement traceability

| Requirement | Scenario | Test/command | Result | Evidence summary |
|---|---|---|---|---|
| `<name>` | `<scenario>` | `<command/test>` | `<pass/fail/not run>` | `<short evidence>` |

## Commands

| Command | Exit status | Result summary |
|---|---:|---|
| `<exact command>` | `<code>` | `<summary>` |

## Failure-side-effect checks

| Case | Expected invariant | Result |
|---|---|---|
| `<write/decode/config failure>` | `<old output preserved/no partial file/no silent fallback>` | `<...>` |

## Not run

- `<command/scenario>`: `<reason and consequence>`

## Residual risks

- `<risk or none>`

## Verdict

<说明为何可以或不可以归档。只有所有必需场景和门禁通过后，才能把顶部 Ready to archive 改为 yes。>
