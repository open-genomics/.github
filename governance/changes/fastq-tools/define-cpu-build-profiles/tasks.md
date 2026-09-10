# Tasks: define-cpu-build-profiles

## 1. Baseline and tests

- [ ] 1.1 记录 HEAD/status，检查 `issues/` 和所有 flag 注入点
- [ ] 1.2 运行相关 baseline build/test，保存 default Release compile commands
- [ ] 1.3 添加 default/v3/native/invalid/unsupported 和 legacy mapping/conflict 的 CMake 或 wrapper tests

## 2. Implementation

- [ ] 2.1 建立严格的 `FQTOOLS_CPU_BASELINE` cache string，默认 portable
- [ ] 2.2 把 CMake、presets 和 build wrapper 映射到该 option，删除隐式 Release v3
- [ ] 2.3 在 configure summary/artifact metadata 暴露 profile
- [ ] 2.4 CI 至少验证 portable 和 v3 配置/构建；native 不生成发布 artifact
- [ ] 2.5 更新 README 和 scripts docs，不宣称 runtime dispatch
- [ ] 2.6 为 `ENABLE_NATIVE_ARCH` 实现单一迁移窗口和冲突失败，不长期保留两套 flags 逻辑

## 3. Verification

- [ ] 3.1 `./scripts/core/lint check`
- [ ] 3.2 `./scripts/core/build --dev` 与相关 configuration tests
- [ ] 3.3 `./scripts/core/test`
- [ ] 3.4 检查三 profile 的最终 compile commands 和 invalid failure
- [ ] 3.5 `git diff --check`、scope 审计并填写 `verification.md`
