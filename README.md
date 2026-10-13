# Bart Porcincula — Senior AI Engineer

LLM agents, evaluation and backend systems. **TypeScript · Python · Go.** Based in the Philippines, open to relocation to **Ireland / UK** or remote roles.

I build production agent systems and the evaluation tooling that shows whether a change actually changed anything. I use a model only where the output is language or a judgement call and keep every other decision deterministic. I measure before I claim, and I correct my own design records when the data proves them wrong.

📫 [porcincula.developer@gmail.com](mailto:porcincula.developer@gmail.com) · 🌐 [porsync.com](https://www.porsync.com) · 📄 [Research write-ups](https://www.porsync.com/research)

---

## Now

**Senior AI Engineer, TOA Global** (June 2026 – present). Building agents for a production multi-agent recruitment-workflow platform on Azure (TypeScript, Claude, Service Bus, Azure SQL, Cosmos DB). That work is closed-source; the projects below are my own.

**Outside work:** a series of pre-registered experiments on small open models running on one 16 GB consumer GPU, covering fine-tuning, calibration, quantisation, serving and small-to-frontier routing. Every number is re-derived from a results database, and negative results are written up as results.

**Certifications in progress:** Microsoft AI-103 (Azure AI Apps and Agents Developer) and AI-500 (Multi-Agent AI Solutions), exams scheduled Q1 2027.

---

## How I run experiments

- **Pre-registration:** the hypothesis, metric, pass/fail threshold and verdict rule are frozen before any result exists. Later changes are dated amendments.
- **Paired bootstrap confidence intervals** over the unit that matters (contract, entity, ticket), not raw deltas.
- **Noise floors first:** run-to-run variance is measured before any change is judged against it.
- **Leakage-tested splits:** temporal, entity-deduplicated or by-document, never by row.
- **Honest verdicts:** an uncleared gate is reported as uncleared, and "unanswered" is not reported as "no".

---

## Featured work

### slm-mesh: can fine-tuned small models replace a 4B generalist in a live pipeline?

*A controlled study on a real lead-qualification workflow: a prompted 4B generalist against fine-tuned specialists (Qwen3 0.6B/1.7B LoRAs and a ModernBERT encoder), both running locally on one 16 GB GPU.*

📄 Write-up: [The teacher sets the ceiling](https://www.porsync.com/research/the-teacher-sets-the-ceiling-slm-distillation)

**Result: training bought reliability and efficiency, not accuracy.**

- **12× faster:** 828 s against 9,925 s per 1,000 leads, with all specialists resident in **5.5 GB VRAM** (LoRA hot-swap on a shared Q4_K_M base)
- **Deterministic:** across seeded repeats no score, tier or decision changed, while the teacher's reject recall swung 0.13 between identical runs
- **Schema defect class removed:** GBNF grammar generated from the data contract gives 100% schema validity across 6,378 records
- **Accuracy was a tie, and the report says so.** Ablations showed grammar, model size and even a 97% cut in training data made no measurable difference. Only the label source did: a stronger teacher added about +10 points balanced accuracy. The labels set the ceiling, not the model size.

**Stack:** Unsloth / TRL SFT · LoRA · ModernBERT · llama.cpp (CUDA, multi-LoRA) · GGUF · GBNF · DuckDB · dbt

---

### cuad-mesh: small model vs frontier model on contract review

*Two pre-registered studies on [CUAD](https://arxiv.org/abs/2103.06268) (510 public contracts, 41 clause types plus verbatim span extraction, split 347 / 61 / 102 by contract).*

**1. Can a QLoRA-tuned 4B model match Sonnet 5.5?** 📄 [Write-up](https://www.porsync.com/research/tuning-a-small-model-precision-not-recall-cuad)

- Gate set in advance: within 5 macro-F1 points of the frontier model (0.783; untuned 4B 0.715, paired 95% CI −10.0 to −3.0)
- Four tuning rounds did not clear the validation gate, so **no tuned model earned a gated test run**, and the main comparison against the frontier model is reported as unanswered, not "no"
- Tuning raised precision (0.78 → 0.93) and verbatim-quote fidelity (87% → 97%) but not recall. Where an invented quote is costly, the tuned model is arguably the better extractor even at an equal score.

**2. Can the small model's confidence decide when to escalate to the frontier model?** 📄 [Write-up](https://www.porsync.com/research/small-model-confidence-cannot-route-to-frontier-cascade)

- Five routing signals against random escalation at 20 / 30 / 40% of contracts. **None reliably beat random.**
- Per-answer routing did find the frontier model's wins (0.766 vs 0.715 random at 10% of answers), **but it saved no money.** Frontier cost is driven by reading the document, so escalating 10% of answers touched 93 of 102 contracts and cost 95% of always-frontier.
- The useful conclusion is about cost structure: savings have to come from amortising the read (batching, cached second passes), not from routing.

**Stack:** QLoRA · gemma-4-e4b · Claude API · llama.cpp · paired bootstrap · SQLite

---

### Local inference serving: what serving choices cost, and what they change

*26 configurations × 3 runs of Qwen3.5-9B classifying 500 support tickets on one RTX 5060 Ti, changing one setup choice at a time.*

📄 Write-up: [Speed, not accuracy](https://www.porsync.com/research/serving-choices-speed-not-accuracy-local-llm)

- **Best setup was 2.2× the Ollama default** (llama-server, 4-bit, 4 concurrent: 178 vs 80 tickets/min). The same model file served by llama-server instead of Ollama was 1.54× faster with nothing else changed.
- **$0.0032 per 1,000 tickets** in electricity, against $22.43 per 1,000 through Claude on the same task
- **Aggregate accuracy didn't move, individual answers did:** 4-bit vs 8-bit changed 8% of category answers, and folding three questions into one request changed 16–19%
- Traced a flat concurrency curve to the server log: Ollama 0.17.6 serialises this architecture whatever `OLLAMA_NUM_PARALLEL` says

**Stack:** llama.cpp (llama-server) · Ollama · GGUF · Python

---

### Data lakehouse: finding and fixing silent data loss in a production pipeline

*Opened a live lead-generation pipeline (~6,400 businesses scraped twice daily) with a formal data-quality audit. It found defects that had never raised an error, failed a run or fired an alert. Rebuilt as a Fabric-shaped medallion lakehouse so the same class of bug can't recur silently.*

📄 Write-up: [porsync.com/case-studies/silent-data-loss-lead-pipeline](https://www.porsync.com/case-studies/silent-data-loss-lead-pipeline)

```
n8n scrape → LLM qualify (local) → Postgres                     bronze (raw truth)
  ▼
Delta Lake export (delta-rs, no JVM)                            contract-validated, quarantine-rate gate
  ▼
DuckDB warehouse → dbt star schema (SCD-2 dim, event-grain fact) silver / gold
  ▼
Dagster — schedule + freshness sensor                           reconciled vs. source, 197+ assertions
```

**What the audit found:**
- A dedup key the scraper never emitted silently fell back to website domain: **545 distinct businesses wrongly merged** on every run
- A `max_tokens: 300` setting truncated **57.8%** of one scraper's enrichment mid-JSON, stored as "nothing found"
- A placeholder cross join fanned a fact table out **28×** while `dbt build` stayed green, because the tests checked nulls and enums, not grain

**Stack:** n8n · Postgres (Supabase) · Delta Lake · DuckDB · dbt · Dagster · GitHub Actions

---

## In progress

**Cascade study 2: what is the best local router per consumer VRAM tier?** A pre-registered follow-up comparing a 9B model, a 27B model at three quantisation levels and a ternary 27B across 8 / 12 / 16 GB tiers, in front of the same frontier model on the same CUAD task. Phase 0 and 1 (probes, measured throughput, frozen protocol) are done and Phase 2 is running.

---

## Stack

| **LLM & agents** | **Fine-tuning & serving** | **Backend & data** |
| :--- | :--- | :--- |
| Claude API · tool-calling loops · streaming · prompt caching | LoRA · QLoRA · TRL SFT · Unsloth · ModernBERT | TypeScript / Node · Python (FastAPI) · Go |
| LangGraph · MCP · prompt-injection containment | Calibration (ECE, temperature scaling) · OOD gating | Azure Functions · Service Bus · Azure SQL · Cosmos DB |
| LLM evaluation: reference sets, control arms, noise floors | GGUF · llama.cpp (CUDA) · vLLM · Ollama | Postgres · DuckDB · dbt · Dagster · Delta Lake · Redis · Qdrant |
| Pre-registration · paired bootstrap · power checks | Presidio PII de-identification | Docker · GitHub Actions · Azure DevOps |

---

## Other projects

| Project | What it is | Stack |
| :--- | :--- | :--- |
| **tally** | CPU-first calibrated decision model: one ModernBERT pass returns probabilities, a calibrated confidence and an act/escalate flag. Beats TF-IDF on Banking77 (0.909) and SMS spam (0.990) over 3 seeds, test ECE 0.008 / 0.005. Reports its misses: the Mahalanobis OOD gate caught 76% of unseen German tickets against a pre-registered 90%. | ModernBERT · PyTorch · calibration |
| **[Vela](https://github.com/icporcincula/vela-pii-compliance)** | Compliance and audit layer on top of the [Eidolon](https://github.com/0M3REXE/eidolon) PII-redaction proxy: async audit capture off the hot path, per-tenant rules, immutable audit log | Go · React · SQLite · Docker |
| **[Sentinel-Extract](https://github.com/icporcincula/ai-document-analyzer)** | Air-gapped PII detection and document intelligence: hybrid NER (Presidio + spaCy), local LLM reasoning, OCR ingestion | Python · Presidio · Ollama · FastAPI · Tesseract |
| **Multi-agent system** | Go orchestrator runs YAML-defined pipelines (dependency ordering, cycle detection, human-approval points) over Python agents on Ollama, coordinated through Redis, with Qdrant memory | Go · Python · gRPC · Redis · Qdrant |
| **Config-driven agent service** | One LangGraph ReAct service for many tenants: a new tenant is a config file, tools self-register, Redis session window + Qdrant memory | Python · FastAPI · LangGraph · Qdrant · Redis |
| **[Filo](https://github.com/icporcincula/filo-pinoy-ai-receptionist)** | Fully local voice agent: VAD → Faster-Whisper → Ollama → Kokoro TTS, no cloud APIs | Go · Faster-Whisper · Ollama · Kokoro · Redis |
| **LangGraph vs CrewAI benchmark** | Same lead-research task on both frameworks: 128.5 s (LangGraph) against 1,178 s (CrewAI) | LangGraph · CrewAI · Playwright |

---

## Background

5+ years of backend engineering. Senior Software Engineer at **Invensity GmbH** (2021–2026) on client work for German companies, including Robert Bosch GmbH: led an AI transcription and summarisation product end to end (Deepgram, custom Presidio PII models, PHP-to-Go migration, CI/CD), and built EU-compliant e-invoicing and whistleblower-reporting tools. GDPR / UK GDPR-aware by habit. BS Computer Engineering, Technological Institute of the Philippines (2021).
