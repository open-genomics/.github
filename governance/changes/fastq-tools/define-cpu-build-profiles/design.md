# Design: define-cpu-build-profiles

## Profiles

| Profile | Compiler flags | Default/release policy |
|---|---|---|
| `portable` | 不添加 `-march=x86-64-v3`、`-march=native` | 默认和正式通用 artifact |
| `x86-64-v3` | 显式 `-march=x86-64-v3`，仅适用支持的 x86-64 compiler | 优化 artifact/部署环境 |
| `native` | 显式 `-march=native` | 本机 benchmark，禁止发布 |

## Configuration model

使用单一 CMake cache string `FQTOOLS_CPU_BASELINE=portable|x86-64-v3|native`，allowed values 严格验证。build wrapper 和 presets 只映射到该 option，不另写第二套 flags。非 x86-64 平台选择 v3 必须在 configure 阶段清晰失败；portable/native 遵循编译器/平台能力。

现有 `ENABLE_NATIVE_ARCH` 是已存在的构建接口，保留一个迁移窗口：仅在新变量未显式设置时，ON 映射到 `native` 并警告弃用；它与显式非 native profile 冲突时 configure 失败。不能让两个变量长期分别修改 flags。

## Allowed surface

- `CMakeLists.txt` 和 CPU/compiler flag module
- `CMakePresets.json`
- `scripts/core/build` 及参数/帮助测试
- CMake/configuration/E2E tests
- `.github/workflows/ci.yml` 的 profile 验证（不改无关 jobs）
- README/scripts docs、`openspec/`

## Verification strategy

1. configure test 检查默认 compile commands 中没有 v3/native；
2. v3/native 显式路径包含对应 flag；
3. invalid profile configure fail；
4. legacy option 映射和冲突路径都有 configure test；
5. portable binary 在 baseline runner/container smoke；若当前环境无法模拟旧 CPU，至少做指令/flags 静态验证并把真正运行 smoke 留给 release change；
6. 所有现有 tests 保持通过。

## Risk

Conan host profile、CMake preset 和 wrapper 都可能注入 flags。实现必须检查最终 compile commands，而不只检查一个 CMake 变量。
