# PLAN: open-genomics 设计文档价值评估与执行计划

> ## ⚠️ 本文件整份已过时（2026-09-10 核对）
>
> 这是 2026-08-19 写给当时下一个执行模型的交接计划，**其中的状态清单与执行步骤
> 均已失效或不必要**，保留仅为追溯。具体：
>
> - **「二、当前状态清单」B 节**列出 8 个 change「待归档」，其中 5 个声称
>   「因环境缺 Conan 无法归档」。2026-09-10 核对：**这 5 个全部早已归档**，
>   verification 均为 `Ready to archive: yes`；且 Conan 2.31.2 在本机可用，
>   「缺 Conan」的阻塞前提不成立。
> - **「阶段 1：收尾 minibwa-rust」** 已完成（改动已提交推送）。
> - **「三、执行计划」与「四、给 Flash 模型的交接 Prompt」** 已执行完毕，
>   **请勿再按此派工**。
> - 当时审计范围为 6 个仓库，现工作区有 12 个（见 [README.md](README.md) 对照表）。
>
> 仍然有效的内容见 [README.md](README.md) 的「仍然有效、可直接使用的部分」。

## 一、设计文档价值评估

### 结论：高价值，值得继续完成

`maintenance-design/` 下的 11 份设计文档 + 模板 + 变更草案构成了一套罕见的"可执行的工程治理包"：

| 维度 | 评价 |
|------|------|
| **问题定位准确度** | 高。每个仓库的问题都基于实际审计提交的代码证据，不是泛化的最佳实践清单 |
| **风险排序合理** | P0 先阻断契约冲突和数据风险（format magic、规范不一致、原子输出），P1 建立可复现性，P2 处理发布质量 |
| **依赖关系清晰** | 09 号文档的任务矩阵 + 11 号文档的依赖图让并行和串行一目了然 |
| **可执行性** | 10 号文档定义了一套完整的 proposal→spec→design→tasks→apply→verify→archive 工作流，每个步骤都有输入/输出/门禁 |
| **决策记录完整** | 4 个关键决策已由负责人确认，消除了最大的不确定性 |
| **模板和提示词** | 提供了 Propose/Apply/Verify/Archive 四个阶段的统一提示词，适合分发给便宜模型执行 |

### 不足

1. 部分 verification 标注"Ready to archive: yes"但缺乏独立验证（同一 agent 自评）
2. 两个 change 的 build/test 因环境缺 Conan 而 deferred，严格按工作流规则不能归档
3. 信息阻塞项（minibwa C 参考 commit、owner/安全联系方式）需要维护者提供

---

## 二、当前状态清单

### A. 已完成且已归档（无需处理）

| 仓库 | Change | 状态 |
|------|--------|------|
| compress-kit | `version-binary-formats-v2` | ✅ Archived |
| fq-compressor-rust | `correct-indexed-v2-spec` | ✅ Archived |
| fq-compressor-rust | `dispatch-all-stream-codecs` | ✅ Archived |
| fq-compressor-rust | `enforce-compress-archive-budget` | ✅ Archived |
| fq-compressor-rust | `enforce-decode-resource-budget` | ✅ Archived |
| fq-compressor-rust | `make-file-output-atomic` | ✅ Archived |
| fq-compressor-rust | `recognize-sequential-fqc-family` | ✅ Archived |

### B. 已完成但未归档（需要归档操作）

