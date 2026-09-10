# MICOS-2024 可复现性与编排治理设计

目标仓库：`/home/shane/github/open-genomics/micos-2024`  
审计提交：`c2c399f89c9c`

## 1. 当前事实

### 1.1 实际生产入口是 Python CLI

`micos full-run` 调用 `micos/full_run.py`，按顺序直接执行 QC、taxonomy、diversity、functional 和 summary。`scripts/run_full_analysis.sh` 已经收缩为该 CLI 的薄包装，并明确拒绝旧的 `--skip/--resume-from`。

仓库同时存在 6 个分散的 WDL workflow，但没有一个把全链路连接起来的顶层 WDL，也没有 WDL 语法/执行 CI。README 却把项目描述为“基于 WDL、支持断点续传与错误恢复”。这是对当前能力的过度承诺。

### 1.2 配置表面大于真实生效范围

`analysis.yaml.template` 已注明只有输入/输出目录、两个数据库路径和最大线程数实际生效。其余 QC 阈值、Kraken2 confidence、QIIME2、HUMAnN 等字段被 `extra="allow"` 接受后忽略。CLI 加载配置失败时还会记录 warning 并退回默认值。

对科研流程而言，“配置看似被接受但未生效”比明确失败更危险，因为运行记录无法代表真实参数。

### 1.3 环境没有单一可解析版本真相

已确认的漂移包括：

| 组件 | 基准/多数引用 | 漂移位置 |
|---|---|---|
| Ubuntu | 24.04 / `noble` | Kraken Dockerfile `FROM ubuntu:26.04`，label 与 apt 源仍为 24.04 |
| Python | 3.9 | kraken-biom Dockerfile 使用 3.14 |
| QIIME2 | 2024.5 | QIIME2 Dockerfile 使用 2024.10 |
| R | 4.3.0 | phyloseq Dockerfile 使用 4.6.1 |

`environment.yml` 还有未固定版本和 `>=` 依赖；容器 base image 未使用 digest；Krona 构建关闭 TLS 验证并在构建时获取最新 taxonomy。这些都使“同一 Git 提交”不足以重建相同环境。

### 1.4 示例携带个人机器绝对路径

多个 WDL JSON 和 Compose 文件包含 `/media/shuai/...`、`/home/shuai/...`。这些文件既不能直接作为可移植示例，也无法在 CI 中验证。

## 2. 决策 `MICOS-ORCH-001`

状态：**Accepted（2026-08-13，组织负责人确认）**

已批准决策：

1. `micos` Python CLI 是唯一生产级编排入口和参数语义来源；
2. `scripts/run_full_analysis.sh` 只保留为薄包装；
3. `steps/**/*.wdl` 标为实验性、单步骤参考实现；
4. 在存在顶层 workflow、固定容器 digest、输入 schema、WDL 验证和最小集成测试之前，WDL 不承诺与 Python 流程等价；
5. 不删除 WDL。未来若 WDL 成熟，应通过新的 ADR 决定它是否替换 Python 编排，而不是长期维护两个“同等权威”的流程。

批准记录：

```text
MICOS-ORCH-001: Accepted
Production orchestrator: Python CLI
WDL current status: experimental
Approved by: organization owner
Date: 2026-08-13
```

## 3. 目标架构

```text
analysis.yaml + databases.yaml + samples.tsv
                    │
           strict configuration load
                    │
              resolved run plan
                    │
         Python stage DAG / state machine
          │       │        │       │
         QC    taxonomy  diversity functional
          └────────────┬────────────┘
                     summary
                        │
      run-manifest.json + stage outputs + logs
```

容器、Conda 和 WDL 都消费同一套已批准工具版本，但不把某个 Dockerfile 当作第二份版本真相。

## 4. 任务 `MICOS-DOC-001`：修正文档能力边界

状态：**已决，P0；不依赖编排决策即可先陈述现状**

