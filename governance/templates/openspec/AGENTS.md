# OpenSpec 工作约束

仓库根 `AGENTS.md` 始终适用，且优先于本文件。本文件只补充规格工作流。

- `openspec/specs/` 只能描述已经实现并验证的当前行为。
- 未来行为只能写入 `openspec/changes/<change-id>/`。
- 一次只处理一个 change；不得顺手实现其他 change。
- Propose 阶段只能修改 OpenSpec artifacts，不得修改产品代码。
- Apply 需要 change 状态为 Approved，并且用户已明确授权修改代码。
- requirement 写可观察行为；实现细节写 design/tasks。
- 任务必须有验证证据，不能因缺依赖静默 skip。
- Verify 未通过不得 archive；archive 前必须同步主规格。
- 不自动 commit、push、开 PR、打 tag 或发布，除非用户明确授权相应动作。
- 跨仓库任务必须在每个仓库建立独立 change 和独立 diff。
