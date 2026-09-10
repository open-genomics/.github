# Tasks: version-binary-formats-v2

## 1. Baseline and legacy evidence

- [ ] 1.1 记录 HEAD/status，运行 `make lint`、`make test`
- [ ] 1.2 从 `v1.0.0` 可信构建生成四算法 v1 input/archive/manifest，不污染当前树
- [ ] 1.3 添加 legacy、unknown magic、truncated、CRC 和 boundary 的预期失败/characterization tests

## 2. V2 implementation

- [ ] 2.1 将四算法常量和 writer 切换为 `HFM2/AEN2/RCN2/RLE2`
- [ ] 2.2 实现 v2/known legacy/unknown/truncated 的确定分类
- [ ] 2.3 保持算法 body 除已审计修复外不变，并验证 checked size/arithmetic

## 3. V2 fixtures and documentation

- [ ] 3.1 生成 v2 empty/small fixtures 与 manifest，加入 decoder compatibility tests
- [ ] 3.2 把 README/docs 中上限、magic、CRC、历史和教学定位统一到精确契约
- [ ] 3.3 将项目版本与 CHANGELOG 收敛为 2.0.0；删除含糊的“格式未变化”声明
- [ ] 3.4 建立 `openspec/project.md` 和归档后 `binary-formats` 主规格

## 4. Verification

- [ ] 4.1 `make lint`
- [ ] 4.2 `make test`
- [ ] 4.3 fixture manifest/hashes 和四算法矩阵全部通过
- [ ] 4.4 搜索所有 legacy/current magic 和 4 GiB 旧声明，人工分类剩余命中
- [ ] 4.5 `git diff --check`、scope 审计并填写 `verification.md`
