# open-genomics/.github

Open Genomics 组织的**组织级配置仓库**。这里的文件对该组织下所有仓库生效，
除非目标仓库自带同名文件。

> ⚠️ **本仓库必须保持 Public。** GitHub 只在 `.github` 仓库公开时，才会把其中的
> 默认社区健康文件应用到组织内的其他仓库。改为私有会让下面的所有兜底全部失效。

## 目录结构

```
.github/
├── profile/
│   └── README.md                 # 组织首页，显示在 github.com/open-genomics
├── .github/                      # 注意：套了两层 .github，这是 GitHub 的硬性要求
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.yml
│   │   ├── feature_request.yml
│   │   └── config.yml
│   └── PULL_REQUEST_TEMPLATE.md
├── workflow-templates/           # 组织内新建 workflow 时可选的起步模板
│   ├── ci-cpp.yml
│   ├── ci-rust.yml
│   ├── ci-python.yml
│   └── ci-node.yml
├── CODE_OF_CONDUCT.md            # 以下四份为组织级兜底
├── CONTRIBUTING.md
├── SECURITY.md
└── SUPPORT.md
```

### 为什么 `ISSUE_TEMPLATE` 外面还套一层 `.github`

GitHub 的规定不是对称的：

- **issue 模板及其 `config.yml`** 必须位于 `.github/ISSUE_TEMPLATE` 目录，
  没有例外。所以在本仓库里就是 `.github/.github/ISSUE_TEMPLATE/`。
- **其余社区健康文件**（`CONTRIBUTING.md`、`SECURITY.md`、`CODE_OF_CONDUCT.md`、
  `SUPPORT.md`）可以放在仓库根目录、`.github/` 或 `docs/` 下，本仓库选根目录。

这是 GitHub 官方文档与实际行为共同确认的结论，不是笔误。

## 生效顺序

目标仓库自带同名文件时，**仓库内的优先**，本仓库的兜底被忽略。
issue 模板是整体覆盖：只要目标仓库自己有 `.github/ISSUE_TEMPLATE/`，
本仓库的模板就完全不生效。

## 如何给新仓库接入

1. 优先让新仓库**自带** `CONTRIBUTING.md` 与 `SECURITY.md`，写清该项目特有的
   构建方式、验证命令与安全范围；本仓库的兜底是保底，不是首选
2. CI 从 `workflow-templates/` 里挑一个最接近的复制到新仓库的
   `.github/workflows/`，再按实际情况调整 —— 模板是起点，不是成品
3. 在新仓库启用 **Settings → Code security → Private vulnerability reporting**，
   否则 `SECURITY.md` 里指向的私有上报通道不可用

## 待办

- [ ] `CODE_OF_CONDUCT.md` 中的举报渠道目前指向 @LessUp 的 GitHub 私信，
      建议补一个专用邮箱后替换
- [ ] 补充本仓库的 `LICENSE`（社区健康文件的许可方式需维护者决定）
- [ ] 将根目录 `maintenance-design/` 的组织治理文档迁入本仓库，
      使其进入版本控制并可协作评审
