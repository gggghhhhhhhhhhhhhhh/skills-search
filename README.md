# GitHub Skills Search

A Codex Skill that discovers and ranks trending AI agent Skills projects on GitHub.

## Features

- Search Skills by domain (bioinformatics, single-cell, medical, NLP, etc.)
- Multi-query deduplication via GitHub Search API
- Top 30 ranking by Stars with ⭐/day growth metrics
- 🔥 Rising Stars detection (recent projects with above-median growth)
- Research-relevance filtering (loose/strict mode)
- Stars quality gate (>= 10)
- Session-level cache
- User feedback loop for result refinement
- Sort toggle: Stars / Stars/day / Last update / Newest

## Installation

```bash
# Clone to Codex skills directory
git clone https://github.com/gggghhhhhhhhhhhhhhh/github-skills-search.git "$env:USERPROFILE\.codex\skills\github-skills-search"
```

## Usage

In conversation:

> **"使用 $github-skills-search 搜索 single-cell 领域的 Skills"**
> **"Use $github-skills-search to find bioinformatics related Skills projects"**

### Workflow

1. You type a research domain (or skip for broad search)
2. Skill searches GitHub, deduplicates, filters, and ranks
3. Returns: Top 30 table + Rising Stars spotlight
4. You can toggle sort order or mark irrelevant results

## License

MIT
