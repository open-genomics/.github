# fq-compressor-rust 加固设计

目标仓库：`/home/shane/github/open-genomics/fq-compressor-rust`  
审计提交：`1a2a2161bed8`

说明：本文沿用初次审计形成的 `FQCR-*` 任务编号以保持引用稳定；该前缀不是产品改名。负责人已决定 Rust 产品继续使用命令 `fqc` 和后缀 `.fqc`，格式族 ID 为 `fqc-indexed/v2`。

## 1. 已确认问题

### 1.1 规范与实现不一致

实现将 codec byte 定义为 `(family << 4) | version`，当前 family 包括 Raw、ABC、SCM、Delta、Zstd 等；规范却把 `0x01` 写成 Zstd、`0x02` 写成 ABC。实际 writer 可产生如 `0x10`（ABC v0）、`0x70`（ZstdPlain v0）、`0x40`（DeltaZstd v0）。

实现中 `ChecksumType::XxHash64 = 0`，规范却声明 `0=None, 1=XxHash64`。规范还画出了 v1 fallback 流程，而读取器对任何 `major != 2` 都返回 `UnsupportedVersion`。

结论：当前代码已经产生真实文件，优先把当前实现字节定义为现行 v2 事实，再通过测试固化。不能仅按旧文档改 enum 数值，否则会在不升版本的情况下破坏现有文件。

### 1.2 block header 没有完全驱动解码

`BlockHeader` 保存 `codec_ids`、`codec_seq`、`codec_qual`、`codec_aux`，但 `BlockCompressor::decompress_raw` 只接收 `codec_seq`。ID、quality 和 aux 的 decoder 来自全局 flags/config。

这造成两个问题：

- header 看似自描述，实际只有 sequence codec 生效；
- 损坏或未来未知 codec 可能被错误 decoder 接受，错误定位不清。

### 1.3 输出不是事务性的

普通、streaming 和 pipeline 路径都存在直接对最终目标调用 `FqcWriter::create` 的路径。`--force` 场景下，旧文件可能在压缩完成前被截断；中途失败会留下看似正式的部分 archive。解压到普通文件也直接 `File::create`，split-PE 会依次创建两个目标，缺少双文件原子提交语义。

### 1.4 解压资源上限未闭合

全局 `--memory-limit` 只进入压缩选项。读取器虽然会验证 offset/extent，但仍根据 archive 内声明值分配 block stream、index、reorder map 和全部原始顺序 reads。恶意或损坏文件可以制造超大分配或解压膨胀。

## 2. 目标架构

```text
CLI options
  ├─ compression request ─┐
  └─ decompression limits ├─ operation-scoped resource policy
                         │
archive reader
  ├─ structural validation
  ├─ codec/checksum validation
  ├─ checked size accounting
  └─ bounded decode

output transaction
  ├─ same-directory temporary file(s)
  ├─ flush + close
  └─ successful commit/rename only
```

核心原则：格式字段必须有唯一语义；所有来自 archive 的计数在转换为 `usize` 或分配前经过 checked conversion 和预算检查；所有普通文件输出只有在完整成功后才替换最终路径。

## 3. 任务 `FQCR-SPEC-001`：校准 v2 规范并冻结字节契约

状态：**已决，P0**

目标：规范准确描述审计提交实际读写的 v2 格式，并用结构测试防止再次漂移。

允许修改：

- `docs/reference/format-spec.md`；
- `src/archive/format.rs` 中与显式验证、测试有关的代码；
- `src/types.rs` 中 codec/checksum 的转换 API 和测试；
- `tests/` 与 `tests/data/`；
- fixture 来源说明。

实施要求：

1. codec 表改为高 4 bit family、低 4 bit version，并列出代码中全部已分配 family；
2. checksum 表声明 `0 = XxHash64`，不要虚构当前不存在的 `None` 编码；
3. 兼容性声明改为“当前读取器只接受 major 2”；删除“尝试 v1 parser”的图和版本历史中的虚构实现；
4. 校正最小 archive 示例的 magic/version offset；按实际实现逐字段复核 header、index、reorder map 和 footer；
5. 为 `CodecFamily` 增加严格解析路径：未知/Reserved 值在 archive 读取或解码前返回格式错误。现有宽松 `from_u8` 可以保留给展示用途，但不能让 decoder 把未知 family 默认为其他算法；
6. 增加 table-driven 测试，覆盖每个当前 writer 能产生的 codec ID 及 checksum ID；
7. 创建至少一个由审计提交生成的冻结 v2 archive：
   - 输入足够小，可人工检查；
   - README 记录生成提交、完整命令、输入 SHA-256、archive SHA-256、预期 reads；
   - 测试必须解码冻结 archive 并逐记录比较；
   - writer 测试断言稳定 header 字段，不要求所有未来 zstd 版本输出整文件 SHA 不变。

