# 首轮 Change 草案包

这些目录是面向六个独立 Git 仓库的可复制 OpenSpec-style change。它们保存在未版本化的工作区根目录，便于负责人派工；不得从这里直接执行 build、test、commit 或 archive。

| 目标仓库 | Change | 首要结果 |
|---|---|---|
| `fq-compressor-rust` | [correct-indexed-v2-spec](fq-compressor-rust/correct-indexed-v2-spec/proposal.md) | 修正 indexed v2 规范并冻结 fixture |
| `compress-kit` | [version-binary-formats-v2](compress-kit/version-binary-formats-v2/proposal.md) | 落地四个 v2 magic、legacy 拒绝和 2.0.0 契约 |
| `micos-2024` | [declare-python-production-orchestrator](micos-2024/declare-python-production-orchestrator/proposal.md) | 固化 Python 唯一生产入口和 WDL 实验性状态 |
| `minibwa-rust` | [correct-parity-contract-docs](minibwa-rust/correct-parity-contract-docs/proposal.md) | 如实公开 parity 的真实执行范围和 skip |
| `fastq-tools` | [define-cpu-build-profiles](fastq-tools/define-cpu-build-profiles/proposal.md) | 默认 portable，v3/native 显式选择 |
| `fq-compressor` | [freeze-sequential-v2-format](fq-compressor/freeze-sequential-v2-format/proposal.md) | 冻结 sequential v2 reader 契约和 fixtures |

每包含：

```text
proposal.md
design.md
tasks.md
verification.md
specs/<capability>/spec.md
```

落地步骤：

1. 模型进入目标子仓库，完整阅读根 `AGENTS.md`；
2. 比较当前 HEAD 与 proposal 的 audit base，重新核对发生漂移的事实；
3. 从 [通用模板](../templates/openspec/README.md) 建立 `openspec/project.md` 和 `openspec/AGENTS.md`；
4. 把该包复制为 `openspec/changes/<change-id>/`，把 audit base 更新为当前完整 SHA；
5. 只修订证据定位、仓库实际命令和已发生的代码漂移，不改变已批准 decision；
6. 若只授权 Propose，状态保持 Proposed 并停止；
7. 若负责人明确说“可以修改代码，批准 apply `<change-id>`”，改为 Approved 后逐项执行；
8. 实施模型不负责 archive；独立 verify 通过并获评审确认后再归档。

`FQCR-*` 是早期审计形成的 Rust 仓任务编号命名空间，不是产品名、命令或后缀。产品继续叫 `fqc`，archive 继续使用 `.fqc`。
