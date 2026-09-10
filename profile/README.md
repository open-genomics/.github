<div align="center">

# Open Genomics

**面向生物信息学的开源工具链与中文知识库**

从 FASTQ 压缩、序列比对到变异检测，以及把它们讲清楚的知识库。

</div>

---

## 工具链

### 序列压缩

| 项目 | 语言 | 说明 |
|:-----|:-----|:-----|
| [fq-compressor](https://github.com/open-genomics/fq-compressor) | C++23 | FASTQ 专用压缩归档：列式分离 + 2-bit 碱基打包 + Zstd，三层 XXH64 校验，内存有界 |
| [fq-compressor-rust](https://github.com/open-genomics/fq-compressor-rust) | Rust | 围绕块索引 `.fqc` 归档格式的独立实现，支持随机访问 |

> ⚠️ **两个 `fqc` 的格式族不同。** 它们共享 `fqc` 命令名与 `.fqc` 后缀，但归档格式族互不兼容，**不能互相解码**。跨实现使用前请先确认格式族，详见各仓库 README。
> 两者的许可证也不同，详见各自 `LICENSE`。

### 序列处理与比对

| 项目 | 语言 | 说明 |
|:-----|:-----|:-----|
| [fastq-tools](https://github.com/open-genomics/fastq-tools) | C++23 | 高性能 FASTQ 质控工具（统计 / 过滤）：零拷贝 I/O、TBB 流水线 |
| [minibwa-rust](https://github.com/open-genomics/minibwa-rust) | Rust | 内存安全的 BWA-MEM 风格单端 DNA 短读比对器，以 C 参考实现为行为标准 |
| [bwa-rust](https://github.com/open-genomics/bwa-rust) | Rust | BWA-MEM 风格单端短读比对：FM-index、SMEM 种子、chaining、Smith-Waterman、SAM 输出 |

### 分析流程

| 项目 | 语言 | 说明 |
|:-----|:-----|:-----|
| [micos-2024](https://github.com/open-genomics/micos-2024) | Python | 2024「猛犸杯」参赛作品 · 宏基因组学端到端分析流程 |

## 知识库

| 项目 | 说明 |
|:-----|:-----|
| [wiki-bioinfo](https://github.com/open-genomics/wiki-bioinfo) | 面向中文社区的生物信息学体系化知识库：连接生物学对象、计算模型、核心算法、分析流程与数据资源 |
| [awesome-bioinfo-algorithms](https://github.com/open-genomics/awesome-bioinfo-algorithms) | 精选生物信息学算法知识库，含复杂度分析、CLI 维护工具与双语文档 |

## 参与贡献

- 各仓库的 `CONTRIBUTING.md` 描述该项目的具体流程；未单独定义的，以组织的
  [贡献指南](https://github.com/open-genomics/.github/blob/main/CONTRIBUTING.md) 为准
- 安全问题请走各仓库的**私有漏洞上报**通道，详见
  [安全策略](https://github.com/open-genomics/.github/blob/main/SECURITY.md)
- 所有仓库遵循[行为准则](https://github.com/open-genomics/.github/blob/main/CODE_OF_CONDUCT.md)
- 需要帮助或不确定该开在哪，见[支持指南](https://github.com/open-genomics/.github/blob/main/SUPPORT.md)
