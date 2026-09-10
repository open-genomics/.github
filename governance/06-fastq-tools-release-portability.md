# fastq-tools 发布可移植性设计

目标仓库：`/home/shane/github/open-genomics/fastq-tools`  
审计提交：`5bab799bb9be`

## 1. 总体判断

这是六个仓库中工程化最成熟的项目：当前 CI 已包含 format、clang-tidy/cppcheck、GCC/Clang、ASan/TSan/UBSan、fuzz smoke、覆盖率阈值和 packaging consumer 测试；输出事务、内存模型和性能证据也已有专门回归。

因此本设计不建议重写构建脚本或再增加抽象层。当前需要修复的是发布边界：

1. Release 默认使用 `-march=x86-64-v3`，却被注释称为“可移植”；
2. 仓库有 `scripts/ci/release-build` 和 release 徽章，但没有本地 `.github/workflows/release.yml` 闭环；
3. release script 与 CI 的 CMake/Conan 版本已经漂移；
4. README/CMake 仍指向旧组织 `LessUp`；
5. 审计 checkout 中没有本地 Git tags，因此不能仅凭 changelog 的历史版本段证明当前 release ref 已建立。实施发布任务时需先核对远端，而不是猜测。

## 2. CPU 基线风险

`x86-64-v3` 要求 AVX2、FMA、BMI 等指令集，不是所有 64-bit x86 CPU 都支持。当前 `quality_trimmer.cpp` 会在 `__AVX2__` 下编译另一条路径，所以这不仅影响速度，也会改变实际进入的实现。虽然已有回归保证两条路径语义一致，默认发布二进制仍会在旧 CPU 上以 illegal instruction 启动失败。

正确模型应区分：

```text
portable      发行与通用部署；使用编译器 x86-64/arm64 默认基线
x86-64-v3     用户明确选择的已知现代 x86 部署
native        仅本机 benchmark/本地运行，不可发布
```

本阶段不引入 runtime CPU dispatch。当前 AVX2 优化范围有限，先让发布可运行性可预测，之后只有 benchmark 证明收益才评估多版本分派。

## 3. 任务 `FQT-CPU-001`：显式 CPU profile

状态：**已决，P0**

目标：默认构建和发布产物可在其声明的基础架构上运行；优化 profile 只能显式选择。

允许修改：

- `CMakeLists.txt`、`CMakePresets.json`；
- `scripts/core/build` 与 `scripts/ci/release-build`；
- CPU 相关单元/CI 配置；
- README、构建文档和 changelog。

实施要求：

1. 新增 CMake cache string：

   ```cmake
   FQTOOLS_CPU_BASELINE=portable|x86-64-v3|native
   ```

   默认 `portable`，配置时校验枚举值并打印最终 profile。
2. `portable` 不添加 `-march`，使用目标平台编译器默认 ABI/ISA；不得继续在 x86 自动加 v3；
3. `x86-64-v3` 只允许在 x86-64 + 支持该 flag 的 GCC/Clang 上配置，否则 CMake 明确失败；
4. `native` 添加 `-march=native`，文档标注“不可分发”。release workflow 必须显式拒绝该值；
5. 旧 `ENABLE_NATIVE_ARCH` 作为已存在的构建接口，先提供一个版本的迁移：
   - ON 且新 profile 未显式设置时映射到 `native` 并发出 deprecation warning；
   - 与显式非 native profile 冲突时配置失败；
   - 在下一次 major 或有记录的日期移除。不要长期维护两个真相。
6. 增加 `portable` 和 `x86-64-v3` 两个 Release CI 构建。至少 quality trimmer 回归在两个 profile 都运行，推荐完整测试；
7. benchmark preset 可以显式选择 v3/native，但 benchmark report 必须记录 profile 和 CPU flags；
8. `ENABLE_FAST_MATH` 继续默认 OFF，不与 CPU profile 混合。

验收条件：