| 仓库 | Change | verification 状态 | 阻塞原因 |
|------|--------|-------------------|----------|
| fq-compressor | `correct-verify-contract` | Completed（无 Ready to archive 行） | 需补填 verification 并归档 |
| fq-compressor | `add-ci-sanitizer-gate` | Completed（远程 CI 已绿） | 需补填 Ready to archive 并归档 |
| fq-compressor | `freeze-sequential-v2-format` | Ready to archive: yes | 可直接归档 |
| fq-compressor | `recognize-indexed-fqc-family` | Completed（16/16 测试通过） | 需补填 Ready to archive 并归档 |
| fq-compressor-rust | `complete-id-and-qvz-modes` | Ready to archive: yes | 可直接归档 |
| micos-2024 | `declare-python-production-orchestrator` | Ready to archive: yes | 可直接归档 |
| fastq-tools | `define-cpu-build-profiles` | Ready to archive: yes（但 build/test 缺 Conan 未跑） | **严格按工作流规则不能归档**，需在 Conan 环境补跑 |
| fq-compressor | `freeze-sequential-v2-format` | Ready to archive: yes（但 build/test 同样缺 Conan） | **同上**（注：该 change 的 verification 显示有 Conan 环境，需核实） |

### C. 进行中（minibwa-rust 收尾计划）

当前 minibwa-rust 工作区有未提交修改，对应 `docs/superpowers/plans/2026-08-18-final-wrapup.md`：

| 任务 | 状态 | 说明 |
|------|------|------|
| Task 1: compare.sh 退出码 | ✅ 已改 | `scripts/compare.sh` 已添加 `exit 1` |
| Task 2: 归档 change | ✅ 已 git mv | `correct-parity-contract-docs/` 已移至 archive/ |
| Task 3: README 更新 | ✅ 已改 | 已知限制表、最终收尾行、功能完备声明 |
| Task 4: 最终验证 | ❌ 未执行 | `cargo fmt --check` / `clippy` / `test` / `compare.sh` 全部未跑 |
| 额外改动 | ⚠️ 需核查 | `src/index.rs` 有函数签名格式化改动（多行→单行），不在原计划范围内 |
| 提交 | ❌ 未提交 | 全部改动未 commit |

### D. 尚未启动的 change（按优先级）

**P0（可立即建 change）：**
- fq-compressor + fq-compressor-rust: `document-fqc-format-family`（两个仓库分别做，README 添加同名共存说明）
- micos-2024: `enforce-effective-configuration`（严格配置校验）
- micos-2024: `lock-reproducible-toolchain`（需先确认已知成功环境）
- compress-kit: `reject-oversize-before-allocation`（P1 但可建）

**P1（前置条件已满足）：**
- fq-compressor: `expose-sequential-fqc-identity`（前置 FMT 已完成）
- fq-compressor-rust: `expose-indexed-fqc-identity`（前置 SPEC 已完成）
- fq-compressor: `add-archive-fuzz-smoke`（前置 FMT+CI 已完成）
- fq-compressor: `correct-project-metadata`（可先修部分）
- fq-compressor-rust: `add-rust-quality-gates`
- micos-2024: `make-examples-portable`
- micos-2024: `record-run-manifest`（前置 CONFIG）
- fastq-tools: `correct-project-metadata`
- fastq-tools: `align-release-toolchain`
- fastq-tools: `add-portable-release-workflow`（前置 CPU+META）

**信息阻塞（需维护者提供）：**
- minibwa-rust: `pin-c-reference-source` 及下游 MBW-CI/FIXTURE/REL（缺 C 参考 commit、构建命令、fixture 来源）
- 所有仓库: `document-project-lifecycle`（缺 owner、支持平台、release channel）
- 所有仓库: `ORG-SEC-001`（缺安全联系方式）

---

## 三、执行计划（按优先级排序）

### 阶段 1：收尾 minibwa-rust（最紧急，唯一有半成品工作区的仓库）

**步骤 1.1** 核查 `src/index.rs` 改动
- 确认这个格式化改动是否在计划内，还是意外修改
- 如果只是 cargo fmt 导致的格式化，保留；如果是手动改的，评估是否应回退

**步骤 1.2** 执行 Task 4 最终验证
```bash
cd /home/shane/github/open-genomics/minibwa-rust
cargo fmt --all -- --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test --all-targets --all-features
./scripts/compare.sh data/toy.fa data/toy_reads.fq
```

