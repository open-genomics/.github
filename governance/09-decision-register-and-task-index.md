# 决策登记与实施任务索引

状态：2026-08-13 负责人裁决后版本

## 1. 使用规则

本文件是多仓库派工索引，不是任何项目的规范来源。后续模型先确认目标仓库和 change ID，再读取该仓库内的 `openspec/changes/<change-id>/`。

“可建 change”只允许在目标仓库建立 proposal/spec/design/tasks；“可 apply”还要求 proposal 已批准、代码修改已获明确授权。每个模型一次只处理一个仓库内 change，不得从根目录执行跨仓提交。

状态含义：

- **Accepted**：负责人已经给出确定值；相关 change 可以据此拟定和实施；
- **可建 change**：设计完整，可以由规格模型在目标仓库落成 artifacts；
- **前置任务**：必须先验证并归档列出的 change；
- **信息阻塞**：不能从仓库可靠推导，等待维护者提供；
- **Not applicable**：已被负责人决策排除，禁止实现；
- **后续**：本轮不实施。

## 2. 已批准决策

| 决策 ID | 状态 | 精确决定 | 落盘位置 |
|---|---|---|---|
| `FQC-DEC-001` | Accepted | 两个实现都保留 `fqc/.fqc`；以不同 magic 和 `fqc-sequential/v2`、`fqc-indexed/v2` 区分；不互相解码 | 两个 FQC 仓库各自 archive-format spec |
| `CK-DEC-001` | Accepted | `HFM2/AEN2/RCN2/RLE2`；writer 只写 v2；v1 明确拒绝；发布 `2.0.0` | `compress-kit` format specs |
| `MICOS-ORCH-001` | Accepted | Python CLI 为唯一生产编排器；当前 WDL 是实验性单步骤参考 | `micos-2024` workflow-orchestration spec |
| `ORG-GOV-001` | Accepted | 根目录不初始化 Git；六个子仓库分别维护 OpenSpec artifacts 和历史 | 六仓各自 `openspec/project.md` |

这些决定已经在本轮对话中获得负责人授权。首个相关 change 必须把精确值写入版本控制；不得继续将它们标成 unresolved，也不得自行选择相反方案。

仍然缺少且不可猜测的信息：

| 信息 | 当前缺口 | 解锁任务 |
|---|---|---|
| minibwa C 参考基线 | URL 已知为 `https://github.com/lh3/minibwa`；缺已验证 commit、确定构建命令和 fixture 来源 | `MBW-REF-001` |
| 项目维护责任 | 每仓库 owner、支持平台和 release channel | `ORG-LIFE-001` |
| 安全联系入口 | 可公开邮箱、GitHub private reporting 或组织 policy | `ORG-SEC-001` |

## 3. 仓库与任务矩阵

### 3.1 `fq-compressor`（C++）

