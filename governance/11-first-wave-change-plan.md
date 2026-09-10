# 首轮 OpenSpec Change 建立计划

## 1. 首轮目标

首轮不是让一个模型修完所有仓库，而是让六个独立仓库各自拥有一个高价值、可评审、可验证的 change。建议分两阶段：

1. **Propose 阶段**：六仓可并行，只创建 `openspec/` artifacts，不改产品代码；
2. **Apply 阶段**：逐仓明确授权后实施；同仓一次一个 change。

每个 Propose 模型都必须以当前仓库 HEAD 为 base，若已偏离审计提交，先重新核对相关事实并在 proposal 记录差异。

## 2. 第一批 change

### 2.1 `fq-compressor-rust/correct-indexed-v2-spec`

关联：`FQCR-SPEC-001`、`ORG-GOV-001`、`ORG-CONTRACT-001`

capability：`archive-format`

proposal 必须覆盖：

- 当前公开 codec 表与 `(family << 4) | version` 实现不一致；
- checksum ID `0` 的真实含义；
- reader 只接受 major 2，不存在文档声称的 v1 fallback；
- 本 change 不改 `fqc/.fqc`、magic 或压缩算法；
- 本 change 建立 `fqc-indexed/v2` 权威规格和冻结 fixture。

delta spec 至少包含 requirements：

- Indexed archive identity；
- Version compatibility；
- Codec identifier encoding；
- Checksum identifier encoding；
- Frozen decoder fixture；
- Unknown identifier rejection。

tasks 顺序：结构测试/fixture → 修正文档 → 错误/边界测试 → Rust 标准门禁。不要同时实施 codec dispatch、资源预算或产品 identity change。

### 2.2 `compress-kit/version-binary-formats-v2`

关联：`CK-DEC-001`、`CK-DOC-001`、`CK-FMT-001`、`CK-TEST-001`、`ORG-CONTRACT-001`

capability：`binary-formats`

已批准值：`HFM2/AEN2/RCN2/RLE2`；writer 只写 v2；可识别 v1 明确拒绝；v1 RLE 返回 bad magic；发布语义 `2.0.0`。

proposal 必须把以下内容视为一个原子格式迁移：新 magic、decoder 分类、v1/v2 fixture、格式文档、changelog 和版本来源。不得把“先改 writer、以后补 fixture”拆成可独立合并状态。

delta spec 至少包含：

- Four v2 wire identities；
- CRC coverage and endian；
- Legacy rejection；
- Truncated/corrupt behavior；
- Raw/compressed size boundary；
- Frozen format fixtures；
- Release major consistency。

tasks 顺序：生成可信 v1 fixture → 添加预期失败测试 → 修改 magic/parser → 生成 v2 fixture → 文档/版本 → 完整 `make lint && make test`。

### 2.3 `micos-2024/declare-python-production-orchestrator`

关联：`MICOS-ORCH-001`、`MICOS-DOC-001`、`ORG-GOV-001`

capability：`workflow-orchestration`

已批准值：Python CLI 是唯一生产编排器；shell 是薄包装；现有 WDL 是实验性单步骤参考；当前不承诺 resume。

本 change 首轮只修正对外契约与仓库规格，不实现 run manifest、resume、顶层 WDL 或环境重建。

delta spec 至少包含：

- Production entry point；
- Shell wrapper behavior；
- Experimental WDL status；
- Unsupported resume behavior；
- Documentation consistency。

tasks 必须包含全仓搜索 WDL/resume/production 声明，以及 README、docs、CLI help、WDL 目录说明的一致性检查。

### 2.4 `minibwa-rust/correct-parity-contract-docs`

关联：`MBW-DOC-001`、`ORG-GOV-001`

capability：`reference-parity`

当前仓库已声明参考 URL 为 `https://github.com/lh3/minibwa`，但尚无可证明的已验证 commit、构建基线和 fixture 来源。因此本 change 只能如实记录：本地缺参考依赖会 skip、当前 CI 未证明 parity、脚本接口已经漂移。不得发明 commit，不得在本 change 声称修复 CI 强制性。

delta spec 至少包含：

- Scope of current parity claim；
- Observable skip conditions；
- Documentation truthfulness；
- Comparison script supported interface。

若修复 `scripts/compare.sh` 需要确定 C 参考来源，任务应停在 Blocked，不用另一个 `bwa` 或假 fixture 替代。

### 2.5 `fastq-tools/define-cpu-build-profiles`

关联：`FQT-CPU-001`、`ORG-GOV-001`

capability：`build-portability`

目标 profiles：

| profile | 编译要求 | 用途 |
|---|---|---|
| `portable` | 不添加 `-march=x86-64-v3` 或 `-march=native` | 默认本地和发布基线 |
| `x86-64-v3` | 显式添加 v3 | 已知部署环境优化 |
| `native` | 显式添加 native | 本机 benchmark，不发布 |

delta spec 至少包含：默认 profile、显式 opt-in、artifact 命名/元数据、unsupported CPU 行为和测试可观察性。当前不引入运行时 SIMD dispatch。

