# open-genomics 组织级工程基线

## 1. 设计取向

open-genomics 当前是 6 个独立仓库的集合，不是 monorepo。组织治理应当提供共同的“交付底线”，而不是建立一套所有语言都必须调用的中央脚本。

推荐模型：

```text
多仓库工作区根目录（不纳入 Git）
  └─ 临时审计、派工索引和跨仓阅读入口

六个独立项目仓库
  ├─ 自己的 AGENTS.md
  ├─ 自己的 openspec/specs 与 openspec/changes
  ├─ 自己的构建与测试命令
  ├─ 自己的格式规范/fixture
  ├─ 自己的 CI 与 release
  └─ 自己的 changelog

组织 .github 仓库（可选）
  └─ profile、默认 issue/PR 模板和安全入口；不存项目实现规格
```

不推荐抽取“共享 FASTQ 核心库”：fastq-tools、两个 compressor 和 MICOS 对 FASTQ 的需求、错误模型、语言与发布节奏不同。没有跨仓库版本治理和真实复用收益前，复制一个稳定的数据契约比制造共享库耦合更健康。

## 2. 推荐生命周期分类

状态：**生命周期分类 Accepted（2026-08-13）；具体 owner、支持平台和 release channel 仍需逐仓填写**

| 仓库 | 推荐阶段 | 对用户的含义 | 主要契约 |
|---|---|---|---|
| `compress-kit` | educational / experimental | 教学验证；不作为生产压缩格式，v2 内保持读取兼容 | 四种 binary magic/布局 |
| `fastq-tools` | stable tool | 稳定 CLI/公共 CMake surface；SemVer；发布产物需兼容声明 | FASTQ CLI、输出 TSV、C++ API |
| `fq-compressor` | incubating / prerelease | v1.0 前允许谨慎收敛；已发布 v2 archive fixture 不得无版本破坏 | `fqc` CLI、顺序 `.fqc` v2 |
| `fq-compressor-rust` | experimental | 保留 `fqc/.fqc`；可加固内部与格式 major，但未完成安全/CI 前不宣称稳定 archive | `fqc-indexed/v2`、范围访问 |
| `micos-2024` | research beta | 科研结果必须绑定环境/数据库/运行清单；非生产编排路径明确标注 | run manifest、结果目录、科学参数 |
| `minibwa-rust` | experimental compatibility rewrite | 兼容性只针对固定 C commit 和明确测试范围 | `.l2b/.mbw`、SAM/PAF 行为 |

生命周期不是质量排名。它决定的是“允许什么变化、需要什么证据”。例如 educational 项目可以不支持旧格式，但必须用新 magic 明确拒绝；research 项目可以没有通用安装包，但不能没有环境和运行 provenance。

## 3. 决策 `ORG-GOV-001`：仓库本地规格治理

状态：**Accepted（2026-08-13，组织负责人确认）**

工作区根目录不是 Git 仓库；这是多仓库布局事实，不是需要修复的问题。负责人选择不创建中央工程仓库，而是在每个子仓库维护自己的规格和 change 历史。

每个子仓库采用以下最小结构：

```text
<repo>/
  openspec/
    project.md
    specs/
      <capability>/spec.md
    changes/
      <change-id>/
        proposal.md
        design.md
        tasks.md
        specs/<capability>/spec.md
      archive/
```

实施要求：

1. 只把与目标仓库有关的专题设计提炼进该仓，不复制整套组织审计；
2. `openspec/specs/` 只描述当前已实现并验证的真相；未来行为写在活动 change 的 delta spec；
3. change 必须包含 proposal、完整未来态 spec、必要 design 和可验证 tasks；
4. apply 前由负责人批准 proposal/spec；verify 通过后才能 archive；
5. 跨仓库决定使用同一 decision ID，但每个仓库只保存自己需要遵守的部分并链接对方 canonical spec；
6. 根目录 `maintenance-design/` 继续作为未版本化的派工快照，不能被项目 CI 或构建依赖；
7. 不在根目录初始化 Git，不制造跨仓提交，不创建中央 build orchestrator；
8. 组织特殊 `.github` 仓库若以后存在，只承载 profile、默认 issue/PR/安全模板。

验收条件：clone 任一子仓库即可看到该项目的当前规格、活动 change、历史归档和验证方法，不需要依赖根目录文件；各仓库仍可独立构建、测试和发布。

## 4. 任务 `ORG-LIFE-001`：公开项目状态与维护责任

状态：**分类已决；待负责人提供 owner/支持范围，P0**

目标：用户不再从版本号、README 修辞或 CI badge 猜测项目成熟度。

每个仓库的 `openspec/project.md` 与 README 记录：

