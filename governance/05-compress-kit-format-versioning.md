# CompressKit 二进制格式版本化设计

目标仓库：`/home/shane/github/open-genomics/compress-kit`  
审计提交：`aa6472604fe2`

## 1. 问题定义

仓库把“二进制格式兼容性”列为首要约束，但当前 Unreleased 相对已发布 `v1.0.0` 已经发生多项破坏性变化：

- 所有当前格式新增覆盖整段内容的 4-byte little-endian CRC-32 trailer；
- Range Coder 修正 renormalisation，旧 `RCNC` payload 不再可解码；
- RLE 从 v1 的裸 `(count, value)` 序列演进到带 magic 的格式；
- 原始/压缩大小契约发生变化。

Huffman、Arithmetic 和 Range 仍沿用 v1 magic：`HFMN`、`AENC`、`RCNC`。decoder 无法在读取 magic 时可靠地区分 v1 与当前格式，只能把旧 payload 的末 4 bytes 当成 checksum 再失败。常量注释还写着这些 magic “MUST NOT change”，与已经改变 payload 语义的现实冲突。

结论：这是一次应发布为 v2 的格式变化，必须拥有新的、无歧义的 wire identity。仅在 changelog 写 `BREAKING` 不能替代版本判别。

## 2. 决策 `CK-DEC-001`

状态：**Accepted（2026-08-13，组织负责人确认）**

### 2.1 已批准决策

将当前“新 payload + CRC trailer”定义为 format v2，为四种算法分配新的 4-byte magic：

| 算法 | v1 / legacy 标识 | 推荐 v2 magic |
|---|---|---|
| Huffman | `HFMN` | `HFM2` |
| Arithmetic | `AENC` | `AEN2` |
| Range | `RCNC` | `RCN2` |
| RLE | v1 无 magic；中间未发布版本为 `RLE\0` | `RLE2` |

选择新 magic 而不是在旧 magic 后追加版本 byte，原因是旧格式中第 5 byte 已有含义；追加 byte 会产生可碰撞的启发式判断。新的 magic 使 parser 在读取前 4 bytes 后就能作出确定选择。

推荐兼容策略：

- writer 只写 v2；
- v2 decoder 不实现 v1 解码，符合项目已经选择的轻量策略；
- 对可识别的 v1 magic 返回 `unsupported legacy v1 format`，不返回含糊的 checksum mismatch；
- v1 RLE 没有 magic，无法可靠自动识别，统一返回 `bad magic; expected RLE2`；
- 不支持当前未发布的 CRC + 旧 magic 中间格式。它没有稳定发布契约。

### 2.2 版本语义

这里的 `2` 是格式 major，不是算法版本或压缩级别。任何使既有 decoder 无法正确解释字节的变化都必须使用新 magic；只改变实现性能而保持完全相同 bytes/语义则不升格式版本。

### 2.3 批准记录

```text
CK-DEC-001: Accepted
Huffman v2 magic: HFM2
Arithmetic v2 magic: AEN2
Range v2 magic: RCN2
RLE v2 magic: RLE2
Legacy policy: reject-with-specific-error
Package release version: 2.0.0
Approved by: organization owner
Date: 2026-08-13
```

如果选择支持 v1 只读解码，需要另写 `CK-V1-READER-001` 设计和四算法 fixture；不能让 `CK-FMT-001` 顺手膨胀成兼容层。

## 3. v2 规范

### 3.1 通用规则

- 所有多字节整数为 little-endian；
- 整个 stream 结构为 `v2 magic + algorithm body + crc32`；
- trailer 是对 trailer 之前全部 bytes 的 IEEE/zlib-compatible CRC-32；
- decode 先验证最小长度和 magic，再验证 CRC，再解析 body；
- 所有长度/频率计算必须使用 checked arithmetic 并遵守大小上限；
- magic 和 CRC 只提供格式识别/随机损坏检测，不提供恶意篡改认证。

### 3.2 v2 布局

```text
Huffman:
| HFM2 | symbol_count:u32 | 257 × frequency:u32 | bitstream | crc32:u32 |

Arithmetic:
| AEN2 | symbol_count:u32 | 257 × frequency:u32 | bitstream | crc32:u32 |

Range:
| RCN2 | symbol_count:u32 | 257 × frequency:u32 | bytestream | crc32:u32 |

RLE:
| RLE2 | zero or more (count:u32, value:u8) | crc32:u32 |
```

