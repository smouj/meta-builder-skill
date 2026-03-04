name: meta-builder
description: Motor de automatización y síntesis de investigación impulsado por IA
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

Motor de automatización y síntesis de investigación impulsado por IA para flujos de trabajo de investigación académica, técnica y de mercado.

## Purpose

Meta Builder automatiza tareas de investigación completas que incluyen:
- Revisiones de literatura académica y análisis de artículos (arXiv, PubMed, Semantic Scholar)
- Recopilación de inteligencia competitiva y análisis de mercado
- Análisis en profundidad de frameworks/libraries con extracción de ejemplos de código
- Análisis de documentos regulatorios e investigación de cumplimiento
- Síntesis de información de múltiples fuentes en informes coherentes
- Mapeo automático de redes de citas
- Análisis de panoramas de patentes
- Seguimiento de sentimiento de noticias a través de períodos de tiempo

Flujos de trabajo reales:
- "Analizar las mejoras en arquitecturas transformer de los últimos 2 años y crear una línea de tiempo"
- "Comparar benchmarks de rendimiento de React 18 vs Vue 3 desde 10 fuentes independientes"
- "Investigar requisitos de cumplimiento GDPR para startups SaaS en mercados de la UE"
- "Recopilar todas las vulnerabilidades de seguridad para log4j desde NVD, avisos de GitHub y blogs de seguridad"

## Alcance

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

Meta Builder se integra con estas fuentes de investigación (habilitar via `--sources`):
- `arxiv`: Papers preimpresión (habilitado por defecto)
- `semantic_scholar`: Citas académicas y papers influyentes
- `pubmed`: Literatura biomédica
- `patents`: Patentes USPTO e internacionales
- `github`: Análisis de problemas/issues, PRs, README de repositorios
- `nvd`: National Vulnerability Database
- `security_advisories`: GitHub Security Advisory Database
- `news`: APIs de noticias (requiere NEWS_API_KEY)
- `reddit`: Análisis de contenido de subreddits
- `stackexchange`: Sitios de Q&A para investigación técnica
- `legal`: Regulaciones y documentos de cumplimiento (requiere APIs especializadas)
- `custom`: URLs proporcionadas por el usuario o archivos locales

## Proceso de Trabajo

### Step 1: Research Planning Phase
Command: `meta-builder research <query> --plan_only`
- Parsea la consulta en sub-preguntas usando LLM
- Identifica tipos de fuente requeridos (académica, noticias, código, patentes)
- Estima uso de tokens y costo
- Genera plan de investigación para aprobación
- Crea manifiesto `research_<id>.plan.json`

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
- Obtención secuencial de fuentes con limitación de velocidad
- Descargas paralelas desde la misma fuente (máximo 5 concurrentes)
- Capacidad de reanudación (almacena checkpoint en `~/.openclaw/meta-builder/checkpoints/`)
- Datos sin procesar almacenados en `~/.openclaw/meta-builder/data/<research_id>/raw/`

Límites de velocidad por fuente (configurables):
- arXiv: 1 solicitud/3 segundos
- Semantic Scholar: 1 solicitud/segundo
- GitHub: 5000 solicitudes/hora (usa token si GITHUB_TOKEN está configurado)
- News APIs: específico del proveedor

### Step 3: Extraction & Processing
Command: `meta-builder process <research_id> [--extract=<pdf|html|code|json>]`
- Extracción de texto PDF usando PyMuPDF
- Análisis HTML (BeautifulSoup4) con aislamiento de contenido
- Extracción de bloques de código y detección de lenguaje
- Extracción de tablas a CSV/JSON
- Extracción de metadatos (autores, fechas, citas)
- Deduplicación entre fuentes (similitud coseno >0.95)

