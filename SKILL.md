---
name: okf-convert
description: 将用户提供的资料（文档、笔记、知识库片段）转换为符合 Google OKF v0.1 规范的 Knowledge Bundle
allowed-tools: Read, Glob, Grep, Write, Edit, Bash, Agent
---

## 什么是 OKF

Open Knowledge Format（OKF）是 Google Cloud 于 2026 年 6 月发布的开放知识格式规范（v0.1），用纯 Markdown + YAML frontmatter 表示知识，让人类和 AI Agent 都能读写。

## 参考资料

执行前先按需加载以下 reference，不要凭记忆猜测规范细节：

- `references/OKF-SPEC.md` — OKF v0.1 完整规范原文，合规判断以此为准
- `references/sample-bundle-ga4/` — Google 官方 GA4 示例 Bundle，目录结构和文件格式的参照样本

具体使用方式：

- 生成 Concept 文件前，先读 `references/sample-bundle-ga4/tables/events_.md` 了解一个典型 Concept 文件长什么样
- 生成 index.md 前，先读 `references/sample-bundle-ga4/index.md` 了解根目录索引的格式
- 合规自检时，以 `references/OKF-SPEC.md` Section 9 的三条 conformance 条件为判定标准
- 遇到不确定的格式问题时，以样本中的实际写法为准

## 触发条件

用户要求将资料「转成 OKF」「生成知识包」「按 OKF 格式整理」或提及 /okf-convert 时触发。

## 输入

用户可能提供以下任意一种输入：

- 一个或多个 Markdown 或文本文件路径
- 一个目录路径（递归扫描其中的内容文件）
- 直接粘贴的文本内容
- 一个 Obsidian vault 目录
- PDF 文件（使用 `mcp__pymupdf4llm__convert_pdf_to_markdown` 工具转换后再处理）

如果用户没有指定输出目录，默认输出到源文件同级的 `okf-bundle/` 目录下。

## 执行流程

### 1. 扫描与分析

读取用户提供的资料，对每份内容判断：

- 它描述的是什么「概念」（一张表、一个指标、一篇教程、一个 API、一条规则、一个流程……）
- 合理的 type 值是什么（参见下方 type 决策参考）
- 它和其他内容之间有没有关联关系

如果输入是 Obsidian vault，识别 `[[wikilink]]` 并记录链接关系。

如果输入包含 PDF 文件，先用 `mcp__pymupdf4llm__convert_pdf_to_markdown` 转换为 Markdown，转换失败则跳过并在最终报告中标记。

如果单个源文件超过 5000 字，评估是否应拆分为多个独立概念。拆分依据是内容是否涵盖了多个可独立描述的主题。如果只是单个主题的详细展开，保持为一个文件。

#### type 决策参考

根据内容特征选择 type 值：

| 内容特征 | 推荐 type 值 |
|---------|-------------|
| 描述数据库表、视图的结构和字段 | `Database Table` 或 `BigQuery Table` |
| 描述数据集、数据库整体 | `Dataset` |
| 定义业务指标、KPI、计算公式 | `Metric` |
| 操作步骤、运维手册、应急流程 | `Playbook` |
| 教程、使用指南、How-to | `Guide` |
| API 接口、端点说明 | `API Endpoint` |
| 架构设计、系统说明 | `Architecture` |
| 业务规则、策略、约束条件 | `Policy` |
| 概念解释、术语定义 | `Concept` |
| 无法归入以上类别 | `Reference`（兜底） |

type 值没有中央注册表，上表仅供参考。选择描述性强、自解释的短语即可，消费端必须容忍未知类型。

### 2. 确定 Bundle 结构

根据概念的类型和数量，设计目录结构。遵循以下原则：

- 同类概念放在同一个子目录下（如 `tables/`、`metrics/`、`guides/`）
- 目录名用小写英文，简短有意义
- 文件名用小写英文 + 连字符，如 `weekly-active-users.md`
- 如果概念少于 5 个，可以全部放在根目录，不必强行分子目录
- 如果用户源文件中有名为 `index.md` 或 `log.md` 的文件，将其内容并入对应概念文件并重命名（如 `overview.md`），避免与 OKF 保留文件名冲突

在动手生成之前，先向用户展示拟定的目录结构和 type 分配方案，等待确认。

如果用户要求调整，按用户意见修改后重新展示，直到确认。

### 3. 生成 Concept 文件

每个概念生成一个 `.md` 文件，格式严格遵循 OKF v0.1 规范：

```markdown
---
type: <概念类型，必填>
title: <显示名称>
description: <一句话摘要>
resource: <原始资源 URI，如有>
tags: [<标签1>, <标签2>]
timestamp: <ISO 8601 格式的最后修改时间>
---

<Markdown 正文>
```

#### frontmatter 规则

