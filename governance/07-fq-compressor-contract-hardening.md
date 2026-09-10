# fq-compressor 契约加固设计

目标仓库：`/home/shane/github/open-genomics/fq-compressor`  
审计提交：`1361d4e8628a`  
当前软件版本：`0.3.0-rc1`

## 1. 总体判断

该仓库的核心架构已经合理：顺序有界帧、压缩/解压镜像流水线、逐帧预检、显式 `finish()`、同目录输出事务、结构化错误和 postmortem 机制都值得保留。本轮不应因 `archive.cpp` 较大就拆文件，也不应重新引入 v1、索引、随机访问或第二套执行引擎。

需要修复的是“外部承诺可证明性”：

- README 声称 `verify` “不解压即可完整校验”，实际代码完整 zstd 解压、逻辑流校验和记录解码，只是不写 FASTQ；
- 格式测试丰富，但没有提交一个由已知版本生成的冻结 `.fqc` archive 证明未来 reader 向后兼容；
- CI 只跑 clang-debug、测试和 format；外部 archive/FASTQ parser 没有持续 sanitizer/fuzz 门禁；
- CI 使用 action 浮动 tag、未固定 Conan 工具版本，也没有 transitive dependency lock；
- 活动元数据仍指向 `LessUp`；LICENSE 还排除已经从当前树删除的 `vendor/spring-core/`。

## 2. `verify` 的正确语义

当前 `ArchiveEngine::verify` 与 decompress 复用同一 `DecompressPipeline`：

1. 读取并验证 global header；
2. 读取 frame header 和压缩 payload；
3. zstd 解压三路 stream；
4. 验证逻辑 frame checksum；
5. 解析并验证 FASTQ records；
6. 按 frame 顺序累积 totals/global checksum；
7. 与 footer 比较；
8. 不写 FASTQ 输出。

这是比“只读 header/checksum”更强的验证，应如实表述为“完整解码校验但不落盘”。

现有 v2 无法实现能够发现任意 payload 损坏的“不解压 quick verify”：frame checksum 覆盖未压缩逻辑 stream，压缩 payload 本身没有单独的 archive-level checksum。只根据 frame header 中已存 checksum 累积 footer，会漏掉 payload 损坏。若未来需要真正 quick verify，应在新格式 major 中给压缩 payload 增加 checksum，不能给 v2 添加一个名不副实的选项。

## 3. 任务 `FQC-CPP-DOC-001`：修正 verify 和产品边界

状态：**已决，P0**

允许修改：`README.md`、`ARCHITECTURE.md`、CLI help/手册中涉及 verify 的文字、`CHANGELOG.md`。

实施要求：

1. 将“不解压即可完整校验”改为“完整解码并验证所有结构、逻辑内容与 footer，但不写解压输出”；
2. 明确 verify 的 CPU/内存成本与 decompress 同量级，不能作为常数时间 metadata check；
3. 说明 XXH64 用于发现随机损坏，不提供对恶意篡改的密码学认证；
4. 在 README 添加与 Rust 同名项目格式不兼容说明，遵循 `FQC-DOC-001`；
5. 不增加 `--quick`，不改变 verify 实现；
6. 保留“可校验”产品卖点，但把机制写准确。

验收条件：

```bash
! rg -n '不解压.*完整校验|无需解压.*完整' README.md ARCHITECTURE.md docs src include
rg -n '完整解码|不写.*输出|XXH64' README.md ARCHITECTURE.md
git diff --check
```

非目标：改变 archive bytes、性能优化、增加新命令。

## 4. 任务 `FQC-CPP-FMT-001`：规范与冻结 v2 decoder 契约

状态：**已决，P0**

目标：从 `0.3.0-rc1` 开始，未来版本能通过 committed fixture 证明仍能读取已发布 v2，而不是只证明当前 writer 与 reader 自洽。

允许修改：

- 新增 `docs/reference/fqc-v2.md`；
- `tests/data/fqc-v2/` 与 manifest/README；
- `tests/format`、CLI e2e 和 CMake 测试注册；
- 与规范验证直接相关的小型 helper。

### 4.1 规范内容

规范必须逐 byte 记录：

- 8-byte archive magic、u16 version、32-byte global header 各 offset；
- flags、profile 与四个 codec 字段的合法值；
- header XXH64 覆盖范围和 seed；
- 72-byte frame header 的 marker、frame ID、record count、raw/compressed sizes、checksum；
- 三个 payload 的顺序和 zstd framing；
- ID/sequence/quality 原始逻辑 stream 的 varint、2-bit、异常碱基编码；
- logical checksum 精确拼接顺序；
- 40-byte footer、rolling checksum 算法、空 archive 值；
- little-endian、大小边界、paired/frame 不变量、trailing-byte 规则；
- version/codec 不支持时的行为。