| 字段 | 说明 |
|---|---|
| Repository | 当前 canonical URL |
| Lifecycle | experimental / incubating / stable / maintenance / archived |
| Category | education / tool / research workflow / compatibility rewrite |
| Maintainer | 至少一个可联系 GitHub team/handle |
| Supported platforms | 只写 CI/发布实际验证的平台 |
| Release channel | none / prerelease / stable |
| Primary contracts | 链接到仓库内 `openspec/specs/` |
| Security contact | 组织默认 policy 或仓库入口 |
| Last reviewed | 日期 |

每个仓库 README 首屏放一行 lifecycle 与支持范围，并链接仓库内 `openspec/project.md`；不要堆一排无法证明的 badge。

状态转移规则：

- `experimental -> incubating`：核心功能可用、有最小 CI、依赖锁、外部契约文档；
- `incubating -> stable`：至少一个受控 release、兼容 fixture、失败/安全路径门禁、维护者和支持窗口；
- `* -> maintenance`：只接受 bug/security/compat 修复，README 明示；
- `* -> archived`：只读、替代方案和最后支持版本明确。

验收条件：6 个仓库都有 owner、阶段、平台和 release channel，且信息与仓库事实一致。

## 5. 任务 `ORG-CONTRACT-001`：建立仓库内外部契约规格

状态：**已决，P0；FQC 和 CompressKit 名称决策已经批准**

目标：任何实现模型在修改 bytes、CLI 或科研产物前，都能查到契约所有者和变更规则。

首批登记：

| Contract ID | Owner | 内容 | 变更门禁 |
|---|---|---|---|
| `fqc-sequential/v2` | `fq-compressor` | C++ 顺序 `.fqc` | 格式 ADR + reader fixture + major/magic 策略 |
| `fqc-indexed/v2` | `fq-compressor-rust` | Rust block-index `.fqc` archive | 规范/fixture + major/magic 策略 |
| `compress-kit/{huffman,arithmetic,range,rle}/v2` | `compress-kit` | 教学压缩格式 | 新 magic + fixture + release major |
| `minibwa/{l2b,mbw}` | `minibwa-rust` | 与固定 C commit 的字节兼容 | pinned reference + forced parity |
| `minibwa/sam-paf` | `minibwa-rust` | 明确参数集下的输出行为 | normalization 规则 + parity cases |
| `micos/run-manifest/v1` | `micos-2024` | resolved config、工具/数据库、stage 状态 | JSON schema version + migration policy |
| `fastq-tools/stat-tsv/v4` | `fastq-tools` | stat 输出字段/语义 | fixture + SemVer/changelog |

这些不是一个中央表中的复制条目，而是各 owner 仓库 `openspec/specs/<capability>/spec.md` 的最小必备字段：owner repo、识别方式（magic/command/schema）、当前版本、兼容矩阵、golden fixture、breaking-change procedure、最后验证 commit。根目录索引只链接，不成为规范来源。

任何跨仓索引都不复制完整格式。详细规范由 owner 仓库维护；其他仓库只保存识别所需的 magic、family ID 和 canonical link。

验收条件：两个 `fqc/.fqc` 分别登记为 `fqc-sequential/v2` 与 `fqc-indexed/v2`；每个 binary/schema contract 都有至少一个可运行 fixture 或明确的 change ID。

## 6. 任务 `ORG-META-001`：修复活动仓库身份

状态：**已决，P1**

已确认需要修改的活动链接集中在：

- `fastq-tools`：README 与 CMake homepage；
- `fq-compressor`：README、CMake、Conan metadata；
- `micos-2024`：README、CONTRIBUTING、CITATION、pyproject、AGENTS 和 docs；
- `compress-kit` 当前活动项目 URL 已迁移；其中 AICL-Lab copyright 是权利归属问题，不因 GitHub 组织名不同自动改写；
- 两个 Rust 仓库当前 canonical repository metadata 已是 `open-genomics`。

实施要求：

1. canonical clone/homepage/issues/docs/CI/release 链接统一为 `open-genomics`；
2. 文档站 base 与部署 URL 逐仓验证，不只字符串替换；
3. CITATION 的 repository URL 更新，但作者、年份、DOI 等学术元数据不能未经确认改写；
4. 个人作者署名和组织项目归属分开，不能把迁移误当版权转让；
5. 自动检查只扫描活动元数据与文档。历史 changelog/postmortem 中旧链接可以保留历史语境；
6. badge 只有在对应 workflow/release/page 确实存在时才保留。

验收条件：每仓 README 的 clone 命令可定位 canonical remote；所有 active badge 指向实际 workflow；旧组织只出现在明确的历史/作者语境。

