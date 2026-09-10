# Tasks: correct-parity-contract-docs

## 1. Baseline

- [ ] 1.1 记录 HEAD/status，运行 `cargo test --all-targets --all-features` 并确认 live parity 是否执行
- [ ] 1.2 核对 `tests/c_parity.rs` 的全部 skip 条件和 CI 提供的依赖
- [ ] 1.3 逐条核对 `scripts/compare.sh` 使用的命令、选项和产物是否存在

## 2. Truthful contract

- [ ] 2.1 建立 `openspec/project.md` 与 reference-parity change
- [ ] 2.2 更新 README/AGENTS/里程碑，区分 Rust tests、live parity、fixtures 和 release gate
- [ ] 2.3 让 skip 原因可观察且测试不把它表述为 comparison pass
- [ ] 2.4 仅在当前 C 接口可以验证时修复 compare script；否则记录 Blocked/unsupported
- [ ] 2.5 更新 CHANGELOG，不宣称尚未实现的 CI 强制 parity

## 3. Verification

- [ ] 3.1 `cargo fmt --all -- --check`
- [ ] 3.2 `cargo clippy --all-targets --all-features -- -D warnings`
- [ ] 3.3 `cargo test --all-targets --all-features`
- [ ] 3.4 分别记录“有/无参考依赖”的可观察行为；无法运行有依赖 case 时保持未验证
- [ ] 3.5 `git diff --check`、scope 审计并填写 `verification.md`