- clean Release configure 默认 cache 为 `portable`，compile commands 不含 `-march=x86-64-v3`/`native`；
- v3 profile 的 compile commands 含且只含批准的 v3 flag；
- native profile 无法进入发布 job；
- portable 与 v3 的 stat/filter 输出逐字节或逐字段一致；
- 发布文档准确声明 OS、architecture、libc 和 CPU minimum；
- 在基础 x86-64 runner/模拟器上运行实际发布 binary 的 `--help` 和小型 smoke，不出现 illegal instruction。

验证：

```bash
./scripts/core/build --preset clang-release --clean
./scripts/core/test --preset clang-release

# 实现者应新增对应 preset 或脚本参数，以下名称只是目标语义：
./scripts/core/build --preset clang-release-v3 --clean
./scripts/core/test --preset clang-release-v3

./scripts/core/lint format-check
```

最终命令名称由实现者按现有脚本风格确定，并同步帮助文本；不要让用户只能通过隐藏的 `-D` 才能选择 profile。

非目标：AVX2 runtime dispatch、AVX-512、NEON 手写优化、性能重构。

## 4. 任务 `FQT-REL-001`：建立保守的发布闭环

状态：**已决，P1**  
前置条件：`FQT-CPU-001`、`FQT-META-001`

目标：从已批准 tag 到 draft GitHub Release 的构建、测试、打包和校验全部可重放，不依赖维护者本机知识。

### 4.1 初始支持范围

第一阶段只发布已经实际验证的 Linux x86-64 glibc `portable` artifact。musl、macOS、ARM 和 Windows 只有在各自 runner 上通过构建、CLI smoke、依赖/链接检查后才加入矩阵。尤其不要在 `issues/002` 仍为 open 时宣称 macOS 正式支持。

可选 musl 静态包进入发布前，`ldd/file` 验证必须是硬失败门禁；当前脚本只打印 warning 不足以证明静态链接。

### 4.2 触发和审批

推荐 workflow 使用手动 `workflow_dispatch`，输入一个已存在的 `vX.Y.Z` tag：

1. checkout tag 指向的完整 SHA；
2. 验证 tag 名与 CMake 版本完全相同；
3. 验证 changelog 存在对应版本条目；
4. 验证 tag 指向默认分支可达的 commit；
5. 使用受保护的 `release` environment 要求维护者批准；
6. 只创建 draft release，由维护者检查后手动发布。

这样避免仅推错 tag 就立刻公开资产。若未来改成 tag 自动发布，必须另行记录权限与回滚方案。

### 4.3 复用质量门禁

不要在 release workflow 手抄一份逐渐漂移的 CI。推荐给现有 `ci.yml` 增加 `workflow_call` 入口，让 release workflow 以 reusable workflow 运行同一套门禁，并使 artifact build job `needs` 该调用成功。需要评估完整 fuzz/TSan 时间时可以保留完整发布门禁，而不是跳过最重要的 parser 检查。

### 4.4 artifact 构建与验证

每个发布目标必须：

1. 在固定 runner/container 环境构建 `FQTOOLS_CPU_BASELINE=portable`；
2. 使用提交的 Conan lock；
3. 安装到 staging prefix，而不是直接从 build tree 随意复制；
4. 对 staging 运行：
   - `FastQTools --help` 与版本输出；
   - 最小 plain/gzip `stat`；
   - 最小 `filter` 并回读结果；
   - packaging consumer 验证（若发布开发包）；
   - 动态依赖白名单或静态链接硬校验；
   - 基础 CPU 兼容 smoke；
5. 包名编码产品版本、OS、architecture、libc 和 CPU profile；
6. 生成 `SHA256SUMS`，资产数与 matrix 精确匹配；
7. 上传失败或缺文件必须失败，不得 `if-no-files-found: ignore`；
8. release notes 包含编译器、依赖 lock、最低 CPU/OS、已知限制和 changelog 链接。

验收条件：

- 任一 required quality job、版本检查、artifact smoke 或 checksum 失败都不会创建 draft；
- 从 release asset 解压后的 binary，而不是 build tree binary，完成 smoke；
- 重跑同一 tag 不会静默覆盖不同 bytes；若重建不保证 bit-reproducible，应先删除 draft/asset 或使用显式重建流程并记录原因；
- workflow 权限默认 `contents: read`，只有创建 draft 的 job 提升 `contents: write`；
- 第三方 actions 固定到完整 commit SHA。

