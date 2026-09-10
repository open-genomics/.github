# minibwa-rust 参考实现一致性与 CI 设计

目标仓库：`/home/shane/github/open-genomics/minibwa-rust`  
审计提交：`dad6e99b064a`

## 1. 核心结论

该项目最重要的质量指标不是普通单元测试数量，而是两项对外承诺：

1. `.l2b/.mbw` 与指定 C 版 minibwa 字节级兼容；
2. 算法和 SAM/PAF 行为以指定 C 参考实现为基准。

当前 `tests/c_parity.rs` 的覆盖思路是正确的，但参考二进制固定为 `../minibwa/minibwa`。当该路径或 chrM 数据不存在时，每个测试直接 `return`，Cargo 将其记为通过。当前 GitHub Actions 只 checkout Rust 仓库，因此绿色 CI 并不能证明 parity。

同时存在两处工程漂移：

- README 将 M7（对比脚本、CI、文档）标为“未开始”，但仓库已经有 CI、release 和脚本；
- `scripts/compare.sh` 比较的是系统 `bwa`，使用的 Rust `align`/`.fm` 命令已经不属于当前 CLI，无法作为 minibwa parity 工具。

## 2. 设计原则

### 2.1 “与哪个 C 版本一致”必须是确定值

不能跟随 upstream HEAD。仓库必须登记：

- C 仓库 URL；
- 完整 40 位 commit SHA；
- 构建命令和所需 submodule；
- 参考 binary 相对路径；
- 参考测试数据路径及许可来源；
- 首次确认 parity 的 Rust commit。

升级参考 C commit 是一次显式兼容性事件：先在独立 PR 中展示差异，再决定 Rust 跟进或继续固定旧参考。

### 2.2 本地便利与 CI 强制模式分开

- 普通开发者没有 C repo 时，可以运行纯 Rust 测试；输出应清楚说明 parity suite 未执行；
- 名为 `C reference parity` 的 CI job 必须设置强制模式；缺 binary、缺 fixture 或 0 个 parity case 都是失败；
- release 必须依赖强制 parity job，不能依赖会静默返回的普通 `cargo test`。

### 2.3 同时保留实时对照和冻结 fixture

实时对照能发现 C/Rust 行为差异；冻结 fixture 能让无网络、无 C compiler 和非 Linux 平台持续检查格式。两者不能相互替代。

## 3. 任务 `MBW-REF-001`：固定 C 参考来源

状态：**已决，P0；具体 commit 需维护者从已验证环境确认**

目标：仓库能确定性获取并构建曾用于现有 parity 声明的 C 版本。

允许修改：

- 新增 `tests/reference/minibwa.toml`（或等价单一 manifest）；
- 新增一个小型 setup/verify 脚本；
- `AGENTS.md` 与测试说明；
- CI workflow。

建议 manifest：

```toml
schema_version = 1
repository = "https://github.com/lh3/minibwa.git"
commit = "<full-40-hex-sha>"
binary = "minibwa"
build = "make"
rust_baseline_commit = "<full-40-hex-sha>"
```

实施要求：

1. 不直接填 upstream 最新 commit。先找回开发现有 parity 功能时使用的 C checkout；若无法找回，则选定一个候选 commit，在所有现有 case 上重新跑完并把结果交给维护者确认；
2. setup 脚本从 manifest 读取 URL/SHA，checkout detached commit，并验证实际 `git rev-parse HEAD`；
3. 若 C 项目含 submodule，必须按该 commit 初始化固定 submodule；
4. 构建输出不写入 Rust source tree 的受跟踪位置；
5. CI 缓存只能以参考 commit 为 key，缓存命中后仍验证 binary 和 commit；
6. 日志打印 C commit、compiler 版本与 build command；
7. 不修改上游 C 源码以“帮助”通过测试。

验收条件：在全新 runner 上只凭 manifest 与脚本可构建 C binary；把 SHA 改成无效值会明确失败。

非目标：自动跟踪 upstream、把 C 源码 vendoring 进 Rust 仓库。

## 4. 任务 `MBW-CI-001`：强制 parity job

状态：**已决，P0**  
前置条件：`MBW-REF-001`

目标：主分支与 PR 的必需检查真实执行所有 parity 测试。

允许修改：`tests/c_parity.rs`、`.github/workflows/ci.yml`、reference setup 脚本、测试辅助模块。

### 4.1 测试接口

使用明确环境变量，不再硬编码相邻目录：

```text
MINIBWA_C_BIN=/absolute/path/to/reference/minibwa
MINIBWA_C_TESTDATA=/absolute/path/to/reference/test
MINIBWA_REQUIRE_PARITY=1
```

行为规则：

- `MINIBWA_REQUIRE_PARITY=1` 时，变量缺失、路径不存在、fixture 缺失或 binary 不可执行立即失败；
- 非强制本地模式允许缺 C binary，但应只打印一次 suite-level 说明，而不是让每个 case 分别伪装完成；
- CI job 名称必须明确为 `C reference parity`，并设置 `MINIBWA_REQUIRE_PARITY=1`；
- job 最后输出实际执行 case 数，断言大于 0 且等于预期清单数。

### 4.2 测试隔离

1. 用唯一 `TempDir` 替换固定的 `/tmp/minibwa-rust-parity-*`；
2. 每个 case 独立目录，退出自动清理，避免并行测试和旧文件污染；
3. 命令 helper 同时捕获 status/stdout/stderr，失败信息必须展示程序、参数和 stderr；
4. parity 固定 `-t1` 以形成确定性基线。多线程确定性另设 case，不改变基线含义；
5. 不能用 `|| true`、ignore 或 feature 默认为关闭来绕过必需 job。

### 4.3 比较策略