验收条件：

- 规范中的每个枚举值都由同一张测试表验证；
- 当前 writer 生成文件能被当前 reader 解码；
- 冻结 fixture 能被 reader 解码；
- 未知 codec、未知 checksum、错误 major 分别产生可识别错误；
- 没有改变现有 v2 writer 的已分配字节值。

验证：

```bash
cargo fmt --all -- --check
cargo clippy --all-targets -- -D warnings
cargo test --lib --tests
cargo doc --no-deps
```

非目标：产品命名或后缀变化、支持 v1、修改压缩算法。

## 4. 任务 `FQCR-CODEC-001`：让每个 stream codec 字段成为解码真相

状态：**已决，P0**  
前置条件：`FQCR-SPEC-001`

目标：解码器依据 block header 的四个 codec 字段选择实现，并在未知或不适用于该 stream 时拒绝文件。

允许修改：

- `src/algo/block_compressor.rs` 及已有 codec 实现；
- `src/archive/` 的 header 验证；
- `src/commands/decompress.rs`、`verify.rs` 与 pipeline 解压调用点；
- 对应单元、集成和 fixture 测试。

实施要求：

1. `decompress_raw` 接收四个 codec ID，或直接接收包含它们的 `BlockHeader`；不要继续只传 `codec_seq`；
2. 为每类 stream 建立显式允许矩阵：

   | Stream | 当前合法 family |
   |---|---|
   | IDs | `Raw`（discard）、`DeltaZstd` |
   | Seq | `AbcV1`、`ZstdPlain` |
   | Qual | `Raw`（discard）、`ScmV1`、`ScmOrder1` |
   | Aux | `DeltaVarint`；若 uniform length 且空 stream，仍验证 header 约定 |

3. family 合法但 version 未实现时返回 unsupported codec version，不可悄悄按 v0 解码；
4. 全局 flags 仍描述用户语义（quality mode、ID mode），但不能覆盖 block 上声明的算法；两者矛盾时返回格式错误；
5. `verify` 和所有普通/pipeline 解压路径必须复用相同分派逻辑；
6. 测试逐一篡改四个 codec byte，证明每个字段均被读取和验证。

验收条件：任一 stream 的 codec 改成另一个合法但不适用的 family 都会稳定失败，且错误包含 block ID、stream 名和 codec byte。

验证：同 `FQCR-SPEC-001`，并单独运行新增 codec 测试。

非目标：插件式 codec 注册、动态加载、自定义 external codec。

## 5. 任务 `FQCR-IO-001`：普通文件输出事务

状态：**已决，P0**  
前置条件：无；可与规范任务独立实施，但不得与格式族识别或 CLI 身份任务混在同一提交。

目标：压缩或解压失败时，目标不存在则仍不存在；目标已存在且使用 `--force` 时，旧内容保持不变。

允许修改：

- 新增一个职责单一的内部 output transaction 模块；
- 所有压缩 engine/pipeline writer 创建点；
- `src/commands/decompress.rs` 的普通文件与 split-PE 输出；
- 失败注入和集成测试；
- 如生产实现需要，可将 `tempfile` 从 dev-dependency 调整为 dependency。

实施要求：

1. 临时文件必须创建在最终目标所在目录，以便最终 rename 保持同一文件系统；
2. 临时名称不能与已有文件冲突，权限遵循安全默认值；
3. 成功路径必须 flush writer、传播关闭前错误，再 commit；
4. 失败或 drop 自动清理临时文件；
5. `--force` 只允许在 commit 时替换目标，不能预先 truncate/unlink 旧目标；
6. split-PE 是一个逻辑事务：两个临时输出都成功后才提交。由于 POSIX 不提供跨两个路径的真正原子 rename，采用：
   - 完成两个 temp；
   - 检查两个目标的覆盖策略；
   - 按稳定顺序提交；
   - 若第二次提交失败，尽最大可能恢复第一个目标；
   - 文档明确这一平台限制。
7. stdout 不走文件事务，不尝试回滚已经写出的 pipe 数据；
8. 不在各 engine 中复制临时文件逻辑，所有模式复用同一抽象。

必须测试：