验证：先在 fork/测试 tag 上执行完整 draft 流程并删除测试 draft；正式 tag 不用于试错。

非目标：Homebrew、Conda、Docker registry、macOS/Windows 多平台一次性发布。

## 5. 任务 `FQT-TOOLCHAIN-001`：消除 release toolchain 漂移

状态：**已决，P1**  
前置条件：可与 `FQT-REL-001` 一起实施，但应保持独立 commit。

已知漂移：CI 使用 CMake `4.3.4` / Conan `2.27.1`，`release-build` 内写死 CMake `4.0.2` / Conan `2.24.0`。

目标：CI、release 和文档从一个小型版本 manifest 读取构建工具版本。

实施要求：

1. 新增简单、机器可读且 shell 可安全加载的 manifest，例如 `build-config/toolchain-versions.env`；只存构建工具版本，不复制依赖 lock；
2. CI 第一步验证并写入 `GITHUB_ENV`，release script source 同一文件；禁止调用方通过未审计环境变量悄悄覆盖正式发布版本；
3. 下载 CMake archive、LLVM installer 或其他二进制时验证固定 SHA-256，或者改用固定 digest 的可信构建镜像；不能只依赖 HTTPS + 可变脚本 URL；
4. `release-build` 的 shell 参数使用数组，修复未引用的 `${BUILD_DIR}` 调用点；
5. release build 不在运行中执行不可复现的系统全量 upgrade；
6. CI 测试修改 manifest 后所有消费者都采用新值，避免另一份硬编码继续存在。

验收条件：

```bash
rg -n '4\.3\.4|2\.27\.1|4\.0\.2|2\.24\.0|LLVM_VERSION' \
  .github scripts build-config CMakeLists.txt
```

版本值只出现在 canonical manifest、必要的 lock/checksum 表和历史 changelog，不再散落两套活动配置。

非目标：自动升级工具版本。

## 6. 任务 `FQT-META-001`：修正归属和发布真相

状态：**已决，P1**

允许修改：`README.md`、`CMakeLists.txt`、package/config metadata、changelog 的新条目。

实施要求：

1. 将活动链接从 `LessUp/fastq-tools` 改为 `open-genomics/fastq-tools`；
2. 作者个人 GitHub 链接与组织归属分开：保留真实作者署名可以，但项目主页、clone、issues、CI、release badge 必须指向当前仓库；
3. 在真正存在 release workflow/tag/asset 前，release badge 不得暗示已有受维护发布；可暂时移除，待 `FQT-REL-001` 完成再恢复；
4. 选定单一版本事实源。推荐保留 CMake `project(VERSION ...)`，release 验证脚本解析它；如果引入 `VERSION` 文件，则 CMake 必须读取它，不能两处手改；
5. 审计远端 tags 与 changelog。缺失的历史 tags 不应由实现模型擅自补推；提交一份差异报告给负责人决定。

验收条件：

```bash
! rg -n 'github\.com/LessUp/fastq-tools|LessUp/fastq-tools' \
  README.md CMakeLists.txt cmake build-config scripts .github
```

历史 changelog 中描述旧迁移的链接可保留，但需人工确认语境。

## 7. 明确不纳入本轮

- `issues/001` 的 JSON 输出是产品功能，不是当前发布阻断项；
- `issues/002` 的 macOS 支持应在真实 macOS CI 可用时单独实施；
- 不以脚本总行数为理由重写 `scripts/`；现有 CI 已证明许多脚本行为，未来只在重复故障有证据时收敛；
- 不把性能 benchmark 结果从 WSL2 外推到其他硬件；
- 不改变 FastQTools 的公共 API 或流水线架构。

## 8. 推荐顺序

1. `FQT-META-001` 的链接和真相修正；
2. `FQT-CPU-001`；
3. `FQT-TOOLCHAIN-001`；
4. `FQT-REL-001`；
5. 发布后再恢复/更新 release badge 和安装说明。
