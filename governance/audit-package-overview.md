# open-genomics 架构与工程治理实施包

状态：负责人裁决后的第二版实施基线  
审计日期：2026-08-13  
适用范围：当前工作区内的 6 个独立仓库

## 1. 目的

这套文档不是泛化的“最佳实践清单”，而是面向后续实现模型的任务规格。它回答四个问题：

1. 当前代码和文档之间有哪些已经证实的矛盾；
2. 哪些问题会损坏数据、破坏兼容性或制造不可复现结果；
3. 应按什么依赖关系拆分修改，避免多个模型互相覆盖；
4. 每项修改完成到什么程度才算验收通过。

本版只形成设计和修复说明，没有修改任何仓库源码、配置、测试或 CI。

## 2. 审计快照

| 仓库 | 审计提交 | 定位 |
|---|---:|---|
| `compress-kit` | `aa6472604fe2` | 教学/实验性质的压缩算法实现，二进制格式是主要契约 |
| `fastq-tools` | `5bab799bb9be` | 工程化较完整的 C++ FASTQ 工具集 |
| `fq-compressor` | `1361d4e8628a` | C++ FASTQ 压缩产品，现有 `.fqc` 实现更成熟 |
| `fq-compressor-rust` | `1a2a2161bed8` | 独立的 Rust FASTQ 压缩实现，当前与 C++ 同名但格式不兼容 |
| `micos-2024` | `c2c399f89c9c` | 多组学研究工作流，核心风险是可复现性与双重编排 |
| `minibwa-rust` | `dad6e99b064a` | 以 C 参考实现为行为标准的纯 Rust 重写 |

审计时六个仓库工作区均为干净状态。工作区根目录 `/home/shane/github/open-genomics` 有意作为多仓库容器，本身不是 Git 仓库；六个一级子目录分别拥有独立 Git 历史。根目录设计包是临时审计与派工入口，不作为实施后的规范真相。正式规格、change、验证记录和归档必须进入各自子仓库；跨仓库决定在相关仓库分别保存同一决策 ID 和相互链接。

## 3. 文档地图

| 文档 | 内容 | 主要执行者 |
|---|---|---|
| [00-roadmap-and-execution-protocol.md](00-roadmap-and-execution-protocol.md) | 总体风险、优先级、依赖顺序、实现模型协议 | 所有实现模型、维护者 |
| [01-fqc-product-and-format-governance.md](01-fqc-product-and-format-governance.md) | 两个 `fqc/.fqc` 项目的产品边界和格式治理 | 两个 FQC 仓库维护者 |
| [02-fq-compressor-rust-hardening.md](02-fq-compressor-rust-hardening.md) | Rust 格式规范校正、原子输出、资源边界、CI | Rust 实现模型 |
| [03-micos-reproducibility.md](03-micos-reproducibility.md) | 单一编排入口、版本矩阵、路径与容器可复现性 | MICOS 实现模型 |
| [04-minibwa-rust-parity-ci.md](04-minibwa-rust-parity-ci.md) | 让“与 C 一致”成为 CI 中真实执行的契约 | minibwa 实现模型 |
| [05-compress-kit-format-versioning.md](05-compress-kit-format-versioning.md) | 为已发生的破坏性格式变化建立明确版本边界 | compress-kit 实现模型 |
| [06-fastq-tools-release-portability.md](06-fastq-tools-release-portability.md) | 发布二进制 CPU 基线、发布闭环和身份修正 | fastq-tools 实现模型 |
| [07-fq-compressor-contract-hardening.md](07-fq-compressor-contract-hardening.md) | 校验语义、兼容性样本、解析器安全门禁 | C++ FQC 实现模型 |
| [08-organization-engineering-baseline.md](08-organization-engineering-baseline.md) | 轻量组织级工程基线、仓库分级、元数据治理 | 组织维护者 |
| [09-decision-register-and-task-index.md](09-decision-register-and-task-index.md) | 未决事项、全部任务状态、依赖和后续模型领取入口 | 负责人、派工模型 |
| [10-repository-local-openspec-workflow.md](10-repository-local-openspec-workflow.md) | 仓库内 proposal/spec/design/tasks/apply/verify/archive 工作流 | 所有实现与评审模型 |
| [11-first-wave-change-plan.md](11-first-wave-change-plan.md) | 六仓首轮 change 名称、任务边界和派工顺序 | 派工模型、维护者 |
| [templates/openspec/README.md](templates/openspec/README.md) | 可直接复制的 project/proposal/spec/design/tasks/verification 模板 | Propose 模型 |
| [changes/README.md](changes/README.md) | 六个首轮 change 的完整可复制草案包 | Propose/Apply 模型 |

## 4. 结论先行

当前最危险的问题不是代码风格，而是契约不清：

- 两个不同实现按负责人决定继续共享 `fqc` 命令和 `.fqc` 后缀，但尚未完整公开格式族差异和跨族错误行为；
- Rust `.fqc` 规范中的 codec/checksum 编号与真实编码不一致；
- MICOS 同时存在 Python 与 WDL 两套流程叙述，但真正的生产入口和配置生效范围不清；
- minibwa-rust 的 CI 在没有 C 参考二进制时静默跳过全部 parity 检查；
- compress-kit 已经做了不兼容的格式修改，却没有更换 magic 或版本字段；
- fastq-tools 将 `x86-64-v3` 称为“portable”发布基线，会排除一部分旧 x86-64 机器。

因此实施顺序必须是“先确定契约，再固化测试，然后修改实现，最后扩展自动化”，不能反过来。

## 5. 决策状态

决策使用 `Accepted / Superseded`，change 使用 `Draft / Proposed / Approved / Applying / Verifying / Ready to archive / Archived / Blocked`。代码只能从 Approved change 开始实施；`Accepted` 决策不等于授权任意范围修改。

负责人已在 2026-08-13 确认：

1. C++ 与 Rust 都保留 `fqc` 命令和 `.fqc` 后缀，以不同 magic 和格式族 ID 区分；不引入新的命令或后缀；
2. compress-kit 采用 `HFM2/AEN2/RCN2/RLE2`，writer 只写 v2，v1 明确拒绝，发布版本为 `2.0.0`；
3. MICOS 以 Python CLI 为唯一生产编排器，当前 WDL 为实验性单步骤参考；
4. 工作区继续保持六个独立 Git 仓库，规格工作流落在各自仓库，不把根目录变成 monorepo。

这些决定已经解除相关实施阻塞。仍需维护者提供的外部事实包括 minibwa C 参考提交、各仓库 owner 和安全联系方式；实现模型不得猜测这些值。

## 6. 给实现模型的最短指令

将下面这段连同一个仓库内 change ID 一起交给实现模型：

> 只应用指定 `openspec/changes/<change-id>/`。先完整阅读目标仓库的 `AGENTS.md`、proposal、delta spec、design 和 tasks；以允许修改范围、非目标和验收场景为边界。不要顺手重构，不要修改其他独立仓库，不要提交、推送或发布。先运行基线验证，逐项实施并记录证据；任何场景未通过都不得归档。

推荐每个任务单独分支、单独评审。跨仓库任务必须拆成各仓库独立提交，不能把六个仓库当作一个 monorepo 操作。
