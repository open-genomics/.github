# 仓库本地 OpenSpec 风格工作流

## 1. 目标与选择

这套流程把“负责人意图”变成便宜模型可执行、评审者可验证、Git 历史可追溯的 change 包。它采用 OpenSpec 的核心模型：

```text
explore → propose → specs → design → tasks → apply → verify → archive
```

每个 change 是一个自包含目录；`openspec/specs/` 描述当前已经实现的行为，`openspec/changes/` 描述尚未归档的变化。官方 OpenSpec 也以 proposal、specs、design、tasks 组织 change，并在归档时把 delta 合并回主规格。本方案增加必需的 `verification.md`，因为低成本实现模型尤其需要明确留下验证证据。

本方案不要求安装 OpenSpec CLI。纯 Markdown 结构即可工作；以后若选择官方 CLI，应在每个子仓库分别初始化和验证，不在工作区根目录初始化。官方参考：

- <https://github.com/Fission-AI/OpenSpec>
- <https://github.com/Fission-AI/OpenSpec/blob/main/docs/overview.md>
- <https://github.com/Fission-AI/OpenSpec/blob/main/openspec/specs/openspec-conventions/spec.md>

### 1.1 本组织采用的轻量 profile

部分仓库曾主动删除完整 OpenSpec/编辑器/Node 元工具，`fq-compressor-rust/AGENTS.md` 还明确反对重型 spec-driven 管理。本轮负责人决定不是恢复那套工具链，而是采用以下最小 agreement layer：

- 只提交纯 Markdown `openspec/` artifacts；
- 不要求 Node、全局 CLI、dashboard 或 docs site；
- 不生成 `.claude/`、`.cursor/` 等工具专属目录；
- 不恢复通用治理样板；
- 只对格式、兼容、安全/资源、科研可复现、CI/release 等高风险 change 强制使用；
- 小型低风险修复仍可直接走仓库既有流程；
- proposal/design/tasks 在 archive 后保留历史，主规格保持紧凑。

因此首个 Rust change 可以在同一文档提交中，把根 `AGENTS.md` 的 blanket prohibition 收窄为上述 lightweight policy；不得借机改写其他项目定位或引入工具依赖。历史 CHANGELOG 中“曾移除旧 OpenSpec 系统”的记录继续保留，因为那是事实；新条目说明本轮引入的是不同的轻量约定。

## 2. 多仓库边界

`/home/shane/github/open-genomics` 是六个 Git 仓库的父目录，不是 monorepo。规则如下：

1. `openspec/` 只创建在目标子仓库；
2. 一个 change 只修改一个 Git 仓库；
3. 跨仓决定使用同一 decision ID，在相关仓库分别记录本仓约束；
4. 一个实现模型不得从根目录批量修改多个仓库；
5. 每个仓库独立 branch、diff、测试、评审、commit 和 archive；
6. 根目录 `maintenance-design/` 是输入快照，不是项目构建依赖或长期规范来源；
7. 不通过 submodule、共享脚本或运行时下载把六仓重新耦合成伪 monorepo。

FQC 是典型跨仓决定：两个仓库都记录 `FQC-DEC-001`，但 C++ change 只实现 sequential reader 的责任，Rust change 只实现 indexed reader 的责任。

## 3. 仓库目录

每个仓库按需建立：

```text
openspec/
  project.md
  AGENTS.md
  config.yaml                 # 可选；使用官方 CLI 时启用
  specs/
    <capability>/
      spec.md                 # 已实现、已验证的当前真相
      design.md               # 可选；长期架构约束
  changes/
    <change-id>/
      proposal.md             # 为什么改、从什么到什么、影响谁
      design.md               # 如何实现、选择与取舍
      tasks.md                # 有顺序和验证点的实施清单
      verification.md         # 本方案要求；apply 前为空模板，verify 后填证据
      specs/
        <capability>/
          spec.md             # ADDED/MODIFIED/REMOVED/RENAMED delta
    archive/
      YYYY-MM-DD-<change-id>/
```

根目录已有 `AGENTS.md` 时，它对整个仓库保持最高的项目内优先级。`openspec/AGENTS.md` 只补充规格流程，不得重复或弱化构建、安全、编辑范围等既有要求。

## 4. Artifact 契约

### 4.1 `project.md`

只保存稳定上下文：项目定位、生命周期、语言/工具链、标准验证命令、核心契约、非目标、决策索引。不要把活动 change 的计划复制进去。

必须包含：