| 产物 | 比较规则 |
|---|---|
| `.l2b/.mbw` | 完整字节相等 |
| `fastmap` / chain-only PAF | stdout 完整字节相等，前提是格式本身不含不稳定元数据 |
| SAM body | 顺序和逐行字段相等 |
| SAM header | `@HD`、`@SQ` 精确比较；只规范化双方必然不同的 `@PG` 程序名/版本/命令行字段 |
| stderr | 不要求文本相等，但双方必须成功；失败时纳入诊断 |

当前测试删除所有 `@` header 会掩盖参考序列长度/顺序错误，应收紧为上述规则。

### 4.4 CI 结构

```text
rust-quality
  ├─ fmt
  ├─ clippy
  └─ pure Rust/unit + frozen fixture tests

c-reference-parity
  ├─ build pinned C reference
  ├─ build Rust debug/release as selected
  └─ MINIBWA_REQUIRE_PARITY=1 cargo test --test c_parity
```

两者均为默认分支保护必需检查。parity job 失败不应被普通 Rust job 的成功覆盖。

验收条件：

- CI 日志能看到固定 C SHA 和每类 parity case；
- 删除 C binary、删除 chrM fixture 或篡改 Rust index byte 都会让 parity job 失败；
- 无 C 的本地纯 Rust命令仍可使用，但不会声称 parity 已通过；
- 所有测试使用独立 temp 目录。

验证：

```bash
cargo fmt --all -- --check
cargo clippy --all-targets -- -D warnings
cargo test
MINIBWA_C_BIN=/path/to/minibwa \
MINIBWA_C_TESTDATA=/path/to/minibwa/test \
MINIBWA_REQUIRE_PARITY=1 \
cargo test --test c_parity -- --nocapture
```

## 5. 任务 `MBW-FIXTURE-001`：提交小型冻结兼容样本

状态：**已决，P1**  
前置条件：`MBW-REF-001`

目标：即使没有 C toolchain，也能持续证明 Rust reader/writer 没有偏离已批准参考输出。

允许修改：`tests/fixtures/reference/`、测试代码、fixture README。

最小矩阵：

| 输入/命令 | 冻结产物 |
|---|---|
| 现有 `data/toy.fa`，默认 index | C 生成的 `.l2b/.mbw` |
| toy，`-u 2 -s 42` | 第二组 `.l2b/.mbw` |
| toy reads + fastmap | 预期 stdout |
| 至少一组单端 map | 规范化规则后的 SAM/PAF |

fixture README 必须记录：C commit、compiler、命令、输入 SHA-256、每个输出 SHA-256、生成日期和许可来源。测试读取已经提交的 C 输出与 Rust 当前输出比较；不可在测试运行时用 Rust 自己重写“expected”。

对 chrM 等较大 upstream fixture 继续在实时 parity job 获取，不盲目复制进仓库。

验收条件：断网且没有相邻 C repo 时，`cargo test` 仍执行冻结兼容测试；任意翻转 fixture 对应 Rust 输出中的一个稳定字节会失败。

非目标：把所有 C 测试数据复制进 Git、用 snapshot 更新命令自动接受差异。

## 6. 任务 `MBW-REL-001`：让发布依赖 parity

状态：**已决，P0**  
前置条件：`MBW-CI-001`、`MBW-FIXTURE-001`

目标：tag 触发的 GitHub Release 不能在 C 对照未执行时发布。

实施要求：

1. release workflow 增加强制 parity job，复用同一个 setup 脚本和测试命令；
2. `publish` 明确 `needs` parity、tag/version verify 和所有 build；
3. build matrix 中能原生运行的目标执行冻结 fixture 测试或最小 CLI smoke，而不只编译；
4. artifact job 不使用 `if-no-files-found: ignore` 隐藏本应存在的包；应为每个平台上传明确单一文件，缺失即失败；
5. SHA256SUMS 生成后验证资产数与 matrix 一致；
6. release notes 记录所对照的 C commit。

验收条件：模拟 parity 失败时 publish job 为 skipped/failed 且不会创建 release；缺任一平台 artifact 时同样不发布。

非目标：自动发布 crates.io，除非另有产品决策。

## 7. 任务 `MBW-DOC-001`：修复里程碑和失效工具

状态：**已决，P1**

允许修改：`README.md`、`AGENTS.md`、`scripts/compare.sh` 或其替代脚本。

实施要求：

1. 将 M7 拆成可核验子项，不再笼统写“未开始”或“完成”；
2. README 对每个“一致”声明注明参考 C commit 和数据范围；不能从 chrM 推导对所有基因组完全一致；
3. 修正 `AGENTS.md` 中“需先 make ../minibwa”为可选本地布局，同时给出环境变量/脚本标准路径；
4. 删除或重写失效的 `compare.sh`：
   - 比较目标应为固定 C minibwa，而不是不同产品 `bwa`；
   - 使用当前 `index/map/mem` 与 `.l2b/.mbw` CLI；
   - 在临时目录运行，不在当前目录留下固定文件；
   - 依赖缺失应非零退出，不能 warning 后继续形成“Done”；
   - 复用与测试相同的 SAM normalization 逻辑，避免脚本与 CI 产生两套判定。

验收条件：文档中的每条命令都由 smoke test 或人工记录验证；`rg -n '\balign\b|\.fm\b|M7.*未开始' README.md AGENTS.md scripts` 不再命中失效说明。

## 8. 推荐实施顺序

1. 找回并批准参考 C commit；
2. `MBW-REF-001`；
3. `MBW-CI-001`；
4. `MBW-FIXTURE-001`；
5. `MBW-REL-001`；
6. `MBW-DOC-001`。

任何发现的 parity 差异都应先保存最小输入和双边输出，再修 Rust。不要为了让 CI 变绿而扩大 normalization、删除比较字段或修改 C reference。
