---
name: meta-builder
description: AI-powered research automation and synthesis engine
version: 1.4.2
author: Meta Builder Team
tags:
  - research
  - ai
  - automation
  - synthesis
  - analysis
maintainer: SMOUJBOT
dependencies:
  - python>=3.10
  - openai>=4.0.0
  - beautifulsoup4>=4.12.0
  - requests>=2.31.0
  - pandas>=2.0.0
  - numpy>=1.24.0
  - arxiv>=1.4.0
  - scholarly>=1.7.0
  - pymupdf>=1.23.0
  - google-search-results>=2.4.0
  - tiktoken>=0.5.0
system_requirements:
  memory_min: "4GB"
  memory_recommended: "8GB"
  disk_space: "500MB"
  network: "Required for web searches and API calls"
compatibility:
  min_openclaw_version: "2.1.0"
  max_openclaw_version: "3.0.0"
---

# Meta Builder

AI-powered research automation and synthesis engine for academic, technical, and market research workflows.

## Purpose

Meta Builder automates comprehensive research tasks including:
- Academic literature reviews and paper analysis (arXiv, PubMed, Semantic Scholar)
- Competitive intelligence gathering and market analysis
- Technical deep-dives on frameworks/libraries with code example extraction
- Regulatory document analysis and compliance research
- Multi-source information synthesis into coherent briefs
- Automated citation network mapping
- Patent landscape analysis
- News sentiment tracking across time periods

Real workflows:
- "Analyze the last 2 years of transformer architecture improvements and create a timeline"
- "Compare React 18 vs Vue 3 performance benchmarks from 10 independent sources"
- "Research GDPR compliance requirements for SaaS startups in EU markets"
- "Gather all security vulnerabilities for log4j from NVD, GitHub advisories, and security blogs"

## Scope

### Primary Commands

```
meta-builder research <query> [--sources=<list>] [--depth=<n>] [--output=<format>]
meta-builder analyze <url_or_file> [--extract=<type>] [--summarize] [--cite]
meta-builder synthesize <input_pattern> [--thematic] [--timeline] [--contradictions]
meta-builder cite <query> [--style=<citation_style>] [--max_results=<n>]
meta-builder validate <source_url> [--check_facts] [--cross_reference=<n>]
meta-builder export <research_id> [--format=<json|markdown|pdf|bibtex>]
meta-builder monitor <query> [--frequency=<cron>] [--alert_threshold=<float>]
meta-builder network <paper_url> [--depth=<n>] [--include_citations=true]
```

### Source Modules

Meta Builder integrates with these research sources (enable via `--sources`):
- `arxiv`: Preprint papers (default enabled)
- `semantic_scholar`: Academic citations and influential papers
- `pubmed`: Biomedical literature
- `patents`: USPTO and international patents
- `github`: Repository issues, PRs, README analysis
- `nvd`: National Vulnerability Database
- `security_advisories`: GitHub Security Advisory Database
- `news`: News APIs (requires NEWS_API_KEY)
- `reddit`: Subreddit content analysis
- `stackexchange`: Q&A sites for technical research
- `legal`: Regulations and compliance documents (requires specialized APIs)
- `custom`: User-provided URLs or local files

## Work Process

### Step 1: Research Planning Phase
Command: `meta-builder research <query> --plan_only`
- Parses query into sub-questions using LLM
- Identifies required source types (academic, news, code, patents)
- Estimates token usage and cost
- Generates research plan for approval
- Creates `research_<id>.plan.json` manifest

Example output:
```json
{
  "research_id": "req_20240315_001",
  "query": "transformer architecture improvements 2022-2024",
  "sub_queries": [
    "efficient attention mechanisms",
    "long context transformers",
    "multimodal transformers",
    "training efficiency improvements"
  ],
  "sources": ["arxiv", "semantic_scholar", "github"],
  "estimated_tokens": 150000,
  "estimated_cost": 3.75
}
```

### Step 2: Data Collection Phase
Command: `meta-builder research <query> --execute`
- Sequential source fetching with rate limiting
- Parallel downloads from same source (max 5 concurrent)
- Resume capability (stores checkpoint in `~/.openclaw/meta-builder/checkpoints/`)
- Raw data stored in `~/.openclaw/meta-builder/data/<research_id>/raw/`

Rate limits per source (configurable):
- arXiv: 1 request/3 seconds
- Semantic Scholar: 1 request/second
- GitHub: 5000 requests/hour (uses token if GITHUB_TOKEN set)
- News APIs: provider-specific

### Step 3: Extraction & Processing
Command: `meta-builder process <research_id> [--extract=<pdf|html|code|json>]`
- PDF text extraction using PyMuPDF
- HTML parsing (BeautifulSoup4) with content isolation
- Code block extraction and language detection
- Table extraction to CSV/JSON
- Metadata extraction (authors, dates, citations)
- Deduplication across sources (cosine similarity >0.95)

