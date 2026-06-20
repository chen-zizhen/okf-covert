[🇬🇧 English](./README.md)

# OKF Convert

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

## 许可证

Apache 2.0
