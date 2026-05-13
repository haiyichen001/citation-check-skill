> **Merged into [Reference Workbench](https://github.com/haiyichen001/reference-workbench-skill). This repository is no longer updated.**

# Citation Check Skill

Claude Code skill for verifying whether academic citations in AI-generated text are real or fabricated. Supports PDF, DOCX, TXT, Markdown, and LaTeX.

Checks three things per citation: **existence**, **metadata accuracy**, and **content support** — whether the cited paper actually says what the author claims it says.

## Token Warning

Each citation check burns ~20k-50k tokens (we launch 10 agents in parallel, each reading paper metadata and abstracts). A 30-citation paper can cost 600k-1.5M tokens total.

Yes, it's heavy. But whoever hallucinated those citations didn't even spend the tokens to look them up. You're already doing better than them.

## Installation

### Step 1 — Install the skill

```bash
git clone git@github.com:haiyichen001/citation-check-skill.git
cp -r citation-check-skill/.claude/skills/citation-check ~/.claude/skills/
```

### Step 2 — Install MCP dependencies

Three MCP servers are required. The skill will detect missing ones and refuse to run until all are installed.

```bash
npx smithery install @smithery-ai/arxiv
npx smithery install @smithery-ai/scholar
npx smithery install @smithery-ai/paper-search
```

| MCP | Backend | Login Required | API Key |
|-----|---------|---------------|---------|
| arxiv | arXiv public API | No | No |
| scholar | Semantic Scholar API | No | No |
| paper-search | PubMed / bioRxiv / Google Scholar | No | No |

Restart Claude Code after installation.

## Usage

```
/citation-check
```

Then provide a file path or paste text directly. The skill will:

1. Extract all citations + their in-text locations + surrounding context
2. Verify each citation in parallel (10 per batch)
3. Output a structured report

## Verification Status

| Status | Meaning |
|--------|---------|
| ✅ Verified | Paper exists, metadata correct, content supports the claim |
| ⚠️ Metadata Mismatch | Paper exists but author/year/venue differs |
| 🔍 Content Mismatch | Paper exists and metadata is correct, but the cited paper does not support the claim made at the citation location |
| ❌ Fabricated | Paper does not exist — likely AI hallucination |
| 🔍 Unverifiable | Insufficient data to verify (e.g. Chinese thesis not indexed) |

## Limitations

**Chinese papers (CNKI/Wanfang/VIP) are not yet supported.** There is currently no open API or MCP server for Chinese academic databases. Chinese-language citations may return "unverifiable" results. Support for Chinese papers is under development.

If you primarily work with Chinese-language literature, this skill is not ready for your use case yet.

## Supported Input Formats

| Format | Method |
|--------|--------|
| PDF | pdf-reader MCP |
| DOCX | python-docx via Bash |
| TXT / Markdown | Read tool |
| LaTeX (.tex) | Read tool + `\cite{}` / `\bibitem{}` parsing |

## License

MIT