- canonical repository；
- lifecycle 与 release channel；
- 当前审计/base commit；
- authoritative build/test/lint 命令；
- 外部契约及其 capability 路径；
- 与其他仓库的边界；
- “模型不得自动提交、推送、发布”的权限规则。

### 4.2 `proposal.md`

回答 why、what、impact，不写函数级实现。顶部必须有：

```text
Change ID
Status: Draft | Proposed | Approved | Applying | Verifying | Ready to archive | Blocked
Repository
Base commit
Task IDs
Decision IDs
Owner/reviewer
```

每项行为变化用明确的 from/to：

```text
**行为名称**
- From: 当前可验证事实
- To: 目标行为
- Reason: 修改原因
- Impact: breaking/non-breaking、受影响用户与文件
```

还必须列出 scope、out of scope、兼容性、回滚方法、依赖和未知项。存在未知外部值时状态只能是 Blocked，不能用占位值进入 apply。

### 4.3 change `spec.md`

spec 只写可观察行为，不写类名、函数拆分或具体库选择。使用：

```markdown
## ADDED Requirements

### Requirement: 精确且唯一的名称
系统 SHALL ...

#### Scenario: 可验证场景
- **GIVEN** 初始条件
- **WHEN** 发生动作
- **THEN** 得到结果
- **AND** 额外约束
```

修改既有 requirement 时使用 `## MODIFIED Requirements`，并复制修改后的完整 requirement；删除使用 `## REMOVED Requirements` 并写原因和迁移；重命名使用 `## RENAMED Requirements`。requirement 标题在同一 capability 内必须唯一且稳定。

格式、CLI、schema 等外部契约至少覆盖：正常路径、边界值、损坏/错误路径、兼容行为、失败时副作用和资源边界。

### 4.4 `design.md`

design 写 HOW，至少包括：

- 当前代码证据（文件、符号、必要行号）；
- 目标数据流或状态转换；
- 决策与被否决方案；
- 允许修改的文件/模块；
- 兼容与迁移；
- 失败原子性、资源和安全考虑；
- 测试/fixture 设计；
- 风险和回滚。

简单纯文档 change 可以省略 design，但二进制格式、资源限制、科研 provenance、CI/release 和跨仓契约不能省略。

### 4.5 `tasks.md`

任务必须小到可以逐项验证，但不能退化成逐行编码指令：

```markdown
## 1. Baseline

- [ ] 1.1 记录工作区状态和基线命令结果
- [ ] 1.2 添加会证明缺失行为的测试/fixture

## 2. Implementation

- [ ] 2.1 实现最小行为变化
- [ ] 2.2 更新用户文档和 changelog

## 3. Verification

- [ ] 3.1 运行专项测试
- [ ] 3.2 运行仓库标准门禁
- [ ] 3.3 运行 `git diff --check` 并审查 scope
```

每项 checkbox 后应写验证命令或关联 scenario。只有命令实际成功后才能勾选；“代码看起来正确”不是完成证据。

### 4.6 `verification.md`

这是本组织的强制扩展，记录：

- 验证时 HEAD/base 与工作区状态；
- requirement → test/command 的追踪矩阵；
- 每条命令、退出状态和结果摘要；
- 未运行项及原因；
- diff scope 审计；
- 已知剩余风险；
- verifier 与日期；
- `Ready to archive: yes/no`。

不得粘贴大量日志；保存能够复现结论的命令和 CI URL。任何必需命令未运行或失败时只能写 `no`。

## 5. Change 状态机

```text
Draft
  │ proposal/spec/design/tasks 齐全
  ▼
Proposed
  │ 负责人确认范围、行为和 breaking 决策
  ▼
Approved
  │ 明确授权修改代码
  ▼
Applying ───────► Blocked
  │ tasks 完成       │ 新事实导致规格不成立
  ▼                 └─ 回到 Proposed，修订后重新批准
Verifying
  │ 独立核对所有 scenario、命令和 diff
  ▼
Ready to archive
  │ 主规格同步且评审确认
  ▼
Archived
```

禁止的跳转：Draft 直接 Apply、失败测试下 Ready、未同步主规格就 Archive、把 Blocked 任务勾成完成。

## 6. 六个动作

### 6.1 Explore

只读检查代码、测试、历史和当前文档，输出证据与边界。Explore 不创建未来承诺，也不修改代码。若设计文档已给出完整证据，可以缩短但不能跳过工作区状态和基线确认。

### 6.2 Propose

规格模型在一个目标仓库中：

1. 阅读根 `AGENTS.md` 与相关设计；
2. 检查 base commit 和已有修改；
3. 建立/更新 `openspec/project.md`；
4. 创建一个 change 的 proposal、delta spec、design、tasks、空 verification；
5. 做 artifact 自洽检查；
6. 停止，不改产品代码。