允许修改：`README.md`、`docs/`、`AGENTS.md` 中描述流程状态的部分、必要的 WDL 目录说明。

实施要求：

1. 删除当前已经不成立的“支持断点续传与错误恢复”；
2. 把快速开始的主路径明确为 `micos full-run`，shell 脚本仅是兼容包装；
3. 在 WDL 章节写明：只有独立步骤、没有顶层端到端 workflow、尚未在 CI 验证；
4. 区分核心 CLI 已集成步骤、实验 WDL、`scripts/` 中独立分析脚本和 `steps/09` 未集成示例；
5. 配置章节只把真实读取字段描述为可用参数；
6. 不使用“完整可重现”描述尚无 lock、digest 和运行清单证明的路径。

验收条件：

- `rg -n "断点续传|错误恢复|完整.*可重现|基于 WDL" README.md docs AGENTS.md` 的结果均与现状一致；
- README 的一条主命令可以追踪到 `micos.cli:main -> run_full_pipeline`；
- 文档没有宣布后续任务尚未实现的能力。

验证：

```bash
pytest tests/test_docs_whitepaper.py -v
git diff --check
```

非目标：实现 resume、删除 WDL、重写分析模块。

## 5. 任务 `MICOS-CONFIG-001`：严格且可追踪的有效配置

状态：**已决，P0**

目标：用户配置要么被明确使用，要么在运行前失败；禁止静默忽略和配置错误后回退。

允许修改：

- `micos/config.py`、`micos/cli.py`；
- `config/*.template`、配置文档；
- 配置与 CLI 测试。

实施要求：

1. 生产配置 model 对未知字段采用 `extra="forbid"`；
2. `analysis.yaml.template` 只保留已接入 CLI 的字段。愿景参数迁到 roadmap 文档，不能继续作为可执行模板；
3. 修正 `max_memory` 与 `memory_gb` 这类名称漂移：未实现资源限制前，两者都不应出现在活动配置；
4. YAML 语法或 Pydantic 验证失败时，`micos --config ... full-run` 以配置错误退出，不得 warning 后使用默认值；
5. 明确优先级：显式 CLI 参数 > `analysis.yaml` > `databases.yaml` 的对应数据库值 > 代码默认值。重复配置时可输出 resolved value 来源；
6. 相对路径统一相对于包含它的配置文件目录解析，而不是进程当前工作目录；
7. `validate-config` 对语法、未知字段和必需的所选 stage 依赖返回非零；占位符、暂不需要的可选数据库可作为 warning；
8. 增加 `micos ... --dry-run` 的 resolved plan 输出，至少列出 stage、输入、输出、threads、数据库和参数来源；不要执行外部工具；
9. 不用宽泛 `except Exception` 吞掉配置失败。只在 CLI 边界把已知配置异常转换成稳定退出码。

必须测试：未知字段、错误类型、空 YAML、相对路径、CLI 覆盖、两个数据库配置来源冲突、损坏 YAML、dry-run 不执行命令。

验收条件：任何被模板展示的字段都能在 dry-run 的 resolved plan 中观察到；拼错字段会在外部工具启动前失败。

验证：

```bash
pytest tests/test_config.py tests/test_cli.py -v
black --check --diff micos tests
isort --check-only --diff micos tests
flake8 micos tests
mypy micos --ignore-missing-imports
pytest tests/ -v
```

非目标：一次性把所有愿景参数接入生信工具。

## 6. 任务 `MICOS-ENV-001`：工具版本与容器供应链闭环

状态：**已决，P0**

目标：从一个已知成功环境生成可复建 lock，并自动阻止环境、Docker、Singularity、Compose 和 WDL 的版本漂移。

### 6.1 版本来源规则

