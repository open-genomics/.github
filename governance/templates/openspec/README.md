# 仓库内 OpenSpec 模板

这些模板供 Propose 模型复制并按目标仓库事实填写。不要把尖括号占位值直接提交；不适用的章节写明“不适用及原因”，不要留空。

目标结构：

```text
openspec/
  project.md                       <- project.md
  AGENTS.md                        <- AGENTS.md
  specs/
  changes/
    <change-id>/
      proposal.md                  <- change/proposal.md
      design.md                    <- change/design.md
      tasks.md                     <- change/tasks.md
      verification.md             <- change/verification.md
      specs/<capability>/spec.md   <- change/spec.md
    archive/
```

首次引入尚无主规格的 capability 时，不要提前在 `openspec/specs/` 写未来行为；用 change spec 的 `## ADDED Requirements` 描述目标，完成验证后再归档成为主规格。

流程详见 [仓库本地工作流](../../10-repository-local-openspec-workflow.md)。
