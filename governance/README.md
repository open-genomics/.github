# Open Genomics 工程治理文档

> ⚠️ **这是 2026-08-13 的一次审计与派工记录，不是现状描述。**
> 审计范围、风险排序与待办清单都会随实施推进而过时——事实上已经大量过时。
> **当前事实请以各仓库自身（`README.md` / `AGENTS.md` / `openspec/`）为准。**
> 本目录的价值在于追溯「当时基于什么证据、做了什么决定」，而不是回答「现在是什么状态」。

本目录原为工作区根目录下不受版本控制的 `maintenance-design/`，2026-09-10 迁入此处纳入版本管理。

## 目录

| 文件 | 内容 | 现状 |
|:-----|:-----|:-----|
| [audit-package-overview.md](audit-package-overview.md) | 审计包原始说明（含审计快照与文档地图） | 部分过时 |
| [00-roadmap-and-execution-protocol.md](00-roadmap-and-execution-protocol.md) | 总体风险、优先级、依赖顺序 | 部分过时 |
| [01-fqc-product-and-format-governance.md](01-fqc-product-and-format-governance.md) | 两个 `fqc` 项目的产品边界与格式治理 | 有效 |
| [02-fq-compressor-rust-hardening.md](02-fq-compressor-rust-hardening.md) | Rust 实现加固规格 | 已实施 |
| [03-micos-reproducibility.md](03-micos-reproducibility.md) | MICOS 可复现性 | 部分实施 |
| [04-minibwa-rust-parity-ci.md](04-minibwa-rust-parity-ci.md) | 与 C 参考实现的一致性契约 | 部分实施 |
| [05-compress-kit-format-versioning.md](05-compress-kit-format-versioning.md) | compress-kit 格式版本化 | **目标仓库已不存在** |
| [06-fastq-tools-release-portability.md](06-fastq-tools-release-portability.md) | 发布二进制 CPU 基线 | 部分实施 |
| [07-fq-compressor-contract-hardening.md](07-fq-compressor-contract-hardening.md) | 校验语义与解析安全 | 已实施 |
| [08-organization-engineering-baseline.md](08-organization-engineering-baseline.md) | 组织级工程基线、仓库分级 | **部分已被 `.github` 仓库取代** |
| [09-decision-register-and-task-index.md](09-decision-register-and-task-index.md) | 决策登记与任务状态索引 | 决策有效，任务索引过时 |
| [10-repository-local-openspec-workflow.md](10-repository-local-openspec-workflow.md) | 仓库内 OpenSpec 工作流定义 | **仍在使用** |
| [11-first-wave-change-plan.md](11-first-wave-change-plan.md) | 首轮 change 的边界与派工顺序 | 已执行完毕 |
| [PLAN.md](PLAN.md) | 执行计划与交接提示词 | **整份过时** |
| [changes/](changes/) | 6 个首轮 change 的可复制草案包 | 历史草案 |
| [templates/openspec/](templates/openspec/) | 可直接复制的 OpenSpec 模板 | **仍可使用** |

## 与现实对照（2026-09-10 核对）

| # | 审计包中的说法 | 现状 | 影响 |
|:-:|:---------------|:-----|:-----|
| 1 | 适用范围为「当前工作区内的 6 个独立仓库」 | 工作区已有 **12 个仓库** + 组织级 `.github` | 7 个仓库从未被审计 |
| 2 | 审计对象含 `compress-kit` | 本地**已不存在**该目录 | `05` 号文档与 `changes/compress-kit/` 已无对应仓库 |
| 3 | `PLAN.md` 列出 5 个 change「因缺 Conan 无法归档」 | **全部早已归档**，verification 均为 `Ready to archive: yes` | 「Conan 阻塞」不成立；实测 Conan 2.31.2 可用 |
| 4 | `PLAN.md` 称 minibwa-rust 有未提交改动待收尾 | 已提交并推送 | — |
| 5 | 「审计时六个仓库工作区均为干净状态」 | 2026-09-10 复查发现 43 项未提交散落 4 个仓库 | 审计后又有大量未提交工作 |
| 6 | 决策 `FQC-DEC-001`：两个实现都保留 `fqc`/`.fqc`，以 magic 区分格式族 | **仍然有效**，已写入两个仓库 README | ✅ |
| 7 | 决策：MICOS 以 Python CLI 为唯一生产编排器 | **仍然有效** | ✅ |

## 仍然有效、可直接使用的部分

- **[10-repository-local-openspec-workflow.md](10-repository-local-openspec-workflow.md)** ——
  仓库内 `proposal → spec → design → tasks → apply → verify → archive` 工作流。
  这**仍是组织内正在使用的工作流**，各仓库 `openspec/` 均按此组织。
- **[templates/openspec/](templates/openspec/)** —— 可直接复制到新仓库的模板。
- **各号文档的问题定位方法** —— 基于具体提交取证、按数据损坏风险而非代码风格排序。
  这套方法值得沿用，即使具体结论已过时。

## 已失效、仅供追溯的部分

- **[PLAN.md](PLAN.md)** —— 整份「当前状态清单」与「执行计划」已过时（见对照表第 3、4 条）。
  其中「给 Flash 模型的交接提示词」已执行完毕，不应再按它派工。
- **[05-compress-kit-format-versioning.md](05-compress-kit-format-versioning.md)** 与
  `changes/compress-kit/` —— 目标仓库已不存在。
- **[09-decision-register-and-task-index.md](09-decision-register-and-task-index.md)** ——
  任务状态索引过时；其中的决策记录部分仍有效。
- **[08-organization-engineering-baseline.md](08-organization-engineering-baseline.md)** ——
  组织级基线的部分内容已由组织级 [`.github` 仓库](https://github.com/open-genomics/.github)
  实际落地（社区健康文件、Issue/PR 模板、CI 起步模板）。

## 本审计包未覆盖的仓库

以下仓库在审计之后才纳入治理范围，本文档不覆盖：
`awesome-bioinfo-algorithms`、`bwa-rust`、`hillock`、`lush-hc`、`lush_aligner1`、
`swift-duck`、`wiki-bioinfo`。

## 阅读顺序建议

若只想了解「现在该怎么做」，读 [10](10-repository-local-openspec-workflow.md) 与
[templates/openspec/](templates/openspec/) 即可，其余是历史。
若要追溯「某个决定当时基于什么证据」，从
[audit-package-overview.md](audit-package-overview.md) 进入。