规范还必须明确空输入、EOF frequency、frequency sum、最小 payload、RLE count 非零、raw/compressed 上限的边界是 `<` 还是 `<=`。不要只依赖代码常量名推断。

## 4. 任务 `CK-FMT-001`：实施新 magic 和明确错误

状态：**可实施，P0；`CK-DEC-001` 已批准**

允许修改：

- `algorithms/shared/cpp/include/compresskit/constants.hpp`；
- 序列化/算法 parser；
- 生命周期和 CLI conformance 测试；
- README、架构/算法文档、CHANGELOG；
- CMake 项目版本或新增单一 VERSION 文件。

实施要求：

1. 使用批准的四个 v2 magic；常量名称应携带版本，例如 `HUFFMAN_V2_MAGIC`，避免未来误以为永不变化；
2. writer 只输出 v2 magic；
3. decoder 读取 magic 后：
   - v2：继续 CRC 和 body 解析；
   - 已知 v1/legacy：返回明确 unsupported version；
   - 其他：返回 bad magic；
4. 不用“最后 4 bytes 是否恰好等于 CRC”来猜版本；
5. package 版本按 SemVer 发布为 2.0.0，并让 CMake/project/doc/changelog 使用同一版本来源；
6. 更新所有图、示例 hex、注释和算法页；
7. 不改变本任务之外的算法 bitstream。Range 当前修复后的 bitstream 作为 v2 基线；
8. decoder 错误至少区分 bad magic、legacy unsupported、checksum mismatch、truncated header/body、size limit。

验收条件：

- 四个空输入和典型输入的编码前 4 bytes 均为批准值；
- v1 fixture 不会进入 v2 body parser；
- 任意 v2 byte 损坏由 CRC 或结构校验拒绝；
- 文档中没有把 v1 magic 描述为当前 v2；
- changelog 将整批格式变化归入 2.0.0，而不是无限停留在 Unreleased。

验证：

```bash
make lint
make test
```

非目标：改变 CLI 形状、重新设计压缩算法、增加通用容器格式。

## 5. 任务 `CK-TEST-001`：格式 fixture 与版本回归矩阵

状态：**已决，P0；v2 fixture 在 magic 决策实施后生成**

目标：格式兼容不再只靠 current-writer/current-reader round-trip。

允许修改：`tests/fixtures/`、conformance runner、CTest 注册、fixture README。

### 5.1 fixture 集

每种算法至少提交：

- `v1-minimal`：由 `v1.0.0` checkout 的 C++ binary 生成；
- `v2-empty`；
- `v2-small`：共同固定输入，例如包含 `00/FF`、重复、ASCII 和全部 256 byte；
- `v2-corrupt` 可由测试从合法 fixture 确定性派生，不必提交许多几乎相同文件。

注意：v1 RLE 空输入编码为空文件，不具备识别信息；其 fixture 仍用于记录历史事实，但预期错误是 bad magic，不宣称版本可识别。

### 5.2 manifest

建议单一 TSV/JSON manifest 包含：algorithm、format_version、source_commit、generator_command、input path/hash、archive path/hash、expected action（decode/reject）和原因。测试从 manifest 参数化运行，避免四份手写逻辑。

### 5.3 测试规则

- v2 frozen archive 必须由以后版本持续成功解码；
- v1 fixture 按批准策略稳定拒绝或只读解码；
- current writer 输出只断言规范规定的稳定 bytes；
- 因算法实现优化而改变 v2 bitstream 时：若 decoder 仍兼容，允许 writer bytes 变化，但 frozen decoder fixture 必须继续通过；
- 如果算法格式要求 canonical bytes，则规范必须明确，之后才比较完整 writer SHA；默认只承诺可解码格式，不承诺唯一压缩表示。

验收条件：把 magic、CRC endian 或 frequency layout 任意改错都会有直接失败的测试，不只是 round-trip 自洽成功。

验证：`make test` 必须包含 fixture suite，不单独依赖维护者记得运行脚本。

## 6. 任务 `CK-DOC-001`：修正文档与上限契约

状态：**已决，P0**

已知文档错误：架构页仍写“编码输入严格小于 4 GiB”，实现常量和同页安全表已经是严格小于 1 GiB。

