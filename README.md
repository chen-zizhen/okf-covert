[🇨🇳 中文版](./README.zh-CN.md)

# OKF Convert

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

## License

Apache 2.0
