# Hi, I'm Bart Porcincula 👋

**Senior Systems Engineer · AI & Audio Architecture** &nbsp;📍 *Manila, Philippines*

I build backend systems and privacy-first AI infrastructure — not wrappers around APIs, but **event-driven microservices**, **local RAG pipelines**, and **programmatic media processing engines** designed to run entirely on private infrastructure.

My primary languages are **Go** for high-performance systems and **Python** for AI/audio workflows. Everything ships as isolated, containerized services.

---

## 🛠 Tech Stack

| **Backend Systems** | **AI & Data Engineering** | **Infrastructure** |
| :--- | :--- | :--- |
| ![Go](https://img.shields.io/badge/Go-00ADD8?style=flat-square&logo=go&logoColor=white) **Go (Golang)** | ![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white) **Librosa / FFmpeg / Whisper** | ![Docker](https://img.shields.io/badge/Docker%20%2F%20Podman-2496ED?style=flat-square&logo=docker&logoColor=white) **Docker / Podman** |
| ![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=flat-square&logo=fastapi&logoColor=white) **FastAPI** | ![Presidio](https://img.shields.io/badge/Presidio%20%2F%20spaCy-412991?style=flat-square) **Presidio · spaCy · xlm-roberta** | ![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white) **Redis (Pub/Sub)** |
| ![Postgres](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white) **PostgreSQL · SQLite** | ![LLMs](https://img.shields.io/badge/Local%20LLMs-1C3C3C?style=flat-square) **Ollama · Qdrant · LangChain** | ![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat-square&logo=linux&logoColor=black) **Nginx · CI/CD · Docker Compose** |

---

## Featured Projects

### [Filo — Local AI Voice Receptionist for Filipino SMEs](#)

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

**Roadmap:** Qdrant RAG over business docs (FAQs, pricing), intent classification, n8n → Google Calendar booking, and a WhatsApp channel via Meta Cloud API.

**Stack:** Go · Faster-Whisper · Ollama · Kokoro ONNX · Redis · Docker

---

### [Sentinel-Extract — Air-Gapped PII Detection & Document Intelligence](#)

*A zero-trust document pipeline that de-identifies sensitive data before it ever reaches an LLM.*

The architecture treats every document as untrusted at ingestion. PII is detected and replaced with tokens on the host; only anonymized text is passed to the local reasoning engine. Structured extraction results are then re-hydrated with original values — the LLM never sees real names, IBANs, or SSNs.

**Key engineering decisions:**
- **Hybrid NER:** Microsoft Presidio with spaCy models + logic-based recognizers for 12+ entity types (Names, IBANs, SSNs, MRNs). Pattern rules cover edge cases that pure ML misses.
- **Local reasoning:** Ollama (DeepSeek-R1 / Gemma 3) via OpenAI-compatible bridge — high-reasoning extraction with 100% data residency.
- **OCR-native ingestion:** FastAPI + Tesseract handles both digital and scanned PDFs uniformly.
- **Bridged container networking:** Specialized Docker networking lets isolated microservices reach host-level GPU/LLM resources without exposing the host broadly.

**Stack:** Python 3.11 · FastAPI · Microsoft Presidio · Ollama · Tesseract · Docker

---

### [Vela — Compliance & Audit Layer for LLM Proxy Traffic](#)

*A multi-tenant compliance layer built on top of [Eidolon](https://github.com/0M3REXE/eidolon), the open-source Rust PII redaction proxy.*

Eidolon strips PII from LLM-bound traffic, but has no memory of what it redacted, for whom, or when. Vela adds the compliance infrastructure that makes it actually auditable and operable at scale — per-tenant rule isolation, a full immutable audit log per request, and a live rule editor — all deployable with `docker compose up`.

```
Your App → Eidolon Proxy (Rust :8000) → OpenAI / Anthropic / Google
                     ↓  audit webhook
              Vela API (Go :8080)
                     ↑
           Dashboard UI (React, served at :8080)
```

| Feature | Eidolon (upstream) | Vela |
|---|---|---|
| Audit trail | None | Full per-request log per tenant |
| Multi-tenancy | No | Isolated rules + API keys per client |
| Rule management | Single config file | Live editor + regex tester |
| Compliance export | None | Export audit logs + rules as TOML/JSON |
| Config | YAML/env | Dashboard UI |

**Stack:** Go · React · SQLite · Docker Compose · Eidolon (Rust, upstream)

---

## 💼 Background

Senior Software Engineer (4.5 YOE) specializing in backend systems, legacy migration, and AI-adjacent infrastructure. Roots in enterprise Java (Spring Boot) and SQL optimization before moving to cloud-native Go/Python stacks. Currently focused on **high-concurrency Go systems** and **"black box" AI automation** that runs entirely on private infrastructure.

---

📫 [LinkedIn](https://www.linkedin.com/in/porcinculabart/) · [Email](mailto:porcincula.developer@gmail.com)
