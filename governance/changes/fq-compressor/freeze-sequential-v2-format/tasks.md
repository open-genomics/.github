# Tasks: freeze-sequential-v2-format

## 1. Baseline and specification

- [ ] 1.1 记录 HEAD/status，运行当前 format/round-trip baseline tests
- [ ] 1.2 从 archive reader/writer 提取精确 layout、endian、checksum 和 rejection 表
- [ ] 1.3 建立 `openspec/project.md`，记录 `FQC-DEC-001` 和 `fqc-sequential/v2`

## 2. Fixtures and tests

- [ ] 2.1 用可信 commit/version 生成 SE/PE/profiled fixtures、inputs 和 manifest
- [ ] 2.2 添加 frozen decoder compatibility 和 metadata tests
- [ ] 2.3 添加 magic/header/frame/footer truncation 和 checksum corruption matrix
- [ ] 2.4 添加 unsupported version/flags/profile/codec/reserved values tests
- [ ] 2.5 明确 writer stable fields 与非 canonical payload，不写脆弱的全文件 writer hash test

## 3. Documentation

- [ ] 3.1 写入 normative sequential v2 spec，更新直接冲突的 format/architecture docs
- [ ] 3.2 README 链接规范并把该实现标识为 `fqc-sequential/v2`
- [ ] 3.3 更新 CHANGELOG；不声称跨 Rust 格式兼容

## 4. Verification

- [ ] 4.1 `./scripts/lint.sh format-check`
- [ ] 4.2 `./scripts/build.sh clang-debug` 与 `./scripts/test.sh clang-debug`
- [ ] 4.3 fixture matrix 和 hashes 全部通过
- [ ] 4.4 运行 `./scripts/test.sh clang-asan`；若命中已记录环境限制，保存证据且不得声称 sanitizer 已通过
- [ ] 4.5 `git diff --check`、scope 审计并填写 `verification.md`
