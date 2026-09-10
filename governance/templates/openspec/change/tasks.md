# Tasks: <change-id>

任务只能在对应行为和验证实际完成后勾选。

## 1. Baseline and tests

- [ ] 1.1 记录 `git status --short`、HEAD 和 change 的 base commit 差异
- [ ] 1.2 运行并记录与本 change 相关的最小基线命令：`<command>`
- [ ] 1.3 添加能证明缺失行为的测试或可信 fixture：`<scenario names>`
- [ ] 1.4 确认新增测试在实现前按预期失败，或说明为何属于 characterization test

## 2. Implementation

- [ ] 2.1 `<最小内聚实现步骤>`；验证：`<command>`
- [ ] 2.2 `<错误/边界行为步骤>`；验证：`<command>`
- [ ] 2.3 更新用户文档、格式/schema 文档和 changelog；验证：`<command/search>`

## 3. Verification

- [ ] 3.1 运行所有 change 专项测试：`<commands>`
- [ ] 3.2 运行仓库标准 format/lint/build/test/doc 门禁：`<commands>`
- [ ] 3.3 逐条核对 delta spec scenarios 并填写 `verification.md`
- [ ] 3.4 运行 `git diff --check`，审查 diff 未超出 allowed surface
- [ ] 3.5 记录未运行项、剩余风险和最终 `git status --short`

## 4. Archive readiness

- [ ] 4.1 Reviewer 确认 `verification.md` 的 `Ready to archive: yes`
- [ ] 4.2 将 delta 同步到主规格，并确认主规格与实现一致
- [ ] 4.3 按日期归档 change，再运行规格校验和最小 smoke
