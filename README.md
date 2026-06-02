# Portfolio Summary

Building custom AI automation systems for domain experts — agentic pipelines, private inference infrastructure, and observable backend services. Focus on eliminating cloud lock-in and keeping sensitive data on-premise.

At [Porsync](https://www.porsync.com) I work with operators who need AI that moves revenue — not dashboards. Primary languages: **Go** for high-performance systems, **Python** for AI/ML pipelines. Everything ships containerised, observable, and without cloud lock-in.

---

## Stack

| **AI & ML** | **Agentic Frameworks** | **Backend & Infra** |
| :--- | :--- | :--- |
| Ollama · ComfyUI (SDXL) · Fish Speech · LatentSync | LangGraph · LangChain · CrewAI · n8n | Go · FastAPI · Docker |
| Unsloth · LoRA · GGUF (llama.cpp) | Presidio · spaCy | Redis · PostgreSQL · Supabase · Convex |
| FFmpeg · Whisper · SpeechBrain | Playwright · Qdrant · nomic-embed | Nginx · CI/CD |

---

## Featured Builds

### ComfyUI AI Product Photography — Studio Relighting from One Product Photo

*Client uploads a real product photo. SDXL img2img re-lights it across 6 studio presets. Consistent seed = consistent brand identity across a batch.*

Built for Shopify and TikTok Shop brands that need consistent output at volume.

```
Client uploads real product photo
  │
  ▼
ImageResize+ (1024px, proportion-safe, multiple_of: 8)
  │
  ▼
VAEEncode → KSampler (SDXL img2img, denoise: 0.35–0.72)
  │           6 studio presets: Clean White · Dark & Moody · Warm Marble
  │           Natural Wood · Hero Shot · Backlit Glow
  ▼
VAEDecode → ImageSharpen → SaveImage
  │
  ▼
FastAPI web UI — upload, preset select, batch output grid
```

**Key engineering decisions:**
- img2img (not text-to-image) — product identity preserved, lighting and background transformed
- Per-preset denoise values — lower = more product fidelity; higher = more dramatic transformation
- Locked seed per client brief — consistent brand identity across a full batch

**Stack:** Python · ComfyUI · SDXL · FastAPI · img2img

**Pricing:** Starter $49 (10 images) · Growth $120 (30 images) · Retainer $250/mo

---

### Local UGC Video Pipeline — Zero Marginal Cost Per Video

*Full-stack local AI video generation: LLM script → voice clone → lip-sync → branded MP4. No ElevenLabs. No HeyGen. No API fees.*

Built for SMBs producing high-volume social content. Entire pipeline runs on a single consumer GPU.

```
Topic brief
  │
  ▼
Ollama (qwen3.5:9b)          — script generation, local inference
  │
  ▼
Fish Speech :8080             — voice cloning from 20s reference clip
  │                              chunked synthesis ≤200 chars/req,
  │                              temp=0.5 (validated optimal for consistency)
  ▼
LatentSync (RTX 5060 Ti)     — diffusion-based lip-sync (~6.5 GB VRAM)
  │                              sdbds/LatentSync-for-Windows fork,
  │                              torch 2.11.0+cu128 (Blackwell sm_120)
  ▼
ffmpeg postprocess            — scale/pad 9:16, caption burn-in (libass),
  │                              watermark overlay, CRF tuning
  ▼
SyncNet QA gate              — automated lip-sync confidence check,
                               non-blocking (warn + continue on low score)
```

**Key engineering decisions:**
- Fish Speech chunked at ≤200 chars/request — prevents truncation artifacts; chunks concatenated via ffmpeg with tuned silence gaps
- Whisper `tiny.pt` reused from LatentSync checkpoints for SRT generation — no extra model download
- `_ensure_ffmpeg()` pattern — injects bundled ffmpeg into subprocess PATH, no system-level install required
- Client profiles (`clients/{id}/profile.json`) — `--client id` auto-fills face video, voice ref, watermark, tone, steps

**Stack:** Python · Ollama · Fish Speech · LatentSync · ffmpeg · Whisper · SpeechBrain

---

### Lead Qualification Pipeline — Directory-to-CRM Prospect Flow

*LangGraph agent scrapes business listings, scores each prospect against a weighted ICP rubric via local LLM, and routes qualified leads to a self-hosted CRM.*

First run: 60 scraped → 38 qualified → 5 hot leads. Architecture ports to any directory-sourced sales motion.

```
Playwright scraper (port 8600)
  │  Business listings — Google Maps + directory sources
  ▼
LangGraph StateGraph
  ├── Scrape node   — multi-source business directory extraction
  ├── Qualify node  — Ollama qwen3.5:9b against weighted ICP rubric
  └── Route node    — hot (score ≥ 8) → immediate handoff · warm → queue
  │
  ▼
Convex (self-hosted, port 3210) — leads table
```

**Stack:** Python · LangGraph · Playwright · Ollama · Convex · FastAPI

---

### Porsync AI Agent — Config-Driven Multi-Client Agent Service

*One container, many clients. A LangGraph-based AI agent service where adding a new client = dropping a config file, no code changes. Powers the agent surface on [porsync.com](https://www.porsync.com) and white-labelled deployments.*

```
POST /v1/{client_id}/chat
  │
  ▼
Agent Factory              — loads client config.yaml + system_prompt.md
  │                           at runtime; hot-reload via POST /v1/.../reload
  ▼
LangGraph ReAct loop       — recursion_limit = max(75, max_iterations × 4)
  │
  ├── Calendly tool        — booking, event creation
  ├── Supabase (generic)   — CRUD across any table, schema-safe
  ├── Web search (ddgs)    — DuckDuckGo, ddgs package (not duckduckgo_search)
  ├── n8n trigger          — fire any workflow by name
  ├── Brevo email          — transactional send
  ├── Telegram notify      — async alerts
  └── Memory tool          — semantic recall via Qdrant
  │
  ▼
Qdrant (semantic memory)   — nomic-embed-text embeddings, per-session store
Redis (session history)    — 20-turn sliding window, 2h TTL
```

**Key engineering decisions:**
- Self-registering tool modules (`@register` decorator) — adding a tool = one new file in `tools/`, zero changes to factory or registry
- `clients/` volume-mounted, not baked into Docker image — config changes take effect without rebuild
- Web search: `ddgs` package (v6 renamed; `duckduckgo_search` returns empty on v6+)
- 5 hardening patterns: plan-before-execute, output validation + retries, trace, loop detection, verbosity control

**Stack:** Python · FastAPI · LangGraph · LangChain · Qdrant · Redis · Ollama · Docker

---

### Podcast Specialist — End-to-End LLM Fine-Tuning Pipeline

*5-step pipeline from raw YouTube audio to a locally-deployed GGUF model that mimics a specific podcaster's voice and style.*

```
YouTube channel
  │  yt-dlp + youtube-transcript-api
  ▼
Raw transcripts (68K chars, 2 podcasts)
  │  prepare_dataset.py → JSONL Q&A pairs
  ▼
291 training examples
  │  Unsloth LoRA fine-tuning (Qwen3.5-4B, bf16)
  ▼
LoRA adapter (~85 MB)         — 3 epochs, ~9 min on RTX 5060 Ti
  │                              eval loss: 3.54 → 1.69
  ▼
Merge + convert to GGUF       — llama.cpp b8967 (CPU build)
  │                              F16 → q4_K_M quantization (7.9 GB → 2.6 GB)
  ▼
Ollama: podcast-specialist    — live inference
  │
  ▼
FastAPI Chat UI :8660          — streaming, 5 demo chips, SSE events
```

**Key engineering decisions:**
- Qwen3.5-4B tokenizer hash differs from 9B-Instruct — patched `convert_hf_to_gguf.py` with correct hash
- Thinking mode causes infinite loops on Qwen3.5 — disabled via Modelfile template pre-fill + stop tokens
- `collect_channel.py` bulk scraper — pull entire YouTube channel; dataset expandable to 20-30+ episodes

**Stack:** Python · Unsloth · LoRA · llama.cpp · Ollama · FastAPI · Qwen3.5-4B

---

## Other Builds

| Project | Description | Stack | Status | Link |
| :--- | :--- | :--- | :--- | :--- |
| **n8n Lead Engine** | Autonomous B2B/RE/Reddit lead gen pipelines — scrape → qualify → Supabase. Daily cadence, runs unattended | n8n · Playwright · FastAPI · Supabase · Ollama | Live | — |
| **LangGraph Lead Intelligence Agent** | Benchmarked against CrewAI (1,178s) and baseline (639s). Result: 5/5 leads in 128.5s with 4 tool calls | LangGraph · FastAPI · Playwright · Supabase | Built | — |
| **Reddit Prospect Monitor** | 6 subreddits · commercial intent signals · AI pain point extraction → personalised outreach drafts in Supabase daily | Reddit API · n8n · Supabase | Live | — |
| **AI Appointment Booking Agent** | CrewAI crew (Lead Scout, Qualifier, Booking Drafter) qualifies inbound leads and generates personalised booking messages. ~60s end-to-end | CrewAI · LangGraph · Calendly | Built | — |
| **GEO / AI Citation Auditing** | Playwright query runner: 30 prompts across Perplexity + Google AI Overviews → structured gap report | Playwright · Python · Perplexity | Research | — |
| **[Filo — AI Voice Receptionist](https://github.com/icporcincula/filo-pinoy-ai-receptionist)** | Fully local voice receptionist: browser mic → VAD → Faster-Whisper STT → Ollama LLM → Kokoro TTS. No cloud APIs | Go · Faster-Whisper · Ollama · Kokoro · Redis · Docker | Built | [GitHub](https://github.com/icporcincula/filo-pinoy-ai-receptionist) |
| **[Sentinel-Extract](https://github.com/icporcincula/ai-document-analyzer)** | Air-gapped PII detection & document intelligence. Hybrid NER (Presidio + spaCy), local LLM reasoning, OCR-native ingestion | Python · Presidio · Ollama · FastAPI · Tesseract | Built | [GitHub](https://github.com/icporcincula/ai-document-analyzer) |
| **[Vela](https://github.com/icporcincula/vela-pii-compliance)** | Compliance & audit layer — per-tenant rules, immutable audit log, live rule editor | Go · React · SQLite · Docker | Built | [GitHub](https://github.com/icporcincula/vela-pii-compliance) |
| **[Portfolio Recorder](https://github.com/icporcincula/portfolio-recorder)** | Playwright-based automated portfolio video pipeline: screen capture → narration → voice clone → ffmpeg render | Python · Playwright · Fish Speech · ffmpeg | Built | [GitHub](https://github.com/icporcincula/portfolio-recorder) |

---

## Background

5 YOE across agentic systems, local ML infrastructure, and AI-driven automation. Roots in enterprise Java (Spring Boot) and SQL optimisation, then cloud-native Go/Python backends, now focused on end-to-end AI pipelines — from LLM fine-tuning and voice cloning to multi-agent orchestration, agentic web (WebMCP), and autonomous lead generation systems.

Everything ships on private infrastructure: no cloud lock-in, no third-party data exposure, zero marginal cost at scale.

**Working with Porsync:** [porsync.com/about](https://www.porsync.com/about) · [Services](https://www.porsync.com/services) · [Case Studies](https://www.porsync.com/case-studies)

---

📫 [connect@porsync.com](mailto:connect@porsync.com)
