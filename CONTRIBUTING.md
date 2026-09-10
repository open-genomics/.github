# 贡献指南（组织默认）

本文件是 Open Genomics 组织的**默认贡献指南**。若目标仓库自带 `CONTRIBUTING.md`，
以仓库内的为准；仓库内没有的，按本文件执行。

## 开始之前

1. 读目标仓库的 `README.md`，确认构建方式与支持的平台
2. 读目标仓库的 `AGENTS.md`（若有），它记录了该仓库的约定、验证命令与已知限制
3. 大改动先开 issue 讨论，避免写完才发现方向不一致

## 报告问题

用仓库的 issue 模板提交。一个高质量的 bug 报告应包含：

- **复现命令**与最小输入（能切成小样本就切）
- **期望行为**与**实际行为**
- 版本 / 提交号、操作系统、编译器或运行时版本

安全问题**不要**走公开 issue，见 [SECURITY.md](SECURITY.md)。

## 提交改动

1. 从默认分支开分支，改动尽量小而完整 —— 一个 PR 做一件事
2. 本地跑通该仓库的门禁（CI 在 GitHub Actions 跑同一套，见 `.github/workflows/`）
3. 提交信息用 [Conventional Commits](https://www.conventionalcommits.org/)：
   `feat:` / `fix:` / `docs:` / `refactor:` / `test:` / `chore:` / `perf:`
   可带作用域，如 `fix(pipeline): ...`
4. 改了 CLI 行为、默认值或产物格式，同步更新对应文档

## 代码风格

各仓库自带格式化与静态检查配置，以其为准，不要引入与既有配置冲突的风格：

| 生态 | 配置 |
|:-----|:-----|
| C++ | `.clang-format`、`.clang-tidy`、`.editorconfig` |
| Rust | `rustfmt.toml`、`clippy.toml` |
| Python | `pyproject.toml`、`.flake8` |
| 通用 | `.editorconfig`、`.pre-commit-config.yaml` |

## 审查与合并

- 改动需要至少一次 review
- 审查关注正确性、契约兼容性与可维护性，不纠结个人风格偏好
- 破坏性变更可以接受，但必须在 PR 描述与 `CHANGELOG.md` 中明确写出影响范围

## 许可证

提交贡献即表示你同意以目标仓库的既有许可证发布该贡献。
提交前请确认你拥有所提交内容的权利，且未引入与仓库许可证不兼容的第三方代码。