- `type` 是唯一必填字段
- `title` 如果省略，消费端可以从文件名推导
- `description` 写一句话，用于 index.md 和搜索摘要
- `resource` 只在概念对应一个具体外部资源时才填（如数据库表的控制台链接）
- `tags` 用于跨目录的分类检索
- `timestamp` 用 ISO 8601 格式，如 `2026-06-20T10:00:00Z`
- 可以添加任意额外字段，消费端会容忍未知字段
- 使用兜底类型 `Reference` 时，在最终报告中标记提醒用户后续细化

#### 正文规则

- 用结构化的 Markdown（标题、列表、表格、代码块），避免大段散文
- 概念之间的关联用标准 Markdown 链接表示，支持两种形式：
  - 绝对路径（推荐），以 `/` 开头，相对于 bundle 根目录，如 `[客户表](/tables/customers.md)`
  - 相对路径，如 `[客户表](../tables/customers.md)`
- 如有外部引用来源，在文件底部加 `# Citations` 章节，编号列出
- 以下章节标题有约定含义，适用时应使用：`# Schema`（字段/结构描述）、`# Examples`（用法示例）、`# Citations`（外部来源）

### 4. 生成 index.md

在 bundle 根目录和每个子目录下生成 `index.md`。

index.md 不带 frontmatter。唯一的例外是 bundle 根目录的 index.md，可以在 frontmatter 中声明 OKF 版本：

```yaml
---
okf_version: "0.1"
---
```

除此之外，所有 index.md 的 frontmatter 都不允许。

正文用标题分组，每个条目链接到对应文件并附一句话描述：

```markdown
# Tables

* [订单表](tables/orders.md) - 每一行代表一笔已完成的客户订单
* [客户表](tables/customers.md) - 客户基本信息

# Subdirectories

* [metrics](metrics/) - 业务指标定义
```

每个条目的描述从对应 concept 的 `description` 字段取。子目录条目可以链接到 `subdir/` 或 `subdir/index.md`，两种写法都合法。

### 5. 生成 log.md（可选）

在 bundle 根目录生成 `log.md`，记录本次创建。log.md 不带 frontmatter：

```markdown
# Bundle Update Log

## 2026-06-20
* **Creation**: 从用户提供的资料初始化 OKF bundle
* **Creation**: 新增 [概念名称](/路径/文件.md)
```

日期用 ISO 8601 `YYYY-MM-DD` 格式，最新的在最前面。

如果用户的输入包含版本信息或修改历史，一并记录。

### 6. 合规自检

生成完成后，按 OKF v0.1 Section 9 的三条 conformance 条件逐条检查：

1. [ ] 每个非保留的 `.md` 文件都有可解析的 YAML frontmatter
2. [ ] 每个 frontmatter 都包含非空的 `type` 字段
3. [ ] 所有保留文件名（`index.md` 和 `log.md`）在出现时遵循各自的格式要求（index.md 无 frontmatter，根目录例外；log.md 无 frontmatter，日期用 `YYYY-MM-DD`）

额外检查（非合规要求，但作为质量指标报告）：

- [ ] 交叉链接的目标文件在 bundle 内存在（断链不违反合规，但应列出警告）
- [ ] 使用了兜底类型 `Reference` 的文件列表（提醒用户后续细化）

自检结果以清单形式展示给用户。

## Obsidian Vault 特殊处理

如果输入是 Obsidian vault：

- `[[wikilink]]` 转换为标准 Markdown 链接 `[标题](./路径.md)`
- `[[wikilink|别名]]` 转换为 `[别名](./路径.md)`
- `![[embed]]` 嵌入语法转换为普通链接 `[embed](./路径.md)`，并在链接前加一行说明「参见」
- `> [!note]` / `> [!warning]` 等 callout 语法转换为标准 Markdown 引用块 `>`，callout 类型作为加粗前缀保留（如 `> **Note:** 内容`）
- Dataview 查询块（` ```dataview ` ）删除，这些是运行时动态内容，无法静态保留
- Obsidian 的 YAML frontmatter 保留，补充缺失的 `type` 字段
- 已有的 tags 字段保留
- 忽略 `.obsidian/` 配置目录
- 非 Markdown 文件（图片、附件等）忽略，但如果 Markdown 中引用了这些文件，保留链接原样

## 输出格式

最终向用户报告：

```
OKF Bundle 生成完成

输出目录：<路径>
概念文件：<N> 个
目录索引：<N> 个 index.md
交叉链接：<N> 条
合规状态：通过 / 有 <N> 项警告

警告（如有）：
- <断链列表>
- <使用兜底类型的文件列表>
- <PDF 转换失败的文件列表>

目录结构：
<树形展示>
```

## 注意事项

- 不编造用户资料中不存在的信息，只做格式转换和结构整理
- 如果用户的资料量很大（超过 20 个文件），先扫描并展示分类方案，确认后再批量生成
- 生成的 bundle 应该是自包含的，拷贝到任何地方都能独立使用
- 所有文件必须是 UTF-8 编码。如果源文件编码不是 UTF-8，转换时统一转为 UTF-8