`ARCHITECTURE.md` 保持高层说明并链接规范，不复制一份会漂移的 offset 表。

### 4.2 fixture 矩阵

至少提交：

| Fixture | 覆盖 |
|---|---|
| `empty.fqc` | zero frame、footer |
| `single-illumina.fqc` | ID/comment、普通 ACGT、quality |
| `iupac-case.fqc` | 全部支持 IUPAC 大小写和质量边界 |
| `paired-multiframe.fqc` | paired、多个 frame、rolling checksum |

每个 fixture 旁边的单一 manifest 记录生成器 Git SHA、软件版本、zstd/xxhash 版本、命令、输入 SHA-256、archive SHA-256、预期 metadata/records。

### 4.3 测试策略

1. decoder 必须读取冻结 fixture 并逐记录/metadata 比较；
2. CLI `verify` 与 `decompress` 都消费 fixture；
3. current writer 结构测试继续断言规范字段；
4. 不把“未来 writer 必须产生整文件同一 SHA”作为默认契约，因为 zstd 升级可能改变合法表示；只有在明确冻结依赖版本和 canonical encoding 后才作此承诺；
5. fixture 不能在测试运行时由当前 writer重新生成；更新必须通过显式维护命令并经格式评审；
6. v1/错误 magic 测试应修复当前无意义的重复条件断言，精确检查 error code/message 类别。

验收条件：

- 替换 current writer 并不影响 fixture 读取测试；
- 改坏任何一个稳定 offset、endian、rolling checksum 或 codec ID 都有直接测试失败；
- `0.3.0-rc1` 或批准的首个 fixture producer commit 在 manifest 中可追溯；
- `make`/标准测试命令自动运行 fixture suite。

验证：

```bash
./scripts/lint.sh format-check
./scripts/build.sh clang-debug
./scripts/test.sh clang-debug
```

非目标：支持 v1、与 Rust archive 互读、要求所有未来 writer 字节完全一致。

## 5. 任务 `FQC-CPP-CI-001`：可信的依赖与 sanitizer 门禁

状态：**已决，P1**  
前置条件：本地 ASan/UBSan 路径按已有 postmortem 的依赖构建要求稳定运行。

目标：PR 至少在普通 debug 和 ASan+UBSan 下执行完整 parser/format 测试，且 CI 自身依赖可重放。

允许修改：`.github/workflows/ci.yml`、Conan lock/profile、build scripts/presets、CI 说明。

实施要求：

1. 固定 GitHub Actions 到完整 commit SHA，注释对应 release tag；
2. 固定 Conan 版本，生成并提交 `conan.lock`，所有本地/CI install 真正消费它；
3. cache key 包含 `conanfile.py`、lock、profile、compiler、build type/sanitizer，避免混用 ABI 不同包；
4. 保留现有 clang-debug job；
5. 新增 ASan+UBSan job并运行全部测试：
   - `halt_on_error=1`；
   - 不用 broad suppression 掩盖项目错误；
   - 按已有 sanitizer postmortem，以相同 instrumentation/stdlib 重新构建需要的测试依赖，解决 GTest/libc++ 混编问题；
   - 受 runner 限制无法启用 leak detection 时可以明确关闭 LSan，但必须保留 ASan/UBSan，并在 workflow 注释原因；
6. TSan 可先作为默认分支 push 或每周 job，不要求每个 PR；发现 data race 时必须非零；
7. CI 安装工具使用固定 runner image/明确版本，禁止 `pip install conan` 浮动；
8. timeout 和取消策略保留，错误日志/崩溃样本上传仅在失败时发生。

验收条件：故意加入一个测试内 heap OOB/UB（只在临时验证分支）会使 sanitizer job 失败；普通 job 成功不能覆盖它。lock 与 conanfile 不同步时在 install 阶段明确失败。

验证：

```bash
./scripts/test.sh clang-debug
ASAN_OPTIONS=detect_leaks=0:halt_on_error=1 \
UBSAN_OPTIONS=halt_on_error=1 \
./scripts/test.sh clang-asan
./scripts/lint.sh format-check
```

实际 LSan 值按 CI 环境验证后记录，不复制本地受管环境限制到所有 runner。

非目标：大规模平台矩阵、自动 release、覆盖率阈值一次性引入。

