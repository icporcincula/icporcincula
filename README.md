# Hi, I'm Bart Porcincula 👋

**Senior AI Engineer · Agentic Systems & Local ML Infrastructure** &nbsp;📍 *Manila, Philippines*

I build production AI systems end-to-end — from fine-tuning LLMs to deploying multi-agent pipelines entirely on private hardware. Not API wrappers: **full-stack AI engineering** across agentic orchestration, local inference, media generation, and autonomous automation infrastructure.

Primary languages: **Go** for high-performance systems, **Python** for AI/ML pipelines. Everything ships containerized, observable, and without cloud lock-in.

---

## 🛠 Tech Stack

| **AI & ML** | **Agentic Frameworks** | **Backend & Infra** |
| :--- | :--- | :--- |
| ![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white) **Ollama · Fish Speech · LatentSync** | ![LangGraph](https://img.shields.io/badge/LangGraph-1C3C3C?style=flat-square) **LangGraph · LangChain** | ![Go](https://img.shields.io/badge/Go-00ADD8?style=flat-square&logo=go&logoColor=white) **Go (Golang)** |
| ![HuggingFace](https://img.shields.io/badge/HuggingFace-FFD21E?style=flat-square&logo=huggingface&logoColor=black) **Unsloth · LoRA · GGUF** | ![CrewAI](https://img.shields.io/badge/CrewAI-FF6B6B?style=flat-square) **CrewAI · n8n** | ![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=flat-square&logo=fastapi&logoColor=white) **FastAPI** |
| ![Qdrant](https://img.shields.io/badge/Qdrant-DC143C?style=flat-square) **Qdrant · nomic-embed** | ![Presidio](https://img.shields.io/badge/Presidio%20%2F%20spaCy-412991?style=flat-square) **Presidio · spaCy** | ![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white) **Redis · PostgreSQL · SQLite** |
| ![FFmpeg](https://img.shields.io/badge/FFmpeg-007808?style=flat-square&logo=ffmpeg&logoColor=white) **FFmpeg · Librosa · Whisper** | | ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white) **Docker · Nginx · CI/CD** |

---

## Featured Projects

### Local Talking-Head UGC Pipeline — Zero Marginal Cost Per Video

*Full-stack local AI video generation: LLM script → voice clone → lip-sync → branded MP4. No ElevenLabs. No HeyGen. No API fees.*

Built for PH SMBs that produce high-volume social content (real estate agents, insurance, Shopee sellers) and can't afford $0.10–$0.30/video at scale. The entire pipeline runs on a single consumer GPU.

```
Topic brief
  │
  ▼
Ollama (qwen3.5:9b)          — script generation, local inference
  │
  ▼
Fish Speech :8080             — voice cloning from 20s reference clip
  │                              chunked synthesis ≤200 chars/req,
  │                              temp=0.9 (89.2/100 eval score)
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
- Fish Speech chunked at ≤200 chars/request — prevents truncation artifacts; chunks concatenated via ffmpeg with tuned silence gaps (350ms/650ms)
- Whisper `tiny.pt` reused from LatentSync checkpoints for SRT generation — no extra model download
- `_ensure_ffmpeg()` pattern — injects bundled ffmpeg into subprocess PATH, no system-level install required
- Client profiles (`clients/{id}/profile.json`) — `--client bart` auto-fills face video, voice ref, watermark, tone, steps; one command for fully personalized output
- Batch mode + auto SRT + QA gate all non-blocking — pipeline continues even if optional steps fail

**Voice Clone Eval — automated scoring pipeline:**
Separate eval harness runs WER (Whisper), speaker similarity (SpeechBrain ECAPA), boundary artifact detection, and prosody scoring. Iterate sweep across temperatures (0.5/0.7/0.9) automated — identified temp=0.9 as optimal without manual listening.

**Stack:** Python · Ollama · Fish Speech · LatentSync · ffmpeg · Whisper · SpeechBrain

---

### Podcast Specialist AI — End-to-End LLM Fine-Tuning Pipeline

*5-step pipeline from raw YouTube audio to a locally-deployed GGUF model that mimics a specific podcaster's voice and style.*

Built to demonstrate that small fine-tuned models outperform prompt-engineered large models for domain-specific content generation — and that the full RLHF-lite pipeline is reproducible on consumer hardware.

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
Ollama: podcast-specialist    — live inference, Taglish style confirmed
  │
  ▼
FastAPI Chat UI :8660          — streaming, 5 demo chips, SSE events
```

**Key engineering decisions:**
- Qwen3.5-4B tokenizer hash differs from 9B-Instruct — patched `convert_hf_to_gguf.py` with correct hash; tokenizer restored from HF cache before conversion
- Thinking mode causes infinite `<think>` loops on Qwen3.5 — disabled via Modelfile template pre-fill + stop tokens
- `collect_channel.py` bulk scraper — pull entire YouTube channel via yt-dlp; dataset ready to expand to 20–30 more episodes

**Stack:** Python · Unsloth · LoRA · llama.cpp · Ollama · FastAPI · Qwen3.5-4B

---

### Porsynth AI Agent — Config-Driven Multi-Client Agent Service

*One container, many clients. A LangGraph-based AI agent service where adding a new client = dropping a config file, no code changes.*

Built to replace a rigid bespoke agent with a reusable, hot-reloadable agentic layer that any client configuration can plug into.

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
  ├── Carousell search     — PH listings lookup
  └── Memory tool          — semantic recall via Qdrant
  │
  ▼
Qdrant (semantic memory)   — nomic-embed-text embeddings, per-session store
Redis (session history)    — 20-turn sliding window, 2h TTL
```

**Key engineering decisions:**
- Self-registering tool modules (`@register` decorator) — adding a tool = one new file in `tools/`, zero changes to factory or registry
- `clients/` directory volume-mounted, not baked into Docker image — config changes take effect without rebuild
- Web search: `ddgs` package (v6 renamed; `duckduckgo_search` returns empty on v6+)
- LangGraph answer capture: `on_chain_end` only — stream buffer garbles mid-reasoning chunks
- Memory: Qdrant (semantic) + Redis (sessions) + Ollama embeddings; LM Studio embedding endpoint rejected LangChain SDK

**Stack:** Python · FastAPI · LangGraph · LangChain · Qdrant · Redis · Ollama · Docker

---

### [Filo — Local AI Voice Receptionist for Filipino SMEs](https://github.com/icporcincula/filo-pinoy-ai-receptionist)

*A fully local, privacy-first voice receptionist. No cloud APIs. No data leaving your machine.*

Built for Philippine SMEs that need an always-on front desk without subscription fees or data exposure. Filo handles real-time voice conversations end-to-end: browser microphone → VAD → STT → LLM → TTS → browser audio, all on your own hardware.

```
Browser mic (16kHz PCM)
  │  WebSocket /ws
  ▼
Go server :8080
  ├── VAD        energy-based, 1200ms silence = end of utterance
  ├── STT   →   Speaches :8000   (Faster-Whisper, CPU or GPU)
  ├── LLM   →   Ollama   :11434  (llama3.1:8b or any local model)
  ├── TTS   →   Kokoro   :5000   (kokoro-onnx)
  └── History → Redis    :6379   (2h TTL, 20-turn window)
```

**Key engineering decisions:**
- Echo cancellation + mic muting during TTS playback to prevent false VAD triggers
- WebSocket keep-alive (20s ping/pong) to survive long silences between utterances
- `WHISPER__MODEL_TTL: 0` keeps the STT model hot — eliminates 4–5s cold starts
- Conversation history windowed at 20 turns with 2h Redis TTL per session

**Stack:** Go · Faster-Whisper · Ollama · Kokoro ONNX · Redis · Docker

---

## 💼 Background

Senior AI Engineer (5 YOE) specializing in agentic systems, local ML infrastructure, and AI-driven automation. Roots in enterprise Java (Spring Boot) and SQL optimization, then cloud-native Go/Python backends, now focused on **end-to-end AI pipelines** — from LLM fine-tuning and voice cloning to multi-agent orchestration and autonomous lead generation systems.

Everything I ship runs on private infrastructure: no cloud lock-in, no third-party data exposure, zero marginal cost at scale.

---

## Other Projects

| Project | Description | Stack | Link |
| :--- | :--- | :--- | :--- |
| **n8n Lead Engine** | Autonomous B2B/RE/Reddit lead generation pipelines — scrape → qualify → Supabase. Orchestrator/subworkflow architecture, 5 active pipelines | n8n · Playwright · FastAPI · Supabase · Ollama | — |
| **Voice Clone Eval** | Automated scoring harness for Fish Speech output: WER, speaker similarity, boundary artifact detection, prosody. Iterate sweep across hyperparams without manual listening | Python · Whisper · SpeechBrain · Librosa | — |
| **CrewAI Demos** | 5 multi-agent crews (RE Benchmark, Job Eval, Recruiter, AI Booking, Survey Report) with structured SSE streaming UI | Python · CrewAI · FastAPI | — |
| **House Tour Pipeline** | Automated real estate video generation: property photos → AI description → script → TTS → Ken Burns + DepthFlow parallax video | Python · DepthFlow · ffmpeg · Fish Speech · Ollama | — |
| **Portfolio Recorder** | Playwright-based automated portfolio video pipeline: screen capture → narration → voice clone → ffmpeg render | Python · Playwright · Fish Speech · ffmpeg | [GitHub](https://github.com/icporcincula/portfolio-recorder) |
| **[Sentinel-Extract](https://github.com/icporcincula/ai-document-analyzer)** | Air-gapped PII detection & document intelligence. Hybrid NER (Presidio + spaCy), local LLM reasoning, OCR-native ingestion | Python · Presidio · Ollama · FastAPI · Tesseract | [GitHub](https://github.com/icporcincula/ai-document-analyzer) |
| **[Vela](https://github.com/icporcincula/vela-pii-compliance)** | Compliance & audit layer on top of Eidolon (Rust PII proxy) — per-tenant rules, immutable audit log, live rule editor | Go · React · SQLite · Docker | [GitHub](https://github.com/icporcincula/vela-pii-compliance) |

---

📫 [LinkedIn](https://www.linkedin.com/in/porcinculabart/) · [Email](mailto:porcincula.developer@gmail.com)
