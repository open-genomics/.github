# Tasks: declare-python-production-orchestrator

## 1. Baseline

- [ ] 1.1 记录 HEAD/status，核对 Python full-run、shell wrapper 和 WDL 数量/入口
- [ ] 1.2 全仓搜索 `WDL|workflow|resume|断点|skip|full-run|production` 并保存需修改清单
- [ ] 1.3 运行现有快速 Python test 基线：`pytest -m "not slow" tests/`

## 2. Specification and documentation

- [ ] 2.1 建立 `openspec/project.md`，记录 `MICOS-ORCH-001`
- [ ] 2.2 更新 README 和 docs 的单一生产入口、WDL 实验性及 resume 不支持声明
- [ ] 2.3 对齐 CLI help、shell/WDL 说明和 `AGENTS.md` 中冲突的当前事实
- [ ] 2.4 添加可维护的文档一致性检查或明确 verification 搜索命令

## 3. Verification

- [ ] 3.1 `black --check --diff micos scripts`
- [ ] 3.2 `flake8 micos scripts`
- [ ] 3.3 `mypy micos --ignore-missing-imports`
- [ ] 3.4 `pytest tests/ -v`（若环境不具备依赖，报告并保持未通过，不伪造）
- [ ] 3.5 重跑能力关键词审计、`git diff --check`、scope 审计并填写 `verification.md`