## 7. 分级 CI 基线

### 7.1 所有 active 仓库的共同底线

- clean checkout 可按 README/AGENTS 构建；
- tests、format/lint 的失败使 CI 非零；
- lockfile 被提交且被构建命令真正消费；
- CI actions 固定到完整 SHA，尤其是带写权限或 secrets 的 job；
- 缺依赖的核心测试不能静默 skip；
- cache key 包含 lock、toolchain 和 ABI 相关配置；
- workflow 默认只读权限，发布 job 单独提升；
- `git diff --check` 或等价格式检查阻止明显损坏。

### 7.2 类别附加门禁

| 类别 | 必需附加门禁 |
|---|---|
| 外部二进制 parser | 冻结合法/损坏 fixture，ASan+UBSan，短 fuzz smoke或等价 corpus regression |
| 稳定 C++ tool | GCC/Clang、Release/Debug、安装 consumer、发布 artifact smoke、CPU baseline |
| Rust tool | fmt、clippy `-D warnings`、tests、doc、Cargo.lock、依赖 audit |
| 科研 workflow | 严格配置、环境 lock、WDL/schema validation、run manifest、固定小数据集集成 |
| 参考实现重写 | pinned upstream、强制 parity job、冻结 reference outputs、release depends on parity |
| 教学项目 | 至少 current round-trip、format version fixture、损坏输入和文档真实性 |

不要为追求表面一致给 compress-kit 加 TSan，也不要用 micos 的大容器构建拖慢所有 README PR。门禁由风险决定。

## 8. 依赖与供应链基线

1. 应用/CLI 仓库提交 lock：Rust `Cargo.lock`、C++ Conan lock、MICOS 科研环境 lock；
2. 直接依赖版本政策由项目决定，但 release/科研运行必须可解析到 immutable artifacts；
3. 容器使用 digest；源码/工具下载校验 checksum；禁止关闭 TLS 验证；
4. 依赖升级单独 PR，不和算法/格式/科学参数变化混在一起；
5. scheduled audit 失败产生可见 issue/告警，但不能自动修改并发布；
6. release 保存依赖/工具版本清单和许可证清单；
7. 不把 CI action major tag 当作 immutable pin。

## 9. 发布最低门槛

任何 stable/prerelease binary 或科研 release 都必须验证：

```text
version source == tag == changelog entry
required CI gates == success
contract fixtures == success
artifact smoke == success
supported platform/CPU == documented and tested
checksums == present
known limitations == explicit
maintainer approval == recorded
```

发布 workflow 首选创建 draft，人工核对后公开。实现模型不得自行 push tag、覆盖 asset 或公开 release。

科研 release 额外保存：environment lock、container digests、database manifests、run manifest、测试数据 provenance 和关键输出检查。

## 10. 任务 `ORG-SEC-001`：统一安全报告入口

状态：**已决，P2；需负责人提供联系方式**

六个仓库都处理外部数据或依赖供应链，但当前没有统一可发现的安全报告策略。建议在组织 `.github` 仓库放默认 `SECURITY.md`，项目只在支持版本不同或风险特殊时覆盖。

内容至少包括：支持版本、私下报告方式、响应预期、公开披露原则、不得上传敏感基因组/患者数据、archive parser DoS/corruption 的报告范围。没有可维护的邮箱/team 前不要填写占位地址，也不要承诺不现实的响应 SLA。

## 11. 不应中央化的内容

- 构建脚本：CMake、Cargo、Python/WDL 的细节留在各仓；
- 领域类型：不要建立跨语言 `ReadRecord` 共享模型；
- 错误码：除非形成进程级集成协议，否则不统一；
- 默认分支名：不为统一外观制造迁移成本；
- release 版本号：各仓独立；
- 测试数量/覆盖率单一阈值：风险不同；
- README 大模板：只统一必需信息，不统一视觉风格。

## 12. 组织级实施顺序

1. 在六个子仓库建立最小 OpenSpec 结构，并提炼各自当前事实；
2. 把四项已批准决定写入相关仓库规格，登记 6 个项目生命周期/owner；
3. 完成 FQC 格式族识别、Rust 规范、CompressKit magic、minibwa parity 四个 P0 契约任务；
4. 完成 MICOS 配置/环境可复现性；
5. 完成 fastq-tools CPU baseline 和 C++ FQC fixture/sanitizer；
6. 统一 active metadata；
7. 再补 release、安全入口和定时重型门禁。

组织治理的成功指标不是“所有仓库看起来一样”，而是任何用户和实现模型都能准确知道：项目是否稳定、契约由谁负责、改变它需要什么证据、失败时哪条门禁会阻止发布。