实施要求：

1. 所有用户文档统一为：encode raw input `< 1 GiB`；decode output `<= 1 GiB` 或实现真实边界（实现需同步精确核对）；compressed decode input `< 8 GiB`；
2. 明确当前实现是全量内存 buffer，不是 streaming，因此实际峰值高于单个输入上限；
3. README 明确项目是教学/验证用途，并说明当前未发布格式将按已批准的 `CK-DEC-001` 收敛为 v2；在 `CK-FMT-001` 完成前不能把工作树状态描述为已发布稳定 v2；
4. CHANGELOG 修正“magics and field layouts are otherwise unchanged”这类会掩盖 RLE 历史差异的表述；
5. 在决策批准前，旧 magic 的命中必须明确标注为 v1/legacy 或“当前未发布格式”，不得擅自写入推荐 v2 magic；格式页、最终版本号和新 magic 由 `CK-FMT-001` 统一修改；
6. 文档站的 changelog 若由根文件生成，继续只维护一个事实源，不手改生成副本。

验收条件：

```bash
rg -n '4 GiB|4GiB|HFMN|AENC|RCNC|RLE\\x00' README.md docs CHANGELOG.md
```

`4 GiB` 的剩余命中只能是明确的历史说明；旧 magic 的剩余命中必须标注为 v1/legacy 或当前未发布格式，不能被描述为已批准的稳定 v2。

## 7. 任务 `CK-LIMIT-001`：分配前执行文件大小检查

状态：**已决，P1**

当前文件路径先由 `read_file` 整体分配，再在 `encode_buffer/decode_buffer` 检查上限。这意味着一个超大输入会在返回 `ERR_SIZE_LIMIT` 之前造成大分配。

目标：文件 CLI 在分配前拒绝超限输入，buffer API 继续保留自己的边界检查。

实施要求：

1. `read_file` 或调用者接收该操作允许的最大输入；用 `std::filesystem::file_size` 或已经打开 stream 的安全长度先检查；
2. encode 使用 `MAX_RAW_SIZE`，decode 使用 `MAX_COMPRESSED_SIZE`；边界与 buffer API 完全相同；
3. 在转换为 `size_t`/`streamsize` 前做范围检查；
4. 空文件、正好边界、边界减一、超边界和无法 seek/read 的路径都有测试；
5. 错误消息包含操作与允许上限，CLI 返回非零；
6. 不为了支持 8 GiB 假装现有全量内存架构适合普通机器。可以在后续另行降低产品上限，但本任务先让声明和执行顺序一致。

验收条件：稀疏超限文件在测试中快速返回，RSS 不随声明文件大小增长。

非目标：本任务不引入 streaming codec。

## 8. 任务 `CK-IO-001`：文件输出失败原子性

状态：**已决，P2**

目标：写文件中途失败不损坏已存在目标，也不留下被误认为有效的部分文件。

实施要求：在同目录创建唯一临时文件，完整写入并检查 close/flush 后 rename；失败清理 temp。不要在 transform 成功前创建最终输出。Linux/macOS/Windows 的替换语义要由测试覆盖或在支持范围中明确。错误不得被 `catch (...)` 静默压成无说明的 `false`。

验收：已有目标 + 模拟写失败时旧内容不变；成功时完整替换；临时文件不残留。

## 9. 条件后续任务 `CK-V1-READER-001`

状态：**Not applicable；`CK-DEC-001` 已选择明确拒绝 legacy**

本轮不得实现 v1 reader。若未来改变策略，必须用新的决策取代 `CK-DEC-001`，并先补独立设计，至少回答四种算法各自的 v1 精确布局、v1 RLE 无 magic 的显式选择方式、旧 Range Coder 行为、输入资源上限、fixture 来源以及停止支持策略。不能用启发式探测把未知损坏文件猜成 v1。

## 10. 推荐提交拆分

1. `docs(format): record v2 magic decision and exact layout`
2. `test(format): add v1 rejection and v2 golden fixtures`
3. `fix(format): identify all CRC formats as v2`
4. `fix(io): reject oversized files before allocation`
5. `fix(io): commit output files atomically`
6. `chore(release): prepare 2.0.0 format release`

格式 magic、fixture 与 release version 必须在同一发布序列内完成；不能先发布 CRC 格式并承诺稍后再版本化。
