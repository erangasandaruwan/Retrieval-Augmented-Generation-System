# Retrieval-Augmented Generation (RAG) Solution Overview

The complete enterprise-grade **Retrieval-Augmented Generation (RAG)** solution has been generated based on the architecture described in the repository.

---

## 🏗️ Architecture & Component Breakdown

### 1. Ingestion & Chunking Layer
- **Multi-Format Document Parsing** (`src/ingestion/parser.py`): Structured parsing for PDF, Markdown, HTML, and plain text with section and page-level metadata extraction.
- **Intelligent Token Chunker** (`src/ingestion/chunker.py`): Target token chunking (~500–800 tokens with 100-token overlap), sentence-boundary preservation, and tracking of `chunk_id`, `source`, `section`, and `page`.
- **Ingestion Pipeline** (`src/ingestion/pipeline.py`): Batch directory and stream ingestion.

### 2. Hybrid Retrieval & Re-ranking Layer
- **Dense Vector Search** (`src/retrieval/vector_store.py`, `src/retrieval/embeddings.py`): Vector indexing with ChromaDB and in-memory fallback, supporting OpenAI, Azure OpenAI, and local SentenceTransformers.
- **Sparse BM25 Search** (`src/retrieval/bm25_search.py`): Okapi BM25 keyword matching for exact IDs, SKUs, and codes (e.g., `VX-VEH-0012398`, contract sections).
- **Hybrid Fusion Engine** (`src/retrieval/hybrid_retriever.py`): Reciprocal Rank Fusion (RRF) and convex combination ($\alpha \cdot \text{Vector} + (1-\alpha) \cdot \text{BM25}$).
- **Cross-Encoder Re-ranker** (`src/retrieval/reranker.py`): Deep transformer cross-attention scoring using `cross-encoder/ms-marco-MiniLM-L-6-v2`.

### 3. Generation & Citation Enforcement
- **LLM Orchestrator** (`src/generation/llm_client.py`, `src/generation/rag_chain.py`): Strict evidence grounding, prompt versioning (`config/prompts/`), and latency tracking.
- **Citation & Refusal Validator** (`src/generation/citation_validator.py`): Validates inline and trailing source citations (`Source: Contract.pdf, Section: 8.2, Page: 14`) and enforces abstention/refusal when context is insufficient.

### 4. Phase 3 Production AI Engineering & CI/CD Quality Gate
- **Golden Evaluation Dataset** (`data/golden_dataset.json`): Verified Q&A pairs covering exact IDs, contracts, policies, and refusal test cases.
- **Evaluation Metrics Engine** (`src/evaluation/metrics.py`, `src/evaluation/evaluator.py`): Computes **Faithfulness**, **Context Precision**, **Context Recall**, **Answer Relevance**, and **Citation Accuracy**.
- **CI/CD Quality Gate Enforcer** (`src/evaluation/quality_gate.py`, `config/eval_thresholds.yaml`): CLI tool (`python -m src.evaluation.quality_gate --strict`) that gates PRs and builds, generating Markdown and JSON regression reports.

### 5. API & Observability
- **FastAPI REST Service** (`src/api/app.py`, `src/api/routes.py`): `/api/v1/query`, `/api/v1/ingest/file`, `/api/v1/ingest/text`, `/api/v1/evaluate`, `/api/v1/health`, and `/api/v1/metrics`.
- **Telemetry & Tracing** (`src/observability/telemetry.py`): OpenTelemetry integration and Prometheus metrics.

### 6. Docker & CI/CD Pipelines
- **Production Dockerfile** (`Dockerfile`): Hardened multi-stage build running under a non-root user.
- **Docker Compose** (`docker-compose.yml`): Complete stack with RAG API, Prometheus, and Jaeger.
- **GitHub Actions CI/CD** (`.github/workflows/ci.yml`, `.github/workflows/cd.yml`): Linting, testing, automated AI Quality Gate SLA evaluation, and Azure Container Apps/AKS deployment.
- **Azure DevOps Pipeline** (`azure-pipelines.yml`): Multi-stage build, test, AI SLA gate, container build, and ACR push.
- **Complete Test Suite** (`tests/`): Unit and integration test coverage for all components.