Example extraction:
```bash
meta-builder process req_20240315_001 --extract=pdf --output=structured
# Outputs: processed/<research_id>/extracted_text.json
# with structure:
{
  "source": "arxiv.org/abs/2301.00001",
  "title": "Efficient Long-Context Transformers",
  "authors": ["Jane Doe", "John Smith"],
  "abstract": "...",
  "sections": [
    {"heading": "Introduction", "content": "..."},
    {"heading": "Method", "content": "...", "code_blocks": [...]}
  ],
  "references": [...],
  "embedding": [0.023, -0.045, ...]  # 1536-dim OpenAI embedding
}
```

### Step 4: Synthesis Phase
Command: `meta-builder synthesize <research_id> [--method=<thematic|chronological|contradiction>]`
**Thematic synthesis** (default):
- Clusters documents by embedding similarity
- Identifies consensus statements
- Flags contradictions across sources
- Generates claim confidence scores

**Chronological synthesis**:
- Timelines for developments
- Trend analysis with statistical significance
- Frequency of terminology over time

**Contradiction analysis**:
- Cross-source fact checking
- Confidence intervals for claims
- Source reliability scoring (based on citation count, venue impact)

Example synthesis command:
```bash
meta-builder synthesize req_20240315_001 --method=thematic --output=report.md
```

Generates report with sections:
```
# Research Synthesis: Transformer Architecture Improvements (2022-2024)

## Executive Summary
- 3 major trends identified
- 12 consensus claims (high confidence)
- 4 contradictions requiring manual review
- 5 emerging techniques (low confidence, early stage)

## Trend 1: Efficient Attention Mechanisms
**Consensus**: Linear attention variants reduce complexity from O(n²) to O(n)
**Sources**: 23 papers, 5 GitHub implementations
**Confidence**: 0.92

## Contradictions
1. **Claim**: "FlashAttention achieves 2x speedup"
   - Supporting: arXiv:2305.12345 (3.1x), arXiv:2307.67890 (1.8x)
   - Contradicting: arXiv:2309.11111 (1.2x on GPU architecture X)
   - Resolution: Performance varies by hardware (see Section 4.2)
```

### Step 5: Citation & Verification
Command: `meta-builder cite <research_id> [--format=apa|mla|chicago|bibtex]`
- Generates formatted bibliography
- Cross-links citations to claims in report
- Verifies DOI/URL accessibility
- Checks for citation completeness

```bash
meta-builder cite req_20240315_001 --format=apa --include_verified=true
# Outputs: req_20240315_001_citations.bib
# With annotation: {verified: true, accessed: "2024-03-15", doi: "10.xxxx/xxxx"}
```

### Step 6: Export & Integration
Command: `meta-builder export <research_id> [--web] [--notion] [--obsidian]`
Formats:
- `markdown` (default): GitHub-flavored with TOC
- `pdf`: via pandoc (requires `pandoc` and `latex`)
- `json`: structured data for programmatic use
- `notion`: uploads to Notion workspace (requires NOTION_API_KEY)
- `obsidian`: creates Markdown with wiki-links for Obsidian vault
- `web`: generates static HTML site with search

## Golden Rules

1. **Source Diversity**: Always use at least 3 independent sources for any factual claim. Single-source findings are flagged as "preliminary".

2. **Rate Limiting Respect**: Never exceed source-specific rate limits. Built-in delays:
   - arXiv: 3s between requests
   - Semantic Scholar: 1s
   - GitHub: use token for higher limits
   - News APIs: provider-specific (default 1 request/2s)

3. **Credential Isolation**: All API keys stored in `~/.openclaw/config/ext/credentials.json` (never in code or command line). Loaded via:
   ```bash
   export META_BUILDER_CREDENTIALS=~/.openclaw/config/ext/credentials.json
   ```

4. **Checkpoint Everything**: After each paper/fetched item, write state to `~/.openclaw/meta-builder/checkpoints/<research_id>.json`:
   ```json
   {
     "completed_sources": ["arxiv", "semantic_scholar"],
     "failed": [{"source": "github", "error": "rate_limit", "retry_at": "2024-03-15T14:30:00Z"}],
     "papers_fetched": 47,
     "last_item_id": "arxiv_2301.00001"
   }
   ```

5. **Token Budget Enforcement**: Maximum token budget per research (default 200,000). LLM calls tracked. Use `--max_tokens=<n>` to override.

6. **PII Redaction**: Automatically redact email addresses, phone numbers, private keys from scraped content. Replaces with `[REDACTED_<TYPE>]`.

7. **Source Attribution**: Every extracted fact must include:
   - Source URL
   - Access date
   - Confidence score (0-1)
   - Extraction method (pdf, html, api)