## 6. 任务 `FQC-CPP-FUZZ-001`：archive/FASTQ 确定性 corpus 与 fuzz smoke

状态：**已决，P1**  
前置条件：`FQC-CPP-FMT-001`、`FQC-CPP-CI-001`

目标：损坏输入解析器在 sanitizer 下不 crash、不 OOM、不挂起，并把发现的 case 固化为普通回归测试。

允许修改：新增最小 fuzz CMake option/targets、`tests/corpus/`、CI fuzz smoke、回归测试。

第一阶段两个 target：

1. `archive_reader_fuzzer`：输入 bytes -> `ArchiveReader::open` -> 有界迭代 `readRawFrame/decodeRawFrame` 直到错误/footer；使用固定 64 MiB 操作预算和小 input max length；
2. `fastq_parser_fuzzer`：输入 bytes -> parser；分别覆盖 plain stream 和可控 gzip 解码入口时要限制解压后数据。

实施要求：

- fuzzer 只调用生产 parser，不复制解析逻辑；
- 错误 `Result` 是正常结束，panic/abort/sanitizer/timeout 才失败；
- corpus 以冻结合法 fixture、空、截断、字段边界、varint、损坏 zstd、footer/trailing bytes 为种子；
- PR 只跑短 smoke，例如每 target 30–60 秒；长期 fuzz 放 scheduled/manual；
- crash artifact 上传并生成最小化命令；修复后必须将最小 case 放进普通确定性测试，不能只留在 fuzz corpus；
- 不让 fuzzer 的 archive 声明绕过现有内存预算；
- 不为了 fuzz 公开新的生产 API。

验收条件：对空/任意 corpus fuzz smoke 无 sanitizer issue；人为移除一个关键 bounds check 可被已有 corpus 或短 smoke 捕获。短 fuzz 不是形式证明，文档不得写成“已证明安全”。

## 7. 任务 `FQC-CPP-META-001`：组织元数据与许可证清理

状态：**已决，P1；版权主体变更部分需负责人确认**

允许修改：`README.md`、`CMakeLists.txt`、`conanfile.py`、`LICENSE`、changelog。

实施要求：

1. active repository/clone/CI URL 改为 `open-genomics/fq-compressor`；
2. Conan `url` 与 CMake homepage 指向当前仓库；作者字段可保留真实个人署名，不应因仓库迁移自动改写历史作者；
3. 当前 Git 历史显示 `vendor/spring-core` 已删除，HEAD 也没有 `vendor/`。在再次检查发布 source archive 后，删除 LICENSE 中只针对该不存在路径的排除段；
4. MIT 标准许可正文不要随意改写；
5. `Copyright (c) 2026 LessUp` 是否改为个人名、组织名或追加新年份，属于权利人决定。实现模型不能仅凭 GitHub 组织迁移推断版权转让；先向负责人取得明确文本；
6. 依赖许可证通过 Conan/package NOTICE 或发行清单处理，不把动态依赖错误描述为本仓库 vendored code。

验收条件：当前发行 source tree 不引用不存在的 vendor exception；活动项目 URL 正确；版权行变更有负责人记录。

## 8. 后续发布门 `FQC-CPP-REL-001`

状态：**P2，`FQC-DEC-001` 已批准；仍等待格式 fixture 和 CI 完成**

在从 RC 进入稳定版本前必须满足：

- C++ 对 `fqc/.fqc` 的产品所有权决策已批准；
- 冻结 v2 fixture 在目标 release binary 上通过 verify/decompress；
- debug + ASan/UBSan 门禁通过；
- VERSION、tag、changelog 一致；
- release artifact 对小型 FASTQ 做 compress -> verify -> decompress -> byte compare；
- artifact checksum、最低 OS/CPU/libc、依赖许可证和已知限制齐全；
- Rust 同名项目的 README 已有迁移指引。

当前 `0.3.0-rc1` 阶段不急于搭建复杂多平台发布矩阵；先让格式契约和名称稳定。

## 9. 推荐顺序

1. `FQC-CPP-DOC-001` 与交叉不兼容警告；
2. `FQC-CPP-FMT-001`；
3. `FQC-CPP-CI-001`；
4. `FQC-CPP-FUZZ-001`；
5. `FQC-CPP-META-001`；
6. 在已批准的同名格式族契约、format fixture 和 CI 门禁落地后规划稳定 release。

不要把本轮工作变成 archive engine 重构。现有压缩/解压共享引擎是资产，修复应围绕它建立外部证明。
