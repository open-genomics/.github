# Design: <change-id>

## Context

<与 proposal 对应的实现背景；引用代码证据，不复制整份审计。>

## Goals

- `<goal>`

## Non-goals

- `<non-goal>`

## Current flow

```text
<current data/control flow>
```

## Target flow

```text
<target data/control flow>
```

## Decisions

### <decision title>

- Choice: `<selected approach>`
- Reason: `<why>`
- Alternatives rejected: `<alternatives and concrete tradeoff>`

## Allowed change surface

- `<file/module>`: `<reason>`

Files/modules outside this list require proposal revision and renewed approval.

## Contract and compatibility

<wire/CLI/schema/version/error behavior, fixture and migration rules。>

## Failure, resource and security behavior

<失败原子性、分配前校验、损坏输入、超限、权限或供应链风险。>

## Test and fixture design

| Requirement/scenario | Test level | Fixture/input | Expected result |
|---|---|---|---|
| `<name>` | `<unit/integration/CI/manual>` | `<path>` | `<observable result>` |

## Risks and mitigations

| Risk | Likelihood/impact | Mitigation |
|---|---|---|
| `<risk>` | `<...>` | `<...>` |

## Rollback details

<明确哪些内容可回退，哪些已生成数据需要继续兼容。>
