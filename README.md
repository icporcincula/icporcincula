# Bart Porcincula — Senior AI Engineer

LLM agents, evaluation and backend systems. **TypeScript · Python · Go.** Based in the Philippines, open to relocation to **Ireland / UK**.

I build production agent systems and the evaluation tooling that shows whether a change actually changed anything. I use a model only where the output is language or a judgement call and keep every other decision deterministic. I measure before I claim, and I correct my own design records when the data proves them wrong.

📫 [porcincula.developer@gmail.com](mailto:porcincula.developer@gmail.com) · 🌐 [porsync.com](https://www.porsync.com)

---

## Now

**Senior AI Engineer, TOA Global** (June 2026 – present). Building agents for a production multi-agent recruitment-workflow platform on Azure (TypeScript, Claude, Service Bus, Azure SQL, Cosmos DB). The platform is a finalist in the **Australian AI Awards 2026** (AI Innovator – Human Resources). That work is closed-source; the projects below are my own.

---

## Featured work

### slm-mesh: can fine-tuned small models replace a 4B generalist in a live pipeline?

*A controlled study on a real lead-qualification workflow: a prompted 4B generalist against a mesh of fine-tuned specialists (Qwen3 0.6B/1.7B LoRAs and a ModernBERT encoder), both running locally on one 16 GB consumer GPU.*

**Result: training bought reliability and efficiency, not accuracy.**

- **12× faster:** 828 s against 9,925 s per 1,000 leads, with all four specialists resident in **5.5 GB VRAM** (served as LoRA hot-swap on a shared Q4_K_M base; an adapter swap costs 31 ms)
- **Deterministic:** across seeded repeats not one score, tier or decision changed, while the teacher's reject recall swung 0.13 between identical runs
- **Schema defect class removed:** GBNF grammar generated from the data contract gives 100% schema validity and zero alias-map dependency across 6,378 records
- **Accuracy was a tie, and the report says so.** The negative-class gate failed once scored on balanced accuracy. Both models' reject precision sits at the base rate, because the students inherited the teacher's bias from its labels. The labels set the ceiling, not the model size.

**How it's kept honest:**
- Pass/fail thresholds locked in the plan before any training ran
- Human-labelled gold set drawn only from the temporally latest split; the teacher is never treated as ground truth (ADR)
- Temporal, entity-deduplicated splits so no business crosses a train/test boundary
- Every number in the write-up is re-derived by a check script from the results database, and the check fails on any unmarked metric
- Shadow-mode A/B against production before any traffic flip; production untouched throughout

**Stack:** Unsloth / TRL SFT · LoRA · ModernBERT · llama.cpp (CUDA, multi-LoRA) · GGUF · GBNF · DuckDB · dbt · n8n

---

### Data lakehouse: finding and fixing silent data loss in a production pipeline

*Opened a live lead-generation pipeline (~6,400 businesses scraped twice daily) with a formal data-quality audit. It found defects that had never raised an error, failed a run or fired an alert. Rebuilt as a Fabric-shaped medallion lakehouse so the same class of bug can't recur silently.*

Write-up: [porsync.com/case-studies/silent-data-loss-lead-pipeline](https://www.porsync.com/case-studies/silent-data-loss-lead-pipeline)

```
Google Maps — 10 categories × 28 PH cities, 2×/day
  │
  ▼
n8n scrape → LLM qualify (local Gemma) → Postgres               bronze (raw truth)
  │
  ▼
Delta Lake export (delta-rs, no JVM)                            contract-validated,
  │                                                              quarantine-rate gate
  ▼
DuckDB warehouse → dbt star schema                              silver / gold
  │                  SCD-2 business dim, event-grain fact
  ▼
Dagster — scheduled + freshness sensor, live daemon             reconciled vs. source
```

**What the audit found:**
- A dedup key the scraper never emitted silently fell back to matching on website domain: **545 distinct businesses wrongly merged** (8.5% of the table) on every run
- A `max_tokens: 300` setting truncated **57.8%** of one scraper's enrichment mid-JSON, stored as "nothing found" and indistinguishable from a real empty result
- A placeholder cross join fanned a fact table out **28×** while `dbt build` stayed green, because the tests checked nulls and enums, not grain
- The silver contract, not the data, was wrong: it understated "hot" leads by 63% because the scoring rules lived in five unlinked places

**Key decisions:**
- One data contract as the source of truth; every consumer is generated from it and `generate.py --check` gates CI
- A quarantine-rate gate instead of silent drops, with its ceiling pinned to a real historical rate so a gate that can't fire doesn't pass for one that works
- Scoring provenance (`scored_by_model`, `prompt_version`, `rules_version`) on every event
- Caught a `RUNNING` schedule flag that described intent rather than a working daemon; the fix includes a test that a tick actually fired

**Stack:** n8n · Postgres (Supabase) · Delta Lake (delta-rs) · DuckDB · dbt · Dagster · GitHub Actions

---

## Stack

| **LLM & agents** | **Fine-tuning & serving** | **Backend & data** |
| :--- | :--- | :--- |
| Claude API · tool-calling loops · streaming · prompt caching | LoRA · TRL SFT · Unsloth · ModernBERT | TypeScript / Node · Python (FastAPI) · Go |
| LangGraph · MCP · prompt-injection containment | GGUF · llama.cpp (CUDA) · Ollama · LM Studio | Azure Functions · Service Bus · Azure SQL · Cosmos DB |
| LLM evaluation: reference sets, control arms, noise floors | Presidio PII de-identification | Postgres · DuckDB · dbt · Dagster · Delta Lake · Redis · Qdrant |

---

## Earlier projects

| Project | What it is | Stack |
| :--- | :--- | :--- |
| **[Sentinel-Extract](https://github.com/icporcincula/ai-document-analyzer)** | Air-gapped PII detection and document intelligence: hybrid NER (Presidio + spaCy), local LLM reasoning, OCR ingestion | Python · Presidio · Ollama · FastAPI · Tesseract |
| **[Vela](https://github.com/icporcincula/vela-pii-compliance)** | Compliance and audit layer on top of the [Eidolon](https://github.com/0M3REXE/eidolon) PII-redaction proxy: async audit webhooks off the hot path, per-tenant rules, immutable audit log | Go · React · SQLite · Docker |
| **[Filo](https://github.com/icporcincula/filo-pinoy-ai-receptionist)** | Fully local voice agent: VAD → Faster-Whisper → Ollama → Kokoro TTS, no cloud APIs | Go · Faster-Whisper · Ollama · Kokoro · Redis |
| **Config-driven agent service** | One LangGraph ReAct service for many tenants: a new tenant is a config file, tools self-register, Redis session window + Qdrant memory | Python · FastAPI · LangGraph · Qdrant · Redis |
| **Podcast specialist** | YouTube audio → 291-example dataset → Qwen3.5-4B LoRA (eval loss 3.54 → 1.69) → merged GGUF q4_K_M served on Ollama | Unsloth · LoRA · llama.cpp · Ollama |
| **LangGraph vs CrewAI benchmark** | Same lead-research task on both frameworks: 128.5 s (LangGraph) against 1,178 s (CrewAI) | LangGraph · CrewAI · Playwright |
| **Local UGC video pipeline** | LLM script → voice clone → diffusion lip-sync → ffmpeg, on one consumer GPU (Blackwell sm_120 build) | Ollama · Fish Speech · LatentSync · ffmpeg |
| **Product-photo relighting** | SDXL img2img with per-preset denoise so product identity survives restyling | ComfyUI · SDXL · FastAPI |

---

## Background

5+ years of backend engineering. Senior Software Engineer at **Invensity GmbH** (2021–2026) on client work for German companies, including Robert Bosch GmbH: led an AI transcription and summarisation product end to end (Deepgram, custom Presidio PII models, PHP-to-Go migration, CI/CD), and built EU-compliant e-invoicing and whistleblower-reporting tools. GDPR-aware by habit.
