# Tasks: correct-indexed-v2-spec

## 1. Baseline

- [ ] 1.1 阅读根 `AGENTS.md`，记录 HEAD、`git status --short` 和审计 base 差异
- [ ] 1.2 运行 `cargo fmt --all -- --check` 与最小 archive tests，记录现有结果
- [ ] 1.3 从 `src/archive/format.rs`、writer、reader 建立实际字段/identifier 对照表

## 2. Characterization and fixture

- [ ] 2.1 添加 codec/checksum/version 精确 bytes 的结构测试
- [ ] 2.2 用固定最小 FASTQ 生成 indexed v2 archive，提交 input/archive/manifest
- [ ] 2.3 添加 frozen decoder round-trip、hash、未知 identifier 和 unsupported major 测试

## 3. Documentation

- [ ] 3.1 修正 format spec 的 codec/checksum 表、版本兼容和 v1 声明
- [ ] 3.2 建立/更新 `openspec/project.md`，登记 `fqc-indexed/v2` 与 `FQC-DEC-001`
- [ ] 3.3 将根 `AGENTS.md` 的 blanket prohibition 收窄到 lightweight high-risk change policy，不引入 CLI/Node/tool-specific skills
- [ ] 3.4 更新直接引用错误编号的文档和 CHANGELOG；保留旧系统曾被移除的历史，不修改产品名/后缀

## 4. Verification

- [ ] 4.1 `cargo fmt --all -- --check`
- [ ] 4.2 `cargo clippy --all-targets -- -D warnings`
- [ ] 4.3 `cargo test --lib --tests`
- [ ] 4.4 `cargo doc --no-deps`
- [ ] 4.5 `git diff --check`、scope 审计并填写 `verification.md`
