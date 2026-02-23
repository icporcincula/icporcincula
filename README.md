# Hi, I’m Immanuel Bart Porcincula 👋

**Senior Systems Engineer | AI & Audio Architecture** 📍 *Manila, Philippines*

I engineer robust backend systems and privacy-first AI infrastructure. My focus is on moving beyond "wrapping APIs" to building **event-driven microservices**, **local RAG pipelines**, and **programmatic media processing engines**.

I specialize in **Golang** for high-performance systems and **Python** for AI/Audio workflows, deploying everything as isolated, containerized services.

---

## 🛠 Tech Stack & Architecture

| **Backend Systems** | **AI & Data Engineering** | **Infrastructure & DevOps** |
| :--- | :--- | :--- |
| ![Go](https://img.shields.io/badge/Go-00ADD8?style=flat-square&logo=go&logoColor=white) **Go (Golang)** | ![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white) **Librosa / FFmpeg** | ![Docker](https://img.shields.io/badge/Podman%20%2F%20Docker-2496ED?style=flat-square&logo=docker&logoColor=white) **Podman / Docker** |
| ![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=flat-square&logo=fastapi&logoColor=white) **FastAPI** | ![OpenAI](https://img.shields.io/badge/Local%20LLMs-412991?style=flat-square&logo=openai&logoColor=white) **LM Studio / Ollama** | ![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white) **Redis (Pub/Sub)** |
| ![Postgres](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white) **PostgreSQL** | ![LangChain](https://img.shields.io/badge/RAG%20%26%20Vectors-1C3C3C?style=flat-square) **Qdrant / LangChain** | ![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat-square&logo=linux&logoColor=black) **Nginx / CI/CD** |

---

## Featured Engineering Projects

### [**AI Agency (Private Project)**](#)

A fully local AI software agency that builds complete projects from a Discord conversation. Describe what you want, approve the story backlog, and watch it code story by story — no cloud APIs, runs entirely on Ollama.

* **Stack:** Python · Ollama · Redis · ChromaDB · Discord

### [**Sentinel-Extract: Air-Gapped PII & Document Intelligence**](#)
*A secure, localized pipeline that de-identifies sensitive data before performing structured AI extraction.*

* **The Architecture:** A "Zero-Trust" data flow where documents are sanitized at the edge. The system detects PII on the host and feeds only anonymized tokens to a local LLM, ensuring 100% data residency within the private network.

* **Key Tech:**
   * **Privacy-First De-Identification:** Integrated Microsoft Presidio to detect 12+ entity types (Names, IBANs, SSNs) using a hybrid of Spacy NER and logic-based recognizers.

   * **Local Reasoning Engine:** Leverages Ollama (DeepSeek-R1 / Gemma 3) via an OpenAI-compatible bridge, providing high-reasoning extraction without cloud exposure.

   * **Robust Document Ingestion:** Built with FastAPI and Tesseract OCR, allowing the system to process both digital and scanned PDFs with equal precision.

   * **Docker-Bridged Connectivity:** Specialized container networking to bridge isolated microservices with host-level GPU/LLM resources.

* **Stack:** Python 3.11, FastAPI, Microsoft Presidio, Ollama, Tesseract, Docker.

### [**Vault-RAG: Air-Gapped Document Analysis**](#)
*A privacy-first document chat system designed for sensitive data environments.*

* **The Architecture:** A fully local Retrieval-Augmented Generation (RAG) pipeline. No data leaves the container network.
* **Key Tech:**
    * **Vector Search:** Self-hosted **Qdrant** instance for high-speed semantic retrieval.
    * **Local Embeddings:** Uses `sentence-transformers` (all-MiniLM-L6-v2) running on CPU.
    * **Interface:** Streamlit frontend tunneled securely via Ngrok for client demos.
* **Stack:** LangChain, Qdrant, Streamlit, Docker Compose.

### [**Microservices Audio Pipeline**](#)
*A scalable, event-driven engine that transforms raw vocal "mumbles" into structured lyrics.*

* **The Architecture:** Deconstructed a monolithic task into 5 isolated microservices (Orchestrator, STT, Analysis, LLM, API).
* **Key Tech:**
    * **Event Bus:** Redis Pub/Sub for asynchronous communication between workers.
    * **Audio Engineering:** Custom **Librosa** implementation to detect BPM/Key and **FFmpeg** for signal processing.
    * **Hybrid AI:** Runs **Whisper (Int8)** for transcription and connects to local LLMs for creative lyric generation.
* **Stack:** Python 3.10, FastAPI, Celery, Podman.

---

## 💼 Professional Background
* **Senior Software Engineer (4.5 YOE):** Specializing in legacy migration and backend optimization.
* **Legacy Experience:** Deep roots in enterprise Java (Spring Boot) and SQL optimization before transitioning to modern Cloud Native stacks.

---

### 📫 Connect
* **Focus for 2026:** High-concurrency Go systems & "Black Box" AI automation.
* [LinkedIn](https://www.linkedin.com/in/porcinculabart/) • [Email](porcincula.developer@gmail.com)