- 目标不存在 + 中途错误；
- 目标存在、无 `--force`；
- 目标存在、有 `--force` + 中途错误；
- 成功替换；
- split-PE 的两个输出；
- streaming、pipeline 和普通路径各至少一个成功测试。

验收条件：上述失败测试验证最终路径内容和临时文件清理，而不只检查返回值。

验证：

```bash
cargo fmt --all -- --check
cargo clippy --all-targets -- -D warnings
cargo test --lib --tests
```

非目标：跨机器分布式事务、为 stdout 缓冲整个结果。

## 6. 任务 `FQCR-LIMIT-001`：解压和校验资源预算

状态：**已决，P0**  
前置条件：`FQCR-SPEC-001`；最好在 `FQCR-CODEC-001` 后实施。

目标：`--memory-limit` 对压缩、解压和完整 verify 都有明确且可测试的约束，损坏 archive 不会导致按声明值无限分配。

允许修改：

- CLI 参数传递；
- `DecompressOptions`、verify options；
- `memory_budget.rs`；
- archive reader 和各 decoder 的边界校验；
- 专项损坏输入测试。

实施要求：

1. 将用户提供的 MB 转为 operation-scoped `MemoryBudget`，不要使用可变全局状态；
2. 所有 `u64 -> usize` 使用 checked conversion；在 `Vec` 分配前先验证：
   - 值没有溢出；
   - stream 大小不超过 block payload extent；
   - 单 block 预估内存不超过预算；
   - index entry 数量与文件可容纳的最小 entry 数一致；
   - reorder map 的压缩和解压后上限与 `total_reads` 一致；
3. zstd 解压必须有输出上限，不能对攻击者控制的数据直接无界 `decode_all`；
4. original-order 模式在开始前估算 map、records、strings 和当前 block 的峰值；超预算应在输出创建前失败；
5. parallel batch size 必须由线程数与预算共同决定；预算不足以容纳一个合法 block 时明确报错，不把并发减到 0；
6. 自动预算（参数为 0）仍需有确定规则和硬性结构上限；在 README 解释“automatic”并非无限；
7. 错误包含声明值、允许值和结构位置，避免只报 allocation failed；
8. `--quick verify` 只做结构/checksum 的话可使用更小预算；完整 verify 与解压共享 decode 限制。

必须构造的测试 archive：超大 block count、溢出 offset、超大 stream size、zstd 膨胀、reorder total_reads 不一致、original-order 超预算。

验收条件：每个样本都返回受控 `Result`，不 panic、不 OOM、不长时间挂起；正常最小 fixture 在合理低预算下通过。

验证：同 `FQCR-SPEC-001`。如添加 fuzz target，只作为补充，不替代确定性回归测试。

非目标：精确预测 Rust allocator 的每一个字节；预算应保守但不过度复杂。

## 7. 任务 `FQCR-CI-001`：最小持续集成

状态：**已决，P1**  
前置条件：上述本地验证命令稳定通过。

目标：为当前约 1.3 万行 Rust 和外部格式解析器建立一份低维护成本的 CI。

允许修改：`.github/workflows/ci.yml`、必要的依赖审计配置和 README badge。

单一 workflow 最小 job：

```text
quality:
  cargo fmt --all -- --check
  cargo clippy --all-targets -- -D warnings
  cargo test --lib --tests
  cargo doc --no-deps
```

建议追加独立、可清楚归因的依赖审计 job；如果选用 `cargo audit`，必须固定安装方式或使用固定 major 的可信 action，不使用浮动 `master`。缓存 key 至少包含 `Cargo.lock` 哈希。

验收条件：

- PR 与默认分支 push 都执行；
- 任一命令失败会使 check 失败；
- 没有静默 skip；
- 不同时引入文档站、发布自动化或复杂矩阵；
- workflow 使用仓库要求的稳定 Rust/MSRV 策略，并记录选择。

非目标：一开始就加入多平台、多 Rust 版本、覆盖率和 nightly fuzz。先保证最小门禁可信。

## 8. 推荐提交拆分

1. `docs(format): align v2 specification with encoded bytes`
2. `test(format): freeze v2 decoder fixture and codec table`
3. `fix(codec): dispatch every block stream from its header`
4. `fix(io): commit archive outputs transactionally`
5. `fix(decode): enforce operation memory budget`
6. `ci: add minimal Rust quality gate`

每个提交必须独立通过测试。负责人已经决定保留 `fqc/.fqc`；格式族识别和 CLI 身份属于独立 change，不应塞进上述提交。