**步骤 1.3** 清理临时文件
- 检查是否有 compare_c.l2b、compare_c.mbw、compare_r.l2b、compare_r.mbw、minibwa_c.sam、minibwa_rust.sam 等临时文件
- 若存在，删除或加入 .gitignore

**步骤 1.4** 提交所有改动
- 提交信息应包含 Task 1-3 的说明

### 阶段 2：归档已完成的 change

**步骤 2.1** 直接归档（verification 完整，Ready to archive: yes）
- fq-compressor: `freeze-sequential-v2-format`
- fq-compressor-rust: `complete-id-and-qvz-modes`
- micos-2024: `declare-python-production-orchestrator`

**步骤 2.2** 补填 verification 后归档
- fq-compressor: `correct-verify-contract`（补 Ready to archive 行）
- fq-compressor: `add-ci-sanitizer-gate`（补 Ready to archive 行）
- fq-compressor: `recognize-indexed-fqc-family`（补 Ready to archive 行）

**步骤 2.3** 需补跑 build/test 后归档
- fastq-tools: `define-cpu-build-profiles`（需 Conan 环境）
- fq-compressor: `freeze-sequential-v2-format`（verification 显示已有 Conan 环境，但需核实）

### 阶段 3：启动 P0 未建 change

按仓库独立并行，每个仓库一个 change：

1. **fq-compressor** + **fq-compressor-rust**: 分别建 `document-fqc-format-family`
   - 两个仓库 README 添加同名共存对照表
   - 明确 magic 区分、不能互相解码
   - 跨仓库任务，但必须拆成两个独立 change、两个独立提交

2. **micos-2024**: 建 `enforce-effective-configuration`
   - 未知/未生效配置必须失败
   - 生成 resolved config 输出

3. **micos-2024**: 建 `lock-reproducible-toolchain`
   - 需先确认一个已知成功环境
   - 锁定 Python/容器/数据库版本

4. **compress-kit**: 建 `reject-oversize-before-allocation`
   - 文件整体分配前检查大小上限

### 阶段 4：启动 P1 change（前置条件已满足）

可在阶段 3 完成后或并行启动（不同仓库可并行）：
- fq-compressor: `expose-sequential-fqc-identity`
- fq-compressor-rust: `expose-indexed-fqc-identity`
- fq-compressor: `add-archive-fuzz-smoke`
- fq-compressor: `correct-project-metadata`
- fq-compressor-rust: `add-rust-quality-gates`
- micos-2024: `make-examples-portable`
- fastq-tools: `correct-project-metadata`
- fastq-tools: `align-release-toolchain`

### 阶段 5：解除信息阻塞后启动

需要维护者提供：
- minibwa C 参考 commit、构建命令、fixture 来源 → 解锁 MBW-REF-001 及下游全部
- 各仓库 owner、支持平台、release channel → 解锁 ORG-LIFE-001
- 安全联系方式 → 解锁 ORG-SEC-001

---

## 四、给 Flash 模型的交接 Prompt

以下是可以直接发给下一个模型的执行指令：

---