输出必须指出需要负责人确认的内容。负责人回复批准前，状态保持 Proposed。

### 6.3 Review/Approve

负责人至少核对：

- from 是否与代码事实一致；
- to 是否正是期望行为；
- breaking/迁移是否接受；
- out of scope 是否足以阻止扩张；
- scenarios 能否由测试或明确人工步骤验证；
- tasks 是否没有隐含另一个产品决策。

批准要指明 change ID 和 decision 值，例如：“批准 `version-binary-formats-v2`，按 `CK-DEC-001` apply”。

### 6.4 Apply

实现模型只读取一个 Approved change：

1. 先运行基线命令；
2. 优先添加能暴露缺失行为的测试；
3. 逐项实施 tasks；
4. 每个逻辑单元后运行专项验证；
5. 新发现若改变外部行为，先更新 artifacts 并重新获得批准；
6. 不 archive，不提交、不推送，除非另获授权。

Apply 可以修正设计里的错误，但不能静默改变 scope 或 decision。

### 6.5 Verify

最好由与实现不同的模型或至少新上下文完成。Verifier：

1. 不先相信已勾选 tasks；
2. 从 delta spec 的每个 scenario 反推测试；
3. 审查 diff 是否越界；
4. 运行专项与仓库完整门禁；
5. 检查文档、changelog、fixture 和失败副作用；
6. 填写 `verification.md`；
7. 失败则退回 Applying 或 Blocked。

Verify 阶段默认只允许修正文档证据和显而易见的小遗漏；任何实质代码修复回到 Apply，不让 verifier 悄悄变成第二个无规格实现者。

### 6.6 Archive

只有 `verification.md` 为 yes 且评审者确认后：

1. 把 change delta 合并进 `openspec/specs/<capability>/spec.md`；
2. 再次确认主规格描述的是已实现行为；
3. 把 change 目录移动到 `openspec/changes/archive/YYYY-MM-DD-<change-id>/`；
4. 保留 proposal/design/tasks/verification 和 delta；
5. 运行 spec 校验与最小项目 smoke；
6. archive 与实现应在同一 PR/提交序列中完成，不能提前归档未来行为。

若使用官方 OpenSpec CLI，可以使用其 propose/apply/verify/archive 能力；执行前先以当前安装版本的 help 为准，不在文档中锁死易变化的 slash command 拼写。

## 7. Artifact 质量门

### Proposal gate

- base commit 存在且与调查一致；
- 没有 `TBD` 影响行为或安全；
- breaking decision 已批准；
- scope/out-of-scope 明确；
- 一个 change 只有一个内聚结果。

### Spec gate

- 每个 requirement 至少一个 scenario；
- SHALL 行为可以观察；
- 错误和失败副作用已覆盖；
- 不包含实现细节；
- 外部契约名称、magic、版本和 schema 是精确值。

### Design gate

- 方案能对应每个 requirement；
- 明确文件范围和数据流；
- 没有顺手重构；
- 兼容、资源、原子性和回滚与风险相称。

### Task gate

- 先测试/fixture，再实现，再完整验证；
- 每项有完成证据；
- 没有“修复其他发现问题”这类无边界任务；
- 没有提交、推送、发布等未经授权动作。

### Archive gate

- 所有必需任务完成；
- 所有 scenario 有通过证据；
- 仓库标准门禁通过；
- `git diff --check` 通过；
- 主规格已同步；
- reviewer 确认；
- 没有未解释的工作区文件。

## 8. 并行与冲突规则

- 不同仓库的 Propose 可以并行；
- 不同仓库的 Apply 可以并行，但 FQC family change 要等待双方 fixture/spec 基线；
- 同一仓库一次只 apply 一个会触碰相同模块的 change；
- 文档 change 与代码 change 若修改同一 README，也按冲突处理；
- CI/release change 排在本地命令稳定之后；
- 一个 change 被阻塞时，不把无关任务塞进它来“保持进度”。

## 9. 何时不需要完整流程

纯拼写、失效链接且不影响能力声明、机械格式化可以走直接小修，但仍需仓库标准验证。以下情况必须走完整 change：

- binary format、CLI/schema、兼容性；
- 数据丢失、输出事务、资源限制；
- 科研参数、环境、数据库 provenance；
- CI 中核心测试是否真实执行；
- release、CPU baseline、供应链；
- 跨仓库产品边界。

本轮路线图中的 P0/P1 任务默认都使用完整流程。
