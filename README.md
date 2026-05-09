# Citation Check Skill

Claude Code skill for verifying whether academic citations in AI-generated text are real or fabricated. Extracts references, cross-checks them against academic databases, and flags ghost citations.

## Background

LLMs are known to hallucinate plausible-looking citations — inventing paper titles, authors, and even DOIs that don't exist. This skill automates the verification process so you don't have to manually check every reference.

## Installation

```bash
# Clone the repo
git clone git@github.com:haiyichen001/citation-check-skill.git

# Copy the skill to your Claude Code skills directory
cp -r citation-check-skill/.claude/skills/citation-check ~/.claude/skills/
```

Or install via Smithery (coming soon).

## Usage

Invoke the skill in Claude Code:

```
/citation-check
```

Then paste the text containing citations you want to verify. The skill will:

1. Extract all citations from the text
2. Search academic databases (Semantic Scholar, arXiv, PubMed, Google Scholar)
3. Cross-check titles, authors, years, and venues
4. Output a structured verification report

You can also pass text directly:

```
/citation-check The following paper claims that "Smith et al. (2023) demonstrated quantum supremacy using 1000 qubits in Nature Physics..."
```

## Verification Status Legend

| Status | Meaning |
|--------|---------|
| ✅ Verified | Paper exists, key metadata matches |
| ⚠️ Mismatch | Paper exists, but author/year/venue differs |
| ❌ Fabricated | No matching paper found — likely hallucinated |
| 🔍 Unverifiable | Insufficient info to verify |

## Requirements

Requires Claude Code with MCP servers configured:
- `scholar` (Semantic Scholar)
- `arxiv` (arXiv API)
- `paper-search` (multi-source paper search)

## License

MIT