Example extraction:
```bash
meta-builder process req_20240315_001 --extract=pdf --output=structured
# Outputs: processed/<research_id>/extracted_text.json
# con estructura:
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
**Síntesis temática** (por defecto):
- Agrupa documentos por similitud de embeddings
- Identifica declaraciones de consenso
- Marca contradicciones entre fuentes
- Genera puntuaciones de confianza de afirmaciones

**Síntesis cronológica**:
- Líneas de tiempo de desarrollos
- Análisis de tendencias con significancia estadística
- Frecuencia de terminología a lo largo del tiempo

**Análisis de contradicciones**:
- Verificación cruzada de hechos
- Intervalos de confianza para afirmaciones
- Puntuación de fiabilidad de fuentes (basada en número de citas, impacto del venue)

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
- Genera bibliografía formateada
- Enlaza citas cruzadamente a afirmaciones en el informe
- Verifica accesibilidad de DOI/URL
- Verifica integridad de citas

```bash
meta-builder cite req_20240315_001 --format=apa --include_verified=true
# Outputs: req_20240315_001_citations.bib
# Con anotación: {verified: true, accessed: "2024-03-15", doi: "10.xxxx/xxxx"}
```

### Step 6: Export & Integration
Command: `meta-builder export <research_id> [--web] [--notion] [--obsidian]`
Formatos:
- `markdown` (por defecto): GitHub-flavored con TOC
- `pdf`: via pandoc (requiere `pandoc` y `latex`)
- `json`: datos estructurados para uso programático
- `notion`: sube a workspace Notion (requiere NOTION_API_KEY)
- `obsidian`: crea Markdown con wiki-links para vault Obsidian
- `web`: genera sitio HTML estático con búsqueda

## Reglas de Oro

1. **Source Diversity**: Siempre usar al menos 3 fuentes independientes para cualquier afirmación fáctica. Hallazgos de fuente única se marcan como "preliminary".

2. **Rate Limiting Respect**: Nunca exceder límites de velocidad específicos de fuente. Retardos integrados:
   - arXiv: 3s entre solicitudes
   - Semantic Scholar: 1s
   - GitHub: usa token para límites más altos
   - News APIs: específico del proveedor (por defecto 1 solicitud/2s)

3. **Credential Isolation**: Todas las API keys almacenadas en `~/.openclaw/config/ext/credentials.json` (nunca en código o línea de comandos). Cargadas via:
   ```bash
   export META_BUILDER_CREDENTIALS=~/.openclaw/config/ext/credentials.json
   ```

4. **Checkpoint Everything**: Después de cada paper/item obtenido, escribir estado en `~/.openclaw/meta-builder/checkpoints/<research_id>.json`:
   ```json
   {
     "completed_sources": ["arxiv", "semantic_scholar"],
     "failed": [{"source": "github", "error": "rate_limit", "retry_at": "2024-03-15T14:30:00Z"}],
     "papers_fetched": 47,
     "last_item_id": "arxiv_2301.00001"
   }
   ```

5. **Token Budget Enforcement**: Presupuesto máximo de tokens por investigación (por defecto 200,000). Llamadas LLM rastreadas. Usar `--max_tokens=<n>` para sobrescribir.

6. **PII Redaction**: Enmascarar automáticamente direcciones de email, números de teléfono, claves privadas de contenido scrapeado. Reemplaza con `[REDACTED_<TYPE>]`.

7. **Source Attribution**: Cada hecho extraído debe incluir:
   - URL de fuente
   - Fecha de acceso
   - Puntuación de confianza (0-1)
   - Método de extracción (pdf, html, api)

8. **No Manufactured Data**: Nunca inferir más allá del material fuente. Distinguir claramente entre:
   - Cita directa (texto exacto de fuente)
   - Contenido parafraseado (con flag de interpretación)
   - Síntesis del modelo (explicitamente etiquetada)

9. **Cost Transparency**: Mostrar costo estimado antes de operaciones costosas (>100 llamadas API o >50k tokens). Requiere flag `--confirm_budget`.

10. **Reproducibility**: Semillar todas las operaciones aleatorias (embeddings, clustering) con `--seed=<integer>` almacenado en manifiesto de investigación.

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

## Solución de Problemas

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

Meta Builder se integra con:
- **workflow engine**: Puede encadenarse como paso en pipelines
- **git hooks**: Genera automáticamente informes de investigación en commit a directorio `research/`
- **notifications**: Envía alertas a webhooks configurados en completion/failure
- **storage**: Archiva automáticamente a S3/MinIO si `S3_BUCKET` configurado

Example workflow integration:
```bash
openclaw workflow add step meta-builder-research \
  --after=git-clone \
  --command="meta-builder research '${TASK_QUERY}' --execute --sources=arxiv,github"
```
```