tasks 顺序：CMake 配置测试 → profile 实现 → CI/preset 覆盖 → README → portable artifact 指令检查和 smoke。

### 2.6 `fq-compressor/freeze-sequential-v2-format`

关联：`FQC-CPP-FMT-001`、`ORG-GOV-001`、`ORG-CONTRACT-001`

capability：`archive-format`

目标是建立 `fqc-sequential/v2` 当前真相，不改变现有 magic/layout。proposal 必须记录审计提交和已发布/RC fixture 的可信来源。

delta spec 至少包含：

- Sequential archive identity；
- Header/frame/footer sizes and endian；
- Decoder version/codec rejection；
- Header/frame/footer checksum behavior；
- Frozen valid archives；
- Deterministic corrupt cases；
- Compatibility promise boundary。

tasks 顺序：规范 → fixture manifest → current-reader tests → 损坏矩阵 → sanitizer-compatible corpus → 文档。不要顺手实现 Rust indexed family rejection；那是后续 `recognize-indexed-fqc-family`。

## 3. 第二批依赖图

```text
fq-compressor-rust:
  correct-indexed-v2-spec
    ├─> recognize-sequential-fqc-family
    ├─> dispatch-all-stream-codecs
    ├─> enforce-decode-resource-budget
    └─> expose-indexed-fqc-identity

fq-compressor:
  freeze-sequential-v2-format
    ├─> recognize-indexed-fqc-family
    ├─> expose-sequential-fqc-identity
    └─> add-sanitizer-contract-gate

compress-kit:
  version-binary-formats-v2
    ├─> reject-oversize-before-allocation
    └─> make-file-output-atomic

micos-2024:
  declare-python-production-orchestrator
    └─> enforce-effective-configuration
          └─> record-run-manifest

minibwa-rust:
  [维护者确认 C commit + build/fixture baseline]
    └─> pin-c-reference-source
          ├─> require-c-parity-in-ci
          └─> freeze-reference-fixtures
                └─> gate-release-on-parity

fastq-tools:
  define-cpu-build-profiles
    └─> add-portable-release-workflow
          └─> align-release-toolchain（也可提前独立完成）
```

## 4. Propose 模型统一提示词

```text
你当前只做 OpenSpec Propose，不修改产品源码、测试、构建配置或 CI。

目标仓库：<repo absolute path>
change ID：<change-id>
关联任务：<task IDs>
输入设计：<maintenance-design document/section>
批准决策：<decision IDs and exact values>

1. 完整阅读目标仓库 AGENTS.md，并检查 git status 与 HEAD。
2. 只读核对设计中的关键事实；若 HEAD 已变化，在 proposal 写明差异。
3. 在目标仓库建立最小 openspec/project.md、openspec/AGENTS.md，以及唯一 change 的 proposal.md、design.md、tasks.md、verification.md、specs/<capability>/spec.md。
4. change spec 使用 ADDED/MODIFIED/REMOVED/RENAMED Requirements；每个 requirement 使用 SHALL 和可测试 GIVEN/WHEN/THEN 场景。
5. proposal 明确 From/To/Reason/Impact、允许修改范围、非目标、兼容、回滚和前置。
6. tasks 按基线/失败测试/最小实现/文档/完整验证排序，每项关联命令或 scenario。
7. verification.md 只创建待填写矩阵，不伪造结果。
8. 最终状态设为 Proposed，然后停止。不得 apply、commit、push、开 PR 或 archive。
```

## 5. Apply 模型统一提示词

使用 [任务索引中的领取模板](09-decision-register-and-task-index.md#5-实现模型领取模板)。额外要求：若 change 状态不是 Approved，立即停止；若用户授权只说“实施某 change”，权限只覆盖该仓库和该 change，不覆盖 commit/push/release。

## 6. Verify 模型统一提示词

```text
你是独立 verifier，不假设 tasks 的勾选可信，也不扩大实现范围。

目标仓库：<repo absolute path>
change ID：<change-id>

从 delta spec 逐条建立 requirement/scenario 到测试或命令的映射；审查实际 diff、失败副作用、兼容性和文档一致性；运行 change 与 AGENTS.md 要求的全部门禁；填写 verification.md。
任何必需场景无证据、命令失败、diff 越界或主规格提前描述未来行为时，将 Ready to archive 设为 no，并列出最小返工项。不要自行提交、推送、发布或归档。
```

## 7. Archive 模型统一提示词

```text
只归档 <change-id>。开始前确认 verification.md 为 Ready to archive: yes、所有任务完成、评审人已确认且工作区只包含本 change。
把 delta 按 requirement 标题同步进 openspec/specs/<capability>/spec.md，确保主规格只描述已实现行为；将完整 change 移入 openspec/changes/archive/YYYY-MM-DD-<change-id>/；再运行规格校验、git diff --check 和最小 smoke。
若遇到 delta 冲突、缺少验证或实现与 spec 不一致，停止归档并报告。不得提交、推送、开 PR 或发布，除非另获明确授权。
```
