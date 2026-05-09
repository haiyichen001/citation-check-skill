# Citation Check Skill

Claude Code skill for verifying whether academic citations in AI-generated text are real or fabricated. Supports PDF, DOCX, TXT, Markdown, and LaTeX.

Checks three things per citation: **existence**, **metadata accuracy**, and **content support** — whether the cited paper actually says what the author claims it says.

## Token Consumption Warning

```
+====================================================================+
|                                                                      |
|   每条引用验证消耗 20,000 ~ 50,000 tokens                             |
|                                                                      |
|   一篇 30 条引用的论文 ≈ 600,000 ~ 1,500,000 tokens                   |
|                                                                      |
|   如果你用 Claude API 按量付费，请做好心理准备。                       |
|                                                                      |
|   但话说回来——                                                       |
|                                                                      |
|   你愿意花 tokens 查引用，说明你认真。                                |
|   那个用 AI 瞎编参考文献的人，连 tokens 都不愿意花。                  |
|                                                                      |
|   谁更值得尊重？不言而喻。                                           |
|                                                                      |
|   学术诚信无价，查出来一条造假你就回本了。                            |
|                                                                      |
+====================================================================+
```

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

## Supported Input Formats

| Format | Method |
|--------|--------|
| PDF | pdf-reader MCP |
| DOCX | python-docx via Bash |
| TXT / Markdown | Read tool |
| LaTeX (.tex) | Read tool + `\cite{}` / `\bibitem{}` parsing |

## License

MIT