> **角色：** 你是 open-genomics 组织的工程执行 agent。你的任务是根据本计划逐项完成收尾和归档工作。
>
> **工作区根目录：** `/home/shane/github/open-genomics`
>
> **重要规则：**
> - 每个仓库独立操作，不要跨仓库修改
> - 不要提交、推送或发布，除非本计划明确要求
> - 先读目标仓库的 `AGENTS.md` 再操作
> - 每步完成后验证，验证失败就停止并报告
>
> ---
>
> ### 第一步：收尾 minibwa-rust（最高优先级）
>
> 1. 进入 `/home/shane/github/open-genomics/minibwa-rust`
> 2. 查看 `git status` 和 `git diff` 确认当前改动
> 3. 阅读 `docs/superpowers/plans/2026-08-18-final-wrapup.md` 了解上下文
> 4. 检查 `src/index.rs` 的改动：如果只是函数签名从多行变成单行（cargo fmt 风格），保留；如果是其他改动，报告
> 5. 检查是否有临时文件（`compare_c.l2b`, `compare_c.mbw`, `compare_r.l2b`, `compare_r.mbw`, `minibwa_c.sam`, `minibwa_rust.sam`），有则删除
> 6. 执行 Task 4 验证：
>    ```bash
>    cargo fmt --all -- --check
>    cargo clippy --all-targets --all-features -- -D warnings
>    cargo test --all-targets --all-features
>    ./scripts/compare.sh data/toy.fa data/toy_reads.fq
>    ```
> 7. 如果全部通过，执行 `git add -A && git commit -m "chore: final wrap-up - compare.sh exit code, archive change, README update"`
>
> ---
>
> ### 第二步：归档已完成的 change
>
> 对于以下每个 change，执行归档操作：
>
> **归档操作定义：**
> 1. 进入目标仓库
> 2. 确认 `openspec/changes/<change-id>/verification.md` 中 `Ready to archive: yes` 存在
> 3. 如果 verification 不完整，先补填（添加 `Ready to archive: yes` 行和日期）
> 4. 将 delta spec 合并进 `openspec/specs/<capability>/spec.md`（如果主 spec 尚不存在，创建之）
> 5. 将 change 目录移入 `openspec/changes/archive/YYYY-MM-DD-<change-id>/`
> 6. 更新 proposal.md 的 Status 为 `Archived`
> 7. 提交（每个仓库一个 commit）
>
> **归档清单：**
>
> | 仓库 | Change ID | Capability | 备注 |
> |------|-----------|------------|------|
> | fq-compressor | `correct-verify-contract` | `archive-format` | 先补 Ready to archive |
> | fq-compressor | `add-ci-sanitizer-gate` | `ci-quality` | 先补 Ready to archive |
> | fq-compressor | `freeze-sequential-v2-format` | `archive-format` | 可直接归档 |
> | fq-compressor | `recognize-indexed-fqc-family` | `archive-format` | 先补 Ready to archive |
> | fq-compressor-rust | `complete-id-and-qvz-modes` | `compression-codecs` | 可直接归档 |
> | micos-2024 | `declare-python-production-orchestrator` | `workflow-orchestration` | 可直接归档 |
>
> **暂缓归档（需 Conan 环境）：**
> - fastq-tools/`define-cpu-build-profiles`：verification 标注 build/test 缺 Conan 未跑，严格按工作流规则不能归档
> - 如果当前环境有 Conan，补跑 `./scripts/core/build --preset clang-release && ./scripts/core/test --preset clang-release` 后再归档
>
> ---
>
> ### 第三步：启动 P0 未建 change（如果有余力）
>
> 按 [11-first-wave-change-plan.md](11-first-wave-change-plan.md) 中的 Propose 提示词，为以下仓库建立 change artifacts（只建 proposal/spec/design/tasks，不改代码）：
>
> 1. **fq-compressor** 建 `document-fqc-format-family`（规格见 [01 号文档 §4](01-fqc-product-and-format-governance.md#4-任务-fqc-doc-001公开同名共存契约)）
> 2. **fq-compressor-rust** 建 `document-fqc-format-family`（同上）
> 3. **micos-2024** 建 `enforce-effective-configuration`（规格见 [03 号文档 §5](03-micos-reproducibility.md#5-任务-micos-config-001严格且可追踪的有效配置)）
>
> 每个 change 建完后状态设为 `Proposed`，不要 apply。
>
> ---
>
> ### 完成报告
>
> 完成后报告每步的执行结果：
> - 哪些步骤成功
> - 哪些步骤失败（附错误信息）
> - 哪些步骤跳过（附原因）
> - 最终每个仓库的 `git status --short`