1. `environment.yml` 继续作为直接依赖和生信工具版本的人工基准，遵循仓库 `AGENTS.md`；
2. 为支持的平台生成并提交解析后的 lock 文件，包含 transitive dependency 与 artifact 哈希；
3. 不凭空给未固定包挑“最新版”。先从维护者已验证的现有环境导出版本，再验证重新求解；
4. `pyproject.toml` 的库安装兼容范围和科研运行环境 lock 是两个概念：开发库可保留合理范围，论文/生产运行必须使用 lock；
5. 支持平台第一阶段只承诺 `linux-64`。没有实际 CI 证明前，不因 Python package metadata 就宣称 Windows/macOS 完整流程可运行。

### 6.2 已知漂移的裁决

以当前 `environment.yml` 的明确值为基准修复，而不是升级：

- Kraken image 回到 Ubuntu 24.04 语义，并使 `FROM`、label、apt suite 一致；
- kraken-biom image 使用 Python 3.9 基线；
- QIIME2 image 使用 2024.5；
- phyloseq/R image 使用 4.3.0；
- 其他无明确值的包从已验证环境冻结，另开升级 PR。

### 6.3 容器要求

1. 发布/论文使用的 base image 与 WDL runtime 记录 immutable digest；tag 只用于可读 label；
2. 下载源码必须固定版本并验证 SHA-256，不能只依赖可变 URL/tag；
3. 禁止 `conda config --set ssl_verify no`；
4. Krona taxonomy 从 image 构建中分离，作为有日期、来源 URL、checksum 的数据库 artifact 管理；
5. 不在同一 Dockerfile 中无版本地升级 pip 和安装浮动依赖；
6. 增加一个轻量校验脚本，解析 `environment.yml` 的关键版本并对 Dockerfile、def、Compose、WDL 引用做一致性检查。例外必须显式登记原因，而非散落注释。

验收条件：

- `rg -n ">=|latest|ssl_verify no" environment.yml steps containers` 的剩余结果均被审查且不影响锁定执行路径；
- 四个已知漂移消失；
- 新环境可从 lock 在干净 Linux 环境创建；
- `python --version` 与关键工具 `--version` 结果写入验证记录；
- 版本一致性脚本故意修改一个 Docker tag 后会失败。

验证命令应由实现者按选定 lock 工具写入仓库文档，并至少包含：环境创建、关键版本打印、`pytest tests/ -v`、版本一致性测试。

非目标：借机升级 QIIME2、Python、R 或生信算法版本。

## 7. 任务 `MICOS-PATH-001`：清除个人路径，建立可运行示例

状态：**已决，P1**

目标：clone 后所有公开示例不依赖原作者目录结构，也不泄露个人路径。

允许修改：`steps/**/*.json`、`steps/**/docker-compose*.yaml`、示例文档、`.gitignore`，以及小型测试 fixture。

实施要求：

1. 删除所有 `/media/shuai/` 和 `/home/shuai/` 路径；
2. Compose 使用明确环境变量，例如 `${MICOS_DATA_ROOT:?set MICOS_DATA_ROOT}`，并提交 `.env.example`，不提交真实 `.env`；
3. WDL 输入 JSON 分成：
   - 可在仓库小 fixture 上运行的 smoke input；或
   - 后缀为 `.example` 且含明显占位符的用户模板；
4. 如 WDL 需要转换脚本，该脚本必须位于仓库并用相对路径引用，不能引用作者工作区外文件；
5. 对生成式的 `commands/docker-compose-*.yaml` 明确其来源。若它们只是一次性命令快照，优先删除或归档到示例目录，只保留一个受维护的 canonical Compose；这一步需先证明没有文档/脚本依赖；
6. CI 增加个人绝对路径扫描，但允许文档中抽象的 `/home/user/...` 示例。

验收条件：

```bash
! rg -n '/(home|media)/shuai/' . --glob '!.git/**'
docker compose --env-file .env.example config   # 对每个受支持 compose
```

WDL JSON 还需通过 JSON 解析及对应 WDL input 名校验。

非目标：把大型数据库和真实患者数据提交到 Git。

## 8. 任务 `MICOS-RUN-001`：运行清单与可靠 stage 状态

