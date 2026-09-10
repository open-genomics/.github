# FQC 同名产品与格式族治理

## 1. 已确认事实

工作区内两个独立仓库都使用：

- CLI 命令 `fqc`；
- 文件扩展名 `.fqc`；
- FASTQ 压缩器这一产品类别。

两者的 archive bytes 并不兼容：

| 属性 | `fq-compressor`（C++） | `fq-compressor-rust`（Rust） |
|---|---|---|
| 格式族 ID | `fqc-sequential` | `fqc-indexed` |
| 起始 magic | `46 51 43 56 32 0D 0A 1A`（`FQCV2\r\n\x1A`） | `89 46 51 43 0D 0A 1A 0A`，后跟版本 byte |
| 布局 | 32-byte 全局头、顺序 frame、40-byte footer | 可扩展全局头、索引 block、32-byte footer |
| 访问模型 | 顺序解码 | block index 与范围访问 |
| 当前成熟度 | 输出事务、结构化错误和 CI 较完整 | 规范、资源边界和 CI 尚待闭合 |

扩展名不是格式身份；magic 才是字节级判别依据。两个项目都叫 `fqc` 也不意味着二进制可以互换。

## 2. 决策 `FQC-DEC-001`

状态：**Accepted**  
决定日期：2026-08-13  
决定者：组织负责人（本轮用户确认）

### 2.1 决策值

1. C++ 与 Rust 项目都保留命令 `fqc`；
2. 两种 archive 都保留扩展名 `.fqc`；
3. 两种格式保持不同且稳定的 magic，不要求彼此解码；
4. C++ 格式族规范名为 `FQC Sequential Archive`，契约 ID 为 `fqc-sequential/v2`；
5. Rust 格式族规范名为 `FQC Indexed Archive`，契约 ID 为 `fqc-indexed/v2`；
6. 每个 reader 只接受自己格式族的 magic；看到另一格式族 magic 时必须返回专门的 `unsupported FQC format family` 错误，并指出正确实现；
7. 对未知 magic 返回 `not an FQC archive` 或等价错误，不把它误报为 checksum/codec/version 错误；
8. 默认输出文件名继续使用 `.fqc`，不得根据扩展名选择 decoder；
9. 两个独立仓库可以各自产生名为 `fqc` 的可执行文件，但组织不提供把二者同时安装到同一 `PATH` 的统一安装包。

### 2.2 共存边界

接受同名同后缀意味着把选择责任放在工具入口，而不是文件名上。为避免不可见歧义：

- 文档和发布页必须始终带实现限定词：`FQC C++ (sequential)` 或 `FQC Rust (indexed)`；
- shell 示例若上下文可能同时存在两个实现，应使用明确构建产物路径，而不是假设裸 `fqc` 指向某一个仓库；
- `--version` 输出必须包含实现族、实现版本和 archive format major；
- `info` 输出必须包含 `format_family` 和 `format_version`；
- 两个仓库不得在 README 中宣称自己是另一实现的 port、drop-in replacement 或兼容 reader，除非以后有跨实现测试证明；
- 包管理器发布名不得发生 registry 冲突。若目标 registry 已被另一仓库占用，需另开包装/发布名决策，但不改变 CLI 与 `.fqc` 后缀。

### 2.3 明确不做

- 不改变 Rust 的 `fqc` 产品名；
- 不引入新的 Rust 专用 archive 后缀；
- 不为了同名而合并两套格式；
- 不让一个 reader 内嵌另一项目 decoder；
- 不通过“先解析看看是否成功”猜格式族；
- 不复用或缩短现有 magic。

## 3. 格式族识别协议

两个 reader 在进入版本、header、checksum 或 codec 解析前，必须先完成固定长度 magic dispatch：

```text
读取前 8 bytes
  ├─ 46 51 43 56 32 0D 0A 1A -> fqc-sequential family
  ├─ 89 46 51 43 0D 0A 1A 0A -> fqc-indexed family
  └─ 其他                         -> unknown/not FQC
```

行为矩阵：

| 调用方 | 顺序 magic | 索引 magic | 未知/截断 magic |
|---|---|---|---|
| C++ `fqc` | 继续解析顺序 v2 | 拒绝并提示使用 Rust indexed 实现 | format/truncated 错误 |
| Rust `fqc` | 拒绝并提示使用 C++ sequential 实现 | 继续解析 indexed v2 | format/truncated 错误 |

错误消息可以调整措辞，但必须稳定地区分：`other known family`、`unknown magic`、`truncated header`。不得把另一格式族继续送进本实现的 version 或 checksum parser。

## 4. 任务 `FQC-DOC-001`：公开同名共存契约

状态：**可实施，P0**

目标：用户在进入任一仓库时，立即知道两个 `fqc/.fqc` 是同名、不同格式族的产品。

允许修改：

- `fq-compressor/README.md` 及格式文档首页；
- `fq-compressor-rust/README.md` 及格式文档首页。

跨仓库规则：这是一个组织任务，但必须拆成两个独立 change、两个独立提交和两次验证。

实施要求：