| 任务 | 建议 change ID | 状态/前置 | 交付结果 | 规格 |
|---|---|---|---|---|
| `FQC-DOC-001`（C++ 部分） | `document-fqc-format-family` | 可建 change，P0 | README 明示同名、同后缀、不同格式族 | [01 §4](01-fqc-product-and-format-governance.md#4-任务-fqc-doc-001公开同名共存契约) |
| `FQC-CPP-DOC-001` | `correct-verify-contract` | 可建 change，P0 | `verify` 准确描述为完整解码校验 | [07 §3](07-fq-compressor-contract-hardening.md#3-任务-fqc-cpp-doc-001修正-verify-和产品边界) |
| `FQC-CPP-FMT-001` | `freeze-sequential-v2-format` | 可建 change，P0 | 顺序 v2 规范、已发布 fixture、损坏矩阵 | [07 §4](07-fq-compressor-contract-hardening.md#4-任务-fqc-cpp-fmt-001规范与冻结-v2-decoder-契约) |
| `FQC-FAMILY-001`（C++ 部分） | `recognize-indexed-fqc-family` | 前置：DOC 与 FMT，P0 | 识别 Rust magic 并专门拒绝 | [01 §5](01-fqc-product-and-format-governance.md#5-任务-fqc-family-001已知格式族的快速拒绝) |
| `FQC-IDENTITY-001`（C++ 部分） | `expose-sequential-fqc-identity` | 前置：FMT，P1 | version/info 展示实现和 format family | [01 §6](01-fqc-product-and-format-governance.md#6-任务-fqc-identity-001cli-身份可观察性) |
| `FQC-CPP-CI-001` | `add-sanitizer-contract-gate` | 先做本地依赖基线，P1 | 锁定依赖，ASan/UBSan 门禁 | [07 §5](07-fq-compressor-contract-hardening.md#5-任务-fqc-cpp-ci-001可信的依赖与-sanitizer-门禁) |
| `FQC-CPP-FUZZ-001` | `add-archive-fuzz-smoke` | 前置：FMT、CI，P1 | 确定性 corpus 和有界 fuzz smoke | [07 §6](07-fq-compressor-contract-hardening.md#6-任务-fqc-cpp-fuzz-001archivefastq-确定性-corpus-与-fuzz-smoke) |
| `FQC-CPP-META-001` | `correct-project-metadata` | 可先修 active URL/失效 vendor 排除；版权行另待权利人确认，P1 | active 链接正确；许可证适用范围准确且不擅改权利人 | [07 §7](07-fq-compressor-contract-hardening.md#7-任务-fqc-cpp-meta-001组织元数据与许可证清理) |
| `FQC-CPP-REL-001` | `gate-stable-fqc-release` | 前置：FMT、CI，P2 | 稳定版发布门 | [07 §8](07-fq-compressor-contract-hardening.md#8-后续发布门-fqc-cpp-rel-001) |

### 3.2 `fq-compressor-rust`

`FQCR-*` 仅是既有任务编号命名空间，不代表产品名或文件后缀；实现仍为 `fqc/.fqc`。

| 任务 | 建议 change ID | 状态/前置 | 交付结果 | 规格 |
|---|---|---|---|---|
| `FQC-DOC-001`（Rust 部分） | `document-fqc-format-family` | 可建 change，P0 | README 明示 `fqc-indexed/v2` 及不兼容边界 | [01 §4](01-fqc-product-and-format-governance.md#4-任务-fqc-doc-001公开同名共存契约) |
| `FQCR-SPEC-001` | `correct-indexed-v2-spec` | 可建 change，P0 | codec/checksum/version 与真实 bytes 一致并冻结 fixture | [02 §3](02-fq-compressor-rust-hardening.md#3-任务-fqcr-spec-001校准-v2-规范并冻结字节契约) |
| `FQC-FAMILY-001`（Rust 部分） | `recognize-sequential-fqc-family` | 前置：DOC 与 SPEC，P0 | 识别 C++ magic 并专门拒绝 | [01 §5](01-fqc-product-and-format-governance.md#5-任务-fqc-family-001已知格式族的快速拒绝) |
| `FQCR-IO-001` | `make-file-output-atomic` | 可建 change，P0 | 同目录临时文件和原子提交 | [02 §5](02-fq-compressor-rust-hardening.md#5-任务-fqcr-io-001普通文件输出事务) |
| `FQCR-CODEC-001` | `dispatch-all-stream-codecs` | 前置：SPEC，P0 | 四个 stream codec 字段驱动各自 decoder | [02 §4](02-fq-compressor-rust-hardening.md#4-任务-fqcr-codec-001让每个-stream-codec-字段成为解码真相) |
| `FQCR-LIMIT-001` | `enforce-decode-resource-budget` | 前置：SPEC；建议 CODEC 后，P0 | 声明、分配和 reorder 受统一预算约束 | [02 §6](02-fq-compressor-rust-hardening.md#6-任务-fqcr-limit-001解压和校验资源预算) |
| `FQC-IDENTITY-001`（Rust 部分） | `expose-indexed-fqc-identity` | 前置：SPEC，P1 | version/info 展示 Rust indexed 身份 | [01 §6](01-fqc-product-and-format-governance.md#6-任务-fqc-identity-001cli-身份可观察性) |
| `FQC-REG-001`（双仓各自部分） | 随两个 format spec change 落地 | 前置：两仓格式规范与 fixture，P1 | 各 owner 仓维护权威格式登记并双向链接 | [01 §7](01-fqc-product-and-format-governance.md#7-任务-fqc-reg-001分别维护格式登记) |
| `FQCR-CI-001` | `add-rust-quality-gates` | 建议 P0 完成后，P1 | fmt、clippy、test、doc 持续门禁 | [02 §7](02-fq-compressor-rust-hardening.md#7-任务-fqcr-ci-001最小持续集成) |
| `FQCR-RENAME-001` | — | **Not applicable** | 负责人明确保留 `fqc/.fqc`，不得实现 | [01 §9](01-fqc-product-and-format-governance.md#9-撤销记录-fqcr-rename-001) |

### 3.3 `compress-kit`

| 任务 | 建议 change ID | 状态/前置 | 交付结果 | 规格 |
|---|---|---|---|---|
| `CK-DOC-001` + `CK-FMT-001` + `CK-TEST-001` | `version-binary-formats-v2` | **决策已批准，可建 change，P0** | 四个 v2 magic、legacy 明确拒绝、fixture、文档和 2.0.0 语义一次闭环 | [05](05-compress-kit-format-versioning.md) |
| `CK-LIMIT-001` | `reject-oversize-before-allocation` | 可建 change，P1 | 文件整体分配前拒绝超限 | [05 §7](05-compress-kit-format-versioning.md#7-任务-ck-limit-001分配前执行文件大小检查) |
| `CK-IO-001` | `make-file-output-atomic` | 可建 change，P2 | 文件输出失败原子性 | [05 §8](05-compress-kit-format-versioning.md#8-任务-ck-io-001文件输出失败原子性) |
| `CK-V1-READER-001` | — | **Not applicable** | 本轮选择明确拒绝 v1 | [05 §9](05-compress-kit-format-versioning.md#9-条件后续任务-ck-v1-reader-001) |

格式 v2 的 magic、fixture、decoder 错误和文档必须放在同一 change 中，避免仓库处于“代码与公开格式身份不同步”的可合并状态；tasks 内仍按测试优先拆分步骤。

### 3.4 `micos-2024`

| 任务 | 建议 change ID | 状态/前置 | 交付结果 | 规格 |
|---|---|---|---|---|
| `MICOS-DOC-001` | `declare-python-production-orchestrator` | **决策已批准，可建 change，P0** | Python/WDL/resume 能力边界准确 | [03 §4](03-micos-reproducibility.md#4-任务-micos-doc-001修正文档能力边界) |
| `MICOS-CONFIG-001` | `enforce-effective-configuration` | 可建 change，P0 | 未知/未生效配置失败，生成 resolved config | [03 §5](03-micos-reproducibility.md#5-任务-micos-config-001严格且可追踪的有效配置) |
| `MICOS-ENV-001` | `lock-reproducible-toolchain` | 先确认一个已知成功环境，P0 | 单一版本真相、lock、固定镜像/数据库来源 | [03 §6](03-micos-reproducibility.md#6-任务-micos-env-001工具版本与容器供应链闭环) |
| `MICOS-PATH-001` | `make-examples-portable` | 可建 change，P1 | 清除个人绝对路径并增加可运行示例 | [03 §7](03-micos-reproducibility.md#7-任务-micos-path-001清除个人路径建立可运行示例) |
| `MICOS-RUN-001` | `record-run-manifest` | 前置：CONFIG；ORCH 已批准，P1 | run manifest 与可靠 stage 状态 | [03 §8](03-micos-reproducibility.md#8-任务-micos-run-001运行清单与可靠-stage-状态) |
| `MICOS-CI-001` | `layer-workflow-validation` | 建议 DOC/CONFIG/ENV/PATH 后，P1 | PR、定时容器和论文冻结三层验证 | [03 §9](03-micos-reproducibility.md#9-任务-micos-ci-001分层验证) |
| `MICOS-RESUME-001` | — | 后续：RUN 稳定且有真实需求 | signature/fingerprint 恢复语义 | [03 §8](03-micos-reproducibility.md#后续任务-micos-resume-001) |

### 3.5 `minibwa-rust`

| 任务 | 建议 change ID | 状态/前置 | 交付结果 | 规格 |
|---|---|---|---|---|
| `MBW-DOC-001` | `correct-parity-contract-docs` | 可建 change，P1 | 修正里程碑、parity 前提和失效脚本 | [04 §7](04-minibwa-rust-parity-ci.md#7-任务-mbw-doc-001修复里程碑和失效工具) |
| `MBW-REF-001` | `pin-c-reference-source` | **信息阻塞，P0** | 确定性 C 来源 manifest 与 build | [04 §3](04-minibwa-rust-parity-ci.md#3-任务-mbw-ref-001固定-c-参考来源) |
| `MBW-CI-001` | `require-c-parity-in-ci` | 前置：REF，P0 | CI 缺依赖必须失败而非 skip | [04 §4](04-minibwa-rust-parity-ci.md#4-任务-mbw-ci-001强制-parity-job) |
| `MBW-FIXTURE-001` | `freeze-reference-fixtures` | 前置：REF，P1 | 小型冻结输出和比较规则 | [04 §5](04-minibwa-rust-parity-ci.md#5-任务-mbw-fixture-001提交小型冻结兼容样本) |
| `MBW-REL-001` | `gate-release-on-parity` | 前置：CI、FIXTURE，P0 | release 必须依赖 parity | [04 §6](04-minibwa-rust-parity-ci.md#6-任务-mbw-rel-001让发布依赖-parity) |

### 3.6 `fastq-tools`

| 任务 | 建议 change ID | 状态/前置 | 交付结果 | 规格 |
|---|---|---|---|---|
| `FQT-META-001` | `correct-project-metadata` | 可建 change，P1 | 组织归属和 release 展示符合事实 | [06 §6](06-fastq-tools-release-portability.md#6-任务-fqt-meta-001修正归属和发布真相) |
| `FQT-CPU-001` | `define-cpu-build-profiles` | 可建 change，P0 | `portable/x86-64-v3/native` 显式 profile，默认不要求 v3 | [06 §3](06-fastq-tools-release-portability.md#3-任务-fqt-cpu-001显式-cpu-profile) |
| `FQT-TOOLCHAIN-001` | `align-release-toolchain` | 可建 change，P1 | CI/release 使用单一受控版本表 | [06 §5](06-fastq-tools-release-portability.md#5-任务-fqt-toolchain-001消除-release-toolchain-漂移) |
| `FQT-REL-001` | `add-portable-release-workflow` | 前置：CPU、META；建议 TOOLCHAIN，P1 | 保守且受门禁的 release workflow | [06 §4](06-fastq-tools-release-portability.md#4-任务-fqt-rel-001建立保守的发布闭环) |

### 3.7 组织级要求的逐仓落地

| 任务 | change 方式 | 状态 | 规格 |
|---|---|---|---|
| `ORG-GOV-001` | 每仓首个 change 建立最小 `openspec/` | Accepted | [08 §3](08-organization-engineering-baseline.md#3-决策-org-gov-001仓库本地规格治理) |
| `ORG-LIFE-001` | 每仓 `document-project-lifecycle` | owner/support 信息阻塞 | [08 §4](08-organization-engineering-baseline.md#4-任务-org-life-001公开项目状态与维护责任) |
| `ORG-CONTRACT-001` | 在对应功能 change 中建立主规格 | 可随各仓首批 change 实施 | [08 §5](08-organization-engineering-baseline.md#5-任务-org-contract-001建立仓库内外部契约规格) |
| `ORG-META-001` | 每仓独立 `correct-project-metadata` | 可建 change，P1 | [08 §6](08-organization-engineering-baseline.md#6-任务-org-meta-001修复活动仓库身份) |
| `ORG-SEC-001` | 每仓继承组织 policy，必要时覆盖 | 联系方式阻塞，P2 | [08 §10](08-organization-engineering-baseline.md#10-任务-org-sec-001统一安全报告入口) |

## 4. 推荐首轮 change

每个仓库先落一个高价值、边界清晰的 change；不要一次把全部任务变成活动 change：

| 顺序 | 仓库 | 首轮 change | 理由 |
|---:|---|---|---|
| 1 | `fq-compressor-rust` | `correct-indexed-v2-spec` | 当前规范与 bytes 冲突，是后续 family/codec/limit 的基础 |
| 2 | `compress-kit` | `version-binary-formats-v2` | 决策已批准，必须把 magic、fixture、decoder 和文档一次闭环 |
| 3 | `micos-2024` | `declare-python-production-orchestrator` | 先消除科研工作流能力误述 |
| 4 | `minibwa-rust` | `correct-parity-contract-docs` | 在参考 commit 缺失时先如实公开 CI 会 skip |
| 5 | `fastq-tools` | `define-cpu-build-profiles` | 修复发布二进制可移植性承诺 |
| 6 | `fq-compressor` | `freeze-sequential-v2-format` | 为跨族识别和稳定 release 建立 fixture 基线 |

两个 FQC 的 `document-fqc-format-family` 可以分别合入各自首轮 change 的文档部分，但不能让一个仓库的模型修改另一个仓库。

## 5. 实现模型领取模板

```text
目标仓库：<绝对路径>
唯一 change：openspec/changes/<change-id>/
关联任务：<TASK-ID>
批准决策：<decision ID 和精确值>

你可以修改目标仓库代码，但只能应用这个 change。开始前完整阅读仓库 AGENTS.md、openspec/project.md、proposal.md、design.md、tasks.md 和 delta spec；检查并保留已有工作区修改；先运行任务规定的基线验证。
逐项实现 tasks，并在每项后执行对应验证。不得处理别的 change、跨仓修改、全局重构、无关升级、提交、推送、开 PR 或发布。
发现 spec 与当前代码事实冲突时停止 apply，报告文件/行号证据并回到 proposal/spec 修订；不得自行创造新产品决策。
完成时报告 change ID、修改文件、行为和兼容影响、所有命令及退出状态、未执行项、剩余风险、git diff --check 与 git status --short。所有验收场景通过前不得 archive。
```

## 6. 归档完成记录

每个仓库在 change 归档时保留：

```text
Change: <change-id>
Tasks: <TASK-IDs>
Decision records: <IDs>
Implemented commit/PR: <SHA or link>
Validation evidence: <commands and CI links>
Reviewed by: <owner>
Archived at: <date>
Residual risk: <short text>
```

只有代码、主规格、验证证据与实现事实一致，活动 change 才能移入 `openspec/changes/archive/YYYY-MM-DD-<change-id>/`。