状态：**已决，P1**  
前置条件：`MICOS-CONFIG-001`；生产入口决策接受 Python 方案。

目标：每次运行都能回答“用什么输入、参数、工具和数据库，哪些步骤成功”。第一阶段建立可审计清单，不直接宣称完整 resume。

在 `results_dir/run-manifest.json` 记录：

- manifest schema version、run ID、开始/结束时间、最终状态；
- MICOS package version 与可用时的 Git commit；
- 原始 argv、resolved config 及其 SHA-256；
- 样本 ID、输入路径、大小、mtime，以及来自 samples sheet 的 checksum；没有 checksum 时标记 `unverified`；
- 数据库逻辑名、版本/构建日期、manifest checksum，避免散列整个超大数据库；
- 每个外部工具版本或容器 digest；
- 每个 stage 的 `pending/running/succeeded/failed/skipped`、时间、输入、输出和失败摘要；
- 明确的 schema version，未来读取不靠猜测字段。

实施要求：

1. manifest 使用临时文件 + rename 原子更新；
2. 启动外部命令前将 stage 写为 `running`，成功验证产物后才写 `succeeded`；
3. `skipped` 必须记录是用户选择还是依赖不可用；后者通常应是失败，不能伪装为成功；
4. 不把密码、token 或完整敏感环境变量写入 manifest；
5. 输出目录已含 manifest 时，默认拒绝覆盖。提供明确的新 run 或继续策略；
6. stage 函数返回结构化产物，不靠扫描目录猜结果；
7. summary 只消费已登记的成功产物，并标识缺失模块。

验收条件：成功、某 stage 失败、用户 skip、Ctrl-C 四种场景均留下可解析清单；失败不会把未完成 stage 标成成功。

### 后续任务 `MICOS-RESUME-001`

只有 `MICOS-RUN-001` 稳定后再实现 resume。缓存命中条件必须同时满足：stage signature（工具身份 + resolved 参数 + 输入 fingerprint）相同、输出存在且 fingerprint 匹配、上游未失效。强制重跑某 stage 必须失效所有下游。仅检查“目录存在”不能作为恢复依据。

## 9. 任务 `MICOS-CI-001`：分层验证

状态：**已决，P1**

目标：PR 快速反馈与重型科研集成测试分开，避免全都 mock，也避免每个 PR 构建全部镜像。

### PR 必跑层

- 现有 Python lint、type check、unit tests 和覆盖率；
- config 模板严格解析与 dry-run；
- shell syntax；
- 所有 WDL 的固定版本 validator 检查；
- JSON/Compose schema 和个人路径扫描；
- 环境/容器版本一致性脚本；
- docs build。

### 定时/手动容器层

- 对受支持 Dockerfile 构建或拉取固定 digest；
- 执行每个关键工具的版本检查；
- 对可用小 fixture 运行单 stage smoke；
- 失败创建可见告警，但不自动改版本。

### 发布/论文冻结层

- 在锁定环境上运行一份可公开的小型端到端数据集；
- 比较结构化结果、关键统计量和 provenance，不比较包含时间戳的整个目录哈希；
- 保存 run manifest、环境 lock、容器 digest 和数据库 manifest；
- 发布不得只依赖 mock command tests。

验收条件：故意破坏 WDL、配置字段、容器版本或真实 smoke 命令中的任一项，都能让相应层失败；重型 job 不阻塞普通文档 PR，除非触发相关路径。

## 10. 推荐实施顺序

1. `MICOS-DOC-001`
2. 在仓库内固化已接受的 `MICOS-ORCH-001`
3. `MICOS-CONFIG-001`
4. `MICOS-ENV-001`
5. `MICOS-PATH-001`
6. `MICOS-RUN-001`
7. `MICOS-CI-001`
8. 在真实需求出现时设计 `MICOS-RESUME-001`

不要在一个 PR 中同时升级生信工具、改变参数默认值和引入恢复机制；这三类变化会让科学结果差异无法归因。
