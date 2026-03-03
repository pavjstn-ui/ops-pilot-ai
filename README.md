# OpsPilot AI

OpsPilot AI is a single-tenant knowledge assistant for internal operations docs. It combines PDF ingestion, vector indexing, and citation-backed answers over a local FastAPI service.

The project supports deterministic document/chunk identifiers during ingestion and a configurable embedding backend for retrieval.

## Status

Active prototype for local/dev deployments.

## Features

- FastAPI service with `/health`, `/ingest`, and `/query` endpoints
- PDF-only ingestion pipeline with chunking and Chroma indexing
- Citation-backed query responses with returned source chunks
- Configurable embedding provider (`ollama`, `openai`, `local`, `chroma`)
- Simple web UI served from `/ui/index.html`
- Deterministic IDs for documents and chunks during indexing

## Architecture

```text
PDF -> /ingest -> chunker + embedder -> Chroma
Query -> /query -> retriever(top-k) -> LLM answer + sources
```

## Basic Structure

- `app/`: API routes, RAG components, settings, and UI
- `tools/`: memory bus and pipeline helper scripts
- `scripts/`: end-to-end demo script
- `data/`: index/registry and local vector data
- `memory_bus/`: local event/state/report artifacts

## Quickstart

### Prerequisites

- Python 3.10+
- `pip`
- Optional: Ollama running locally if `EMBEDDING_PROVIDER=ollama`
- Optional: OpenAI API key if `EMBEDDING_PROVIDER=openai` or using `/query`

### Setup

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Run

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

Open `http://127.0.0.1:8000/ui/index.html`.

### Ingestion and Query

```bash
curl -X POST http://127.0.0.1:8000/ingest \
  -F "file=@/absolute/path/to/document.pdf"

curl -X POST http://127.0.0.1:8000/query \
  -H 'Content-Type: application/json' \
  -d '{"query":"Summarize key controls.","top_k":5}'
```

### Demo Script

```bash
python scripts/demo_test.py /absolute/path/to/document.pdf
```

## Configuration

Set these in `.env` (see `.env.example`):

- `EMBEDDING_PROVIDER`
- `OPENAI_API_KEY`
- `OPENAI_EMBEDDING_MODEL`
- `OPENAI_CHAT_MODEL`
- `OLLAMA_BASE_URL`
- `OLLAMA_EMBEDDING_MODEL`
- `CHROMA_COLLECTION`
- `CHUNK_SIZE`
- `CHUNK_OVERLAP`
- `EMBED_BATCH_SIZE`
- `REGISTRY_PATH`

## Tests and Lint

No dedicated test/lint configuration is currently defined. For a basic integration check, use `scripts/demo_test.py` against a running local API.

## Roadmap

- Add automated tests for API routes and retrieval quality
- Add deployment profile for reproducible single-tenant hosting
- Add evaluation harness for citation and answer quality

## License

MIT License. See `LICENSE`.