8. **No Manufactured Data**: Never infer beyond source material. Distinguish clearly between:
   - Direct quote (exact source text)
   - Paraphrased content (with interpretation flag)
   - Model synthesis (explicitly labeled)

9. **Cost Transparency**: Display estimated cost before expensive operations (>100 API calls or >50k tokens). Require `--confirm_budget` flag.

10. **Reproducibility**: Seed all random operations (embeddings, clustering) with `--seed=<integer>` stored in research manifest.

## Examples

### Example 1: Academic Literature Review
```bash
# Plan the research first
meta-builder research "few-shot learning for medical imaging 2023-2024" --plan_only --sources=arxiv,semantic_scholar,pubmed

# Output:
# Plan approved. Estimated tokens: 85,000. Cost: $2.13
# Sources: arxiv (50 papers), semantic_scholar (120 papers), pubmed (45 papers)
# Sub-queries:
#   - "few-shot segmentation medical images"
#   - "meta-learning radiology"
#   - "prompt-based medical diagnosis"

# Execute with approval
meta-builder research "few-shot learning for medical imaging 2023-2024" --execute --depth=3 --sources=arxiv,semantic_scholar,pubmed --max_results=50

# Process extracted text
meta-builder process fewshot_medical_20240315 --extract=pdf --summarize

# Synthesize findings thematically
meta-builder synthesize fewshot_medical_20240315 --method=thematic --output=report.md

# Generate APA citations
meta-builder cite fewshot_medical_20240315 --format=apa > citations.bib

# Export to Obsidian vault
meta-builder export fewshot_medical_20240315 --obsidian --vault=~/vaults/research/
```

### Example 2: Security Vulnerability Tracking
```bash
# Monitor for new log4j vulnerabilities
meta-builder monitor "CVE-2021-44228 log4j" \
  --frequency="0 9 * * *" \
  --sources=nvd,github,security_advisories \
  --alert_threshold=0.8 \
  --output=json

# Cron job created in ~/.openclaw/meta-builder/cron/monitor_log4j.json
# Runs daily at 9 AM. If new vulnerability with confidence >0.8 found:
# - Creates new research ID
# - Runs full analysis pipeline
# - Sends alert to webhook (configured in ~/.openclaw/config/ext/alerts.json)
```

### Example 3: Codebase Research
```bash
# Analyze React 18 server components across multiple repos
meta-builder research "react 18 server components implementation patterns" \
  --sources=github \
  --query_filter="language:javascript stars:>1000 topic:react-18" \
  --depth=2 \
  --extract=code \
  --output=patterns_report.md

# Extracts:
# - README explanations
# - Example implementations in /app, /pages directories
# - Package.json dependencies
# - Issues tagged "server-components"
# Generates comparative table of patterns across repos
```

### Example 4: News Sentiment Analysis
```bash
# Track sentiment on AI regulation news
meta-builder research "EU AI Act implementation" \
  --sources=news \
  --date_range="2024-01-01:2024-03-15" \
  --sentiment_analysis=true \
  --output=timeline.html

# Requires NEWS_API_KEY in credentials
# Generates interactive timeline with:
# - Sentiment scores per article
# - Top mentioned entities (companies, politicians)
# - Geographic distribution of sources
```

## Configuration

Environment variables (set in `~/.openclaw/config/ext/credentials.json`):
```json
{
  "meta_builder": {
    "openai_api_key": "sk-...",         // Required for synthesis
    "anthropic_api_key": "sk-...",      // Optional, alternative LLM
    "github_token": "ghp_...",          // Higher GitHub rate limits
    "news_api_key": "...",              // News source access
    "semantic_scholar_key": "...",      // Optional for higher limits
    "arxiv_user_agent": "MetaBuilder/1.0 (your.email@domain.com)",
    "default_output_dir": "~/.openclaw/meta-builder/output",
    "checkpoint_dir": "~/.openclaw/meta-builder/checkpoints",
    "data_dir": "~/.openclaw/meta-builder/data",
    "max_concurrent_downloads": 5,
    "default_embedding_model": "text-embedding-3-small"
  }
}
```

Command-line overrides:
```bash
meta-builder research "query" --llm=claude --embedding_model=text-embedding-ada-002
```

## Verification

Post-research verification checklist:
```bash
# 1. Check source diversity
meta-builder verify <research_id> --sources_min=3

# 2. Validate all citations are accessible
meta-builder verify <research_id> --check_links --timeout=5

# 3. Ensure token budget not exceeded
meta-builder verify <research_id> --token_report

# 4. Verify cross-source contradictions flagged
meta-builder verify <research_id> --contradictions

# 5. Generate completeness report
meta-builder verify <research_id> --completeness --expected_claims=<n>
```

