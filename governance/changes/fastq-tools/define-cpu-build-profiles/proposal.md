# Change Proposal: define-cpu-build-profiles

## Metadata

- Status: `Proposed`
- Repository: `open-genomics/fastq-tools`
- Audit base: `5bab799bb9beea272ee07857b31046060e29db53`
- Capability: `build-portability`
- Task IDs: `FQT-CPU-001`, `ORG-GOV-001`
- Decision IDs: none

## Why

Release 构建默认加入 `-march=x86-64-v3`，但项目把该路径称为 portable。生成的二进制会在部分基线 x86-64 CPU 上因缺少指令集无法运行，且用户没有清晰 profile 选择。

## Changes

**CPU build profile**

- From: Release 隐式等同 `x86-64-v3`，portable 声明与实际最低 CPU 不一致。
- To: `portable` 为默认；`x86-64-v3` 和 `native` 只能显式选择；构建/产物元数据可识别 profile。
- Reason: 让发布兼容范围可声明、可测试。
- Impact: 默认 Release 性能可能略降；显式 v3/native 保留优化能力；不改变 CLI 数据语义。

## Scope

- CMake CPU flags、presets 和现有 build wrapper 的 profile 参数；
- 配置测试/CI 覆盖和 README；
- 后续 release workflow 可消费 profile，但本 change 不创建 release。

## Out of scope

- 不实现运行时 SIMD dispatch；
- 不改变质量修剪算法；
- 不升级编译器/Conan/CMake；
- 不创建 macOS/Windows artifact；
- 不处理全部 release workflow 或旧组织链接。

## Compatibility and rollback

源码/API/CLI 兼容。依赖 undocumented implicit v3 的本地 benchmark 应改为显式 profile。回滚不能继续把 v3 artifact 标为 portable。

## Approval

- Apply approval: `pending explicit authorization`
