# OKF Convert

<details>
<summary>🇨🇳 中文</summary>

## 这是什么

一个 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) Skill，能把你手头的文档、笔记、知识库片段自动转换成符合 [Google OKF v0.1](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md) 规范的 Knowledge Bundle。

OKF（Open Knowledge Format）是 Google Cloud 于 2026 年 6 月发布的开放知识格式，用 Markdown + YAML frontmatter 组织知识，让人类和 AI Agent 都能直接读写。

## 支持的输入

- Markdown / 纯文本文件
- 整个目录（递归扫描）
- Obsidian Vault（自动转换 `[[wikilink]]`、callout、Dataview 等语法）
- PDF 文件（通过 pymupdf4llm MCP 工具转换）
- 直接粘贴的文本内容

## 使用方法

### 1. 安装 Skill

将本仓库克隆到你的 Claude Code Skills 目录：

```bash
git clone https://github.com/chen-zizhen/okf-covert.git ~/.claude/skills/okf-convert
```

或拷贝到项目级 Skills 目录：

```bash
git clone https://github.com/chen-zizhen/okf-covert.git .claude/skills/okf-convert
```

### 2. 触发转换

在 Claude Code 中使用以下任一方式触发：

```
/okf-convert
把这些文档转成 OKF 格式
帮我生成一个知识包
```

### 3. 确认结构

Skill 会先分析你的资料，展示拟定的目录结构和类型分配方案，确认后再生成。

## 输出结构

```
your-bundle/
├── index.md              # 根目录索引（含 okf_version: "0.1"）
├── log.md                # 创建/更新日志
├── tables/
│   ├── index.md
│   └── orders.md         # 概念文件（带 YAML frontmatter）
├── metrics/
│   ├── index.md
│   └── weekly_active_users.md
└── ...
```

每个概念文件自动生成规范的 frontmatter（`type` 必填，`title`、`description`、`tags` 等可选），概念之间通过 Markdown 链接形成知识图谱。

## 合规自检

生成完成后自动按 OKF v0.1 Section 9 的三条 conformance 条件检查：

- 每个概念文件都有可解析的 YAML frontmatter
- 每个 frontmatter 都包含非空的 `type` 字段
- 保留文件名（`index.md`、`log.md`）遵循格式要求

## 参考资料

本仓库 `references/` 目录包含：

- `OKF-SPEC.md` — OKF v0.1 完整规范原文
- `sample-bundle-ga4/` — Google 官方 GA4 示例 Bundle

## 相关链接

- [OKF 规范](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md)
- [Google Cloud 官方博客](https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing)
- [Karpathy 的 LLM Wiki Gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)

</details>

<details open>
<summary>🇬🇧 English</summary>

## What is this

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) Skill that automatically converts your documents, notes, and knowledge base fragments into a [Google OKF v0.1](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md)-compliant Knowledge Bundle.

OKF (Open Knowledge Format) is an open knowledge format released by Google Cloud in June 2026. It uses Markdown + YAML frontmatter to organize knowledge so that both humans and AI Agents can read and write it directly.

## Supported Inputs

- Markdown / plain text files
- Directories (recursive scan)
- Obsidian Vaults (auto-converts `[[wikilinks]]`, callouts, Dataview syntax, etc.)
- PDF files (via pymupdf4llm MCP tool)
- Pasted text content

## Usage

### 1. Install the Skill

Clone this repo into your Claude Code Skills directory:

```bash
git clone https://github.com/chen-zizhen/okf-covert.git ~/.claude/skills/okf-convert
```

Or into a project-level Skills directory:

```bash
git clone https://github.com/chen-zizhen/okf-covert.git .claude/skills/okf-convert
```

### 2. Trigger Conversion

Use any of the following in Claude Code:

```
/okf-convert
Convert these docs to OKF format
Generate a knowledge bundle for me
```

### 3. Confirm Structure

The Skill analyzes your materials first, presents the proposed directory structure and type assignments, and only generates after your confirmation.

## Output Structure

```
your-bundle/
├── index.md              # Root index (with okf_version: "0.1")
├── log.md                # Creation/update log
├── tables/
│   ├── index.md
│   └── orders.md         # Concept file (with YAML frontmatter)
├── metrics/
│   ├── index.md
│   └── weekly_active_users.md
└── ...
```

Each concept file gets spec-compliant frontmatter (`type` required; `title`, `description`, `tags` optional). Cross-references between concepts form a knowledge graph via standard Markdown links.

## Conformance Check

After generation, the Skill automatically validates against OKF v0.1 Section 9's three conformance conditions:

- Every concept file has parseable YAML frontmatter
- Every frontmatter contains a non-empty `type` field
- Reserved filenames (`index.md`, `log.md`) follow their format requirements

## References

The `references/` directory in this repo contains:

- `OKF-SPEC.md` — Full OKF v0.1 specification
- `sample-bundle-ga4/` — Google's official GA4 sample Bundle

## Links

- [OKF Specification](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md)
- [Google Cloud Blog](https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing)
- [Karpathy's LLM Wiki Gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)

</details>

## License

Apache 2.0