Output:
```json
{
  "research_id": "req_20240315_001",
  "verification_passed": true,
  "checks": {
    "source_diversity": {"passed": true, "sources_used": 5, "min_required": 3},
    "citation_validity": {"passed": true, "broken_links": 0, "total": 127},
    "token_budget": {"passed": true, "used": 85000, "budget": 200000},
    "contradictions_flagged": {"passed": true, "count": 4},
    "completeness": {"passed": false, "missing_claims": 3}
  }
}
```

## Troubleshooting

### Rate Limit Exceeded
```bash
# Check current rate limits status
meta-builder status --rate_limits

# Manually set delay between requests
meta-builder config set arxiv.delay 5.0  # seconds

# Resume from checkpoint (automatic on restart)
meta-builder research "query" --resume --research_id=req_20240315_001
```

### API Key Errors
```bash
# Validate credentials
meta-builder auth test --all

# Specific test:
meta-builder auth test --source=arxiv
# Output: "✓ arxiv authentication valid (user_agent: MetaBuilder/1.0)"
```

### Memory Issues (Large PDFs)
```bash
# Process in chunks
meta-builder process <research_id> --chunk_size=10 --overlap=2

# Use lighter extraction (skip images/tables)
meta-builder process <research_id> --extract=text_only
```

### Incomplete Results
```bash
# Retry failed items only
meta-builder retry <research_id> --only_failed

# Expand depth (fetch more sources)
meta-builder expand <research_id> --depth=5 --sources=add_pubmed
```

### LLM Cost Overrun
```bash
# Check token usage before synthesis
meta-builder usage <research_id> --breakdown

# Force cheaper model
meta-builder synthesize <research_id> --model=gpt-3.5-turbo-0125 --max_tokens=4000
```

## Rollback Commands

### 1. Cancel Active Research
```bash
# Cancel running research (graceful shutdown)
meta-builder cancel <research_id_or_pid>

# Force kill (if hung)
meta-builder cancel <research_id> --force
# Removes: checkpoint file, temporary output, but preserves raw/ and processed/
```

### 2. Remove Research Artifacts
```bash
# Delete entire research project (with confirmation)
meta-builder delete <research_id> --purge

# Interactive mode:
meta-builder delete <research_id>
# Prompts:
#   Keep raw/ data? [y/N]
#   Keep processed/ data? [y/N]
#   Keep report outputs? [y/N]
#   Destroy checkpoint? [y/N]

# Prune all research older than 30 days
meta-builder prune --older_than=30d --dry_run
meta-builder prune --older_than=30d --exclude=req_20240315_001
```

### 3. Revert Configuration Changes
```bash
# View config history
meta-builder config history

# Restore previous config
meta-builder config restore --backup=config_20240315_120000.json

# Reset to defaults (backup auto-created)
meta-builder config reset
```

### 4. Restore from Checkpoint
```bash
# Auto-resume from last checkpoint on interrupt
meta-builder research "query" --resume --research_id=req_20240315_001

# Specify older checkpoint (if checkpoints archived)
meta-builder restore <research_id> --checkpoint=req_20240315_001_chkpt_20240315_143022.json
# Restores state and continues from that point
```

### 5. Database Reset (if using SQLite backend)
```bash
meta-builder db reset --confirm
# Backs up: ~/.openclaw/meta-builder/state.db → ~/.openclaw/meta-builder/backups/
# Then recreates empty state database
# Safe: raw/ and processed/ data preserved
```

### 6. Rollback Single Operation
```bash
# Undo last synthesis (regenerates from processed/ data)
meta-builder unsynthesize <research_id>
# Deletes: output/report.md, output/citations.bib
# Keeps: raw/, processed/

# Undo processing (reverts to raw state)
meta-builder unprocess <research_id>
# Deletes: processed/
# Keeps: raw/
```

## Performance Optimization

- Use `--batch_size=<n>` for bulk operations
- Set `--embedding_cache=true` to cache embeddings across research projects
- Parallel sources: `--concurrent=3` (default: 5)
- Disable heavy features: `--skip_embeddings` (faster but no clustering)
- Use local LLM: `--llm=local --model=/path/to/gguf` (requires `ollama` or `llama.cpp`)

## Integration with OpenClaw

Meta Builder integrates with:
- **workflow engine**: Can be chained as step in pipelines
- **git hooks**: Auto-generate research reports on commit to `research/` directory
- **notifications**: Sends alerts to configured webhooks on completion/failure
- **storage**: Auto-archives to S3/MinIO if `S3_BUCKET` configured

Example workflow integration:
```bash
openclaw workflow add step meta-builder-research \
  --after=git-clone \
  --command="meta-builder research '${TASK_QUERY}' --execute --sources=arxiv,github"
```

```