1. 两个 README 首屏添加对照表：仓库、实现语言、格式族 ID、完整 magic、访问模型、对方链接；
2. 明确扩展名不能判定格式，reader 必须检查 magic；
3. 明确两个实现不能互相解码；
4. 安装说明提醒同名二进制的 `PATH` 覆盖风险；
5. Rust 文档不得出现产品名或 archive 后缀迁移内容；
6. 不宣称跨实现自动分派已经存在；该能力由下一任务实现。

验收条件：

- 两个 README 搜索 `.fqc` 时都能在首次格式介绍附近看到共存说明；
- magic 以完整 hex 或精确 escaped bytes 展示；
- 文档不再使用“C++/Rust 版本”暗示同格式实现；
- 每个仓库仅产生自己的文档 diff。

## 5. 任务 `FQC-FAMILY-001`：已知格式族的快速拒绝

状态：**可实施，P0；在 `FQC-DOC-001` 后执行**

目标：两个 reader 对另一已知 FQC 格式族给出确定且可操作的错误，而不是普通 bad magic。

每个仓库分别建立一个 OpenSpec change，建议名称：

- C++：`recognize-indexed-fqc-family`；
- Rust：`recognize-sequential-fqc-family`。

实施要求：

1. 只增加 magic 分类和错误映射，不实现另一格式族解码；
2. 使用固定 8-byte 比较，不使用扩展名、文件大小或解析失败启发式；
3. 另一格式族 fixture 至少包含精确 magic 加最小占位 bytes；优先使用对方仓库冻结的真实最小 archive；
4. 错误中包括检测到的 family ID 和 canonical repository；
5. CLI 非零退出，且不会创建/截断输出文件；
6. library 层返回结构化 format-family 错误；若现有错误模型暂时不能新增枚举，先使用专门且测试锁定的格式错误，不顺手重构整个错误体系；
7. `info`、`verify`、`decompress` 三条入口都覆盖；压缩入口不受影响。

验收矩阵：

| case | C++ reader | Rust reader |
|---|---|---|
| 本族最小 archive | 接受 | 接受 |
| 对方最小 archive | family-specific reject | family-specific reject |
| 8 bytes 未知 magic | unknown reject | unknown reject |
| 0–7 bytes | truncated reject | truncated reject |
| 正确 magic、错误版本 | unsupported version | unsupported version |

非目标：跨格式转换、自动调用另一个可执行文件、修改 magic、修改后缀、统一 codec。

## 6. 任务 `FQC-IDENTITY-001`：CLI 身份可观察性

状态：**可实施，P1；建议在各自格式规范任务后执行**

目标：用户能够从运行结果而不是可执行文件名判断自己调用了哪个实现。

实施要求：

1. `fqc --version` 至少显示：产品名、implementation（C++/Rust）、实现版本、写入的 format family/major；
2. `fqc info <archive>` 的稳定字段至少包含 `format_family`、`format_version`、`implementation`；
3. 人类可读输出允许排版变化；若存在 `--json`，JSON 字段视为对外契约并需要 fixture；
4. README 的故障排查给出 `command -v fqc` 和 `fqc --version` 用法，帮助发现 `PATH` 覆盖；
5. 不增加运行时 launcher，也不试图让两个 binary 同时占用同一路径。

验收：两个仓库的输出可明显区分 `fqc-sequential/v2` 与 `fqc-indexed/v2`；版本测试不能只断言包含字符串 `fqc`。

## 7. 任务 `FQC-REG-001`：分别维护格式登记

状态：**可实施，P1；依赖两仓格式规范和 fixture**

根目录不是 Git 仓库，因此不建立根目录中央登记文件。每个仓库在自己的 `openspec/specs/archive-format/spec.md` 维护权威契约，并在 README 链接对方规范。

两个规范必须包含相同的登记字段：

| 字段 | C++ 值 | Rust 值 |
|---|---|---|
| Contract ID | `fqc-sequential/v2` | `fqc-indexed/v2` |
| Command | `fqc` | `fqc` |
| Extension | `.fqc` | `.fqc` |
| Magic | `46 51 43 56 32 0D 0A 1A` | `89 46 51 43 0D 0A 1A 0A` |
| Normative owner | `open-genomics/fq-compressor` | `open-genomics/fq-compressor-rust` |
| Golden fixture | 仓库内固定路径 | 仓库内固定路径 |
| Breaking policy | 新 major/magic + ADR | 新 major/magic + ADR |

不得在两个仓库复制对方完整格式布局；只复制识别信息和 canonical link，避免双重规范。

## 8. 实施顺序

1. 两仓分别完成 `FQC-DOC-001`；
2. Rust 完成 `FQCR-SPEC-001`，C++ 完成 `FQC-CPP-FMT-001`，各自冻结 archive fixture；
3. 两仓分别完成 `FQC-FAMILY-001`；
4. 两仓分别完成 `FQC-IDENTITY-001`；
5. 完成 `FQC-REG-001` 的双向规范链接；
6. 后续任何格式 major 变化都在拥有该格式的仓库内建立独立 change。

同名共存是明确产品决定，不再把重命名作为后续任务。

## 9. 撤销记录 `FQCR-RENAME-001`

状态：**Not applicable，被 `FQC-DEC-001` 取代**

后续模型不得实施 Rust 命令改名、crate/binary 改名或专用后缀迁移。若未来重新考虑命名，必须创建新的 decision 和 change，不能恢复本任务或沿用旧设计假设。
