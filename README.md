# OpsPilot AI

OpsPilot AI is a FastAPI-based RAG service for ingesting PDF documents into ChromaDB and answering semantic queries with source citations. It is designed for local, single-tenant usage with configurable embedding providers.

## What It Does

- Serves a FastAPI API with health, ingestion, and semantic query endpoints
- Ingests PDF files, splits them into chunks, generates embeddings, and stores vectors in ChromaDB
- Persists ingestion metadata to a registry (`data/index/doc_registry.jsonl`)
- Retrieves relevant chunks for a query and returns an answer with source excerpts
- Serves a lightweight UI at `/ui/index.html`

## Tech Stack

- Python
- FastAPI
- ChromaDB
- OpenAI SDK (chat + embeddings)
- FastEmbed (local embeddings option)
- Optional Ollama embeddings provider
- pypdf for PDF parsing

## Quick Start

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

API root redirects to UI: `http://127.0.0.1:8000/`

## API Endpoints

### `GET /health`

Health check.

Example response:

```json
{"status": "ok"}
```

### `POST /ingest`

Uploads and ingests a PDF file (multipart form-data).

Request:

- Content-Type: `multipart/form-data`
- Field: `file` (PDF only)

Example:

```bash
curl -X POST "http://127.0.0.1:8000/ingest" \
  -F "file=@/absolute/path/to/document.pdf"
```

Successful response (`201`):

```json
{
  "doc_id": "...",
  "filename": "document.pdf",
  "page_count": 10,
  "chunk_count": 42,
  "chunk_ids": ["..."],
  "collection": "ops_pilot_docs",
  "registry_path": "data/index/doc_registry.jsonl"
}
```

### `POST /query`

Runs semantic retrieval + answer generation.

Request JSON:

- `query` (string, required)
- `top_k` (integer, optional, 1-20, default 5)

Example:

```bash
curl -X POST "http://127.0.0.1:8000/query" \
  -H "Content-Type: application/json" \
  -d '{"query":"Summarize the key controls.","top_k":5}'
```

Successful response (`200`):

```json
{
  "answer": "...",
  "sources": [
    {
      "doc_id": "...",
      "chunk_id": "...",
      "source": "document.pdf#page1",
      "text": "..."
    }
  ]
}
```

## Project Status

Active development.

## Basic Project Layout

- `app/` - FastAPI app, API routes, RAG components, UI
- `data/` - Chroma data and ingestion registry
- `docker/` - container files
- `scripts/` - utility scripts (including demo/test helper)
- `tools/` - project tooling helpers
- `memory_bus/` - local memory bus utilities and state files
- `requirements.txt` - Python dependencies

## License

This project is licensed under the terms in [LICENSE](LICENSE).
