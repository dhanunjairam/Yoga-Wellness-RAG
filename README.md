# RAG API (Domain-Agnostic)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115.0-brightgreen.svg)](https://fastapi.tiangolo.com)
[![Python](https://img.shields.io/badge/Python-3.11%2B-blue.svg)](https://python.org)
[![Pinecone](https://img.shields.io/badge/Pinecone-5.1.0-orange.svg)](https://pinecone.io)
[![RAGAS](https://img.shields.io/badge/RAGAS-0.1.9-purple.svg)](https://github.com/explodinggradients/ragas)
[![UV](https://img.shields.io/badge/UV-0.4.18-brightgreen)](https://astral.sh/uv)

A production-ready **Retrieval-Augmented Generation (RAG)** system that works with **any PDF documents** - not limited to any specific domain.

## 🚀 Features
[![FastAPI](https://img.shields.io/badge/-REST_API-blue)](https://fastapi.tiangolo.com)
[![Pinecone](https://img.shields.io/badge/-Vector_DB-orange)](https://pinecone.io)
[![RAGAS](https://img.shields.io/badge/-Eval_Framework-purple)](https://github.com/explodinggradients/ragas)

- **Full RAG Pipeline**: PDF → Markdown → Vector Store → LLM → RAGAS Evaluation
- **Pinecone Vector Search** with hybrid retrieval & MMR diversity
- **Configurable LLM** with strict "context-only" prompting
- **RAGAS Metrics** for retrieval & generation quality (async evaluation)
- **FastAPI** with `/chat` (full pipeline) + `/index-status` endpoints
- **Automatic PDF Processing** - converts only when needed

## 🏗️ Architecture

PDFs → [data_source/] → Markdown Pipeline → [markdown_data_sources/] → Chunking → Pinecone
↓
User Query → Retriever (Hybrid Search + MMR) → Chatbot (Context-Only LLM) → RAGAS Eval

## 📁 Project Structure

├── main.py # FastAPI app + endpoints

├── README.md

├── pyproject.toml # Dependencies

├── env_example.txt # Env vars template

├── uv.lock # UV lockfile

├── src/
│ ├── data/
│ │ ├── markdown_data_pipeline.py # PDF → Markdown conversion
│ │ ├── data_source/ # 📥 INPUT: Raw PDFs here
│ │ └── markdown_data_sources/ # 📤 OUTPUT: Generated .md files
│ ├── data_processing/
│ │ ├── data_chunking_loading.py # Markdown → Pinecone ingestion
│ │ └── check_pincone_index.py # List indexed files
│ ├── data_retriver/
│ │ └── data_pinecone_retriver.py # Hybrid search + MMR + metrics
│ ├── llm/
│ │ └── llm_file.py # Configurable ChatOpenAI + strict prompting
│ └── evaluation/
│ └── ragas_evaluation.py # Async RAGAS metrics (5 core metrics)


## ⚙️ Quick Start with UV

### 1. **Install UV** (if not installed)
```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"


2. Setup Environment
# Clone & enter project
git clone <repo> && cd rag-api
uv sync          # Installs all dependencies from pyproject.toml

# Copy & configure environment
cp env_example.txt .env
# Edit .env with your keys:
# PINECONE_API=your_pinecone_key
# _API_KEY=your_llm_key (Perplexity/OpenAI/etc)
# INDEX_NAME=your_pinecone_index

3. Add Your PDFs

src/data/data_source/
└── your_document_1.pdf
└── your_document_2.pdf
└── any_topic.pdf


4. Run API

bash
# Development (auto-reload)
uvicorn main:app --reload --port 8000

# Production
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4

🌐 API Endpoints
POST /chat

Full RAG Pipeline - Retrieve → Generate → Evaluate

bash
curl -X POST "http://localhost:8000/chat" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "your question here",
    "top_k": 7,
    "score_threshold": 0.7,
    "use_mmr": true
  }'

Sample Response:

json
{
  "success": true,
  "data": {
    "answer": "Answer from your documents... [Source.pdf | Section]",
    "retrieval_metrics": {"precision_at_k": 0.857, "latency_ms": 23.4},
    "ragas_scores": {"context_precision": 0.89, "faithfulness": 0.94},
    "evaluation_summary": {"overall_score": 0.823, "status": "🟢 PASS"},
    "sources": [...],
    "indexed_files": {"doc1.pdf": 45, "doc2.pdf": 32}
  }
}

GET /index-status

Vector Store Health Check

curl http://localhost:8000/index-status

GET /health

API Health Check

curl http://localhost:8000/health

🔧 Environment Variables

Create .env from env_example.txt:

bash
# Required
PINECONE_API=your_pinecone_key
_API_KEY=your_llm_key
INDEX_NAME=your_pinecone_index

# Optional Tuning
EMBEDDING_MODEL=nomic-ai/nomic-embed-text-v1.5
LLM_MODEL=your_model_name
LLM_BASE_URL=your_llm_base_url  # Perplexity/OpenAI/etc
OLLAMA_BASE_URL=http://localhost:11434  # Local embeddings (optional)


📈 Quality Metrics

Retrieval Metrics:

    precision_at_k: Relevant chunks ranked higher

    source_diversity: Multi-file coverage

    latency_ms: End-to-end speed

RAGAS Metrics:

    Context Precision/Recall/Relevancy

    Faithfulness + Answer Relevancy

🔄 Auto Data Pipeline

    On Startup: Auto-checks data_source/*.pdf vs markdown_data_sources/*.md

    Conversion: Only processes new/changed PDFs → Markdown

    Ingestion: Chunks → Pinecone vectors

    Status: /index-status shows indexed files + chunk counts


🌍 Domain Agnostic

Works with any PDF documents:

src/data/data_source/
├── legal_contracts.pdf     → Legal RAG
├── medical_papers.pdf      → Medical RAG
├── technical_docs.pdf      → Technical RAG
└── any_content.pdf         → Any RAG

Just drop PDFs and query!
🛠️ UV Development Workflow

# Clean environment
uv cache clean
uv sync --dev

# Run with auto-reload
uv run uvicorn main:app --reload

# Lint & Format
uv tool install ruff
uv run ruff check . && uv run ruff format .

# Shell with dependencies
uv run -- python  # Opens Python REPL with all deps

# Add new dependency
uv add requests
uv sync