## Repository Guidelines

This document summarizes how to work with the cognee repository: how it's organized, how to build, test, lint, and contribute. It mirrors our actual tooling and CI while providing quick commands for local development.

## Project Overview

Cognee is an open-source AI memory platform that transforms raw data into persistent knowledge graphs for AI agents. It replaces traditional RAG with an ECL (Extract, Cognify, Load) pipeline combining vector search, graph databases, and LLM-powered entity extraction.

**Requirements**: Python >= 3.10 and < 3.14 (recommended: Python 3.11)

## Project Structure & Module Organization

- `cognee/`: Core Python library and API.
  - `api/`: FastAPI application with versioned routes.
    - `v1/`: API v1 endpoints: add, cloud, cognify, config, datasets, delete, exceptions, health, memify, notebooks, ontologies, permissions, prune, responses, search, session, settings, sync, ui, update, users, visualize
  - `cli/`: CLI entry points and subcommands.
    - `commands/`: add, cognify, config, delete, search
  - `infrastructure/`: Databases, LLM providers, embeddings, loaders, and storage adapters.
    - `databases/`: cache, graph, hybrid, relational, unified, vector
    - `llm/`: LLM gateway supporting multiple providers
    - `files/`: Document loaders for various formats
    - `engine/`: Core engine components
    - `context/`: Context management
    - `session/`: Session handling
  - `modules/`: Domain logic.
    - `chunking/`: Text chunking strategies
    - `cloud/`: Cloud-specific functionality
    - `cognify/`: Knowledge graph construction
    - `data/`: Data management
    - `engine/`: Engine module
    - `graph/`: Graph operations
    - `ingestion/`: Data ingestion
    - `memify/`: Graph enrichment
    - `metrics/`: Metrics collection
    - `notebooks/`: Notebook utilities
    - `observability/`: Tracing and monitoring
    - `ontology/`: Ontology support
    - `pipelines/`: Pipeline orchestration
    - `retrieval/`: Retrieval strategies
    - `run_custom_pipeline/`: Custom pipeline execution
    - `search/`: Search functionality
    - `settings/`: Configuration settings
    - `storage/`: Storage management
    - `sync/`: Synchronization
    - `users/`: User management and permissions
    - `visualization/`: Graph visualization
  - `tasks/`: Reusable pipeline tasks.
    - `chunks/`: Chunk processing
    - `cleanup/`: Cleanup operations
    - `codingagents/`: Coding agent tasks
    - `completion/`: Completion tasks
    - `documents/`: Document processing
    - `entity_completion/`: Entity completion
    - `graph/`: Graph extraction and manipulation
    - `ingestion/`: Data ingestion tasks
    - `memify/`: Graph enrichment tasks
    - `schema/`: Schema operations
    - `storage/`: Storage tasks
    - `summarization/`: Summarization
    - `temporal_awareness/`: Temporal processing
    - `temporal_graph/`: Temporal graph tasks
    - `translation/`: Translation tasks
    - `web_scraper/`: Web scraping
  - `eval_framework/`: Evaluation utilities and adapters.
  - `exceptions/`: Custom exceptions.
  - `shared/`: Cross-cutting helpers (logging, settings, utils).
  - `tests/`: Test suite organized by type.
    - `unit/`: Unit tests
    - `integration/`: Integration tests
    - `cli_tests/`: CLI tests
    - `tasks/`: Task-specific tests
    - `test_data/`: Test fixtures
    - `utils/`: Test utilities
    - `subprocesses/`: Subprocess tests
  - `alembic/`: Database migrations.
  - `__main__.py`: Entrypoint routing to CLI.
- `cognee-mcp/`: Model Context Protocol server (v0.5.3) exposing cognee as MCP tools (SSE/HTTP/stdio).
  - `src/`: Server implementation with codingagents support
  - Entry points: `cognee` and `cognee-mcp`
- `cognee-frontend/`: Next.js 16 UI with React 19 and Tailwind CSS 4 for local development and demos.
- `distributed/`: Utilities for distributed execution (Modal, workers, queues).
- `examples/`: Example scripts demonstrating the public APIs.
  - `configurations/`: Configuration examples
  - `custom_pipelines/`: Custom pipeline examples
  - `database_examples/`: Database backend examples (ChromaDB, Kuzu, Neo4j, Neptune, PGVector)
  - `demos/`: Demo scripts (simple examples, multimedia, ontology, temporal awareness, DLT ingestion)
  - `guides/`: Tutorial guides (ontology, search basics, S3 storage, memify, custom models)
  - `pocs/`: Proof of concept implementations
- `notebooks/`: Jupyter notebooks for demos and tutorials.
- `evals/`: Evaluation framework and benchmarks.

Notes:
- Co-locate feature-specific helpers under their respective package (`modules/`, `infrastructure/`, or `tasks/`).
- Extend the system by adding new tasks, loaders, or retrievers rather than modifying core pipeline mechanisms.

## Build, Test, and Development Commands

Python (root) – requires Python >= 3.10 and < 3.14. We recommend `uv` for speed and reproducibility.

### Setup

```bash
# Create virtual environment and install dependencies
uv sync --dev --all-extras --reinstall

# Install with specific extras
uv pip install -e ".[postgres,neo4j,docs,chromadb]"

# Set up pre-commit hooks
pre-commit install
```

### CLI Commands

```bash
uv run cognee-cli add "Cognee turns documents into AI memory."
uv run cognee-cli cognify
uv run cognee-cli search "What does cognee do?"
uv run cognee-cli config                      # Configuration management
uv run cognee-cli delete --all                # Delete data
uv run cognee-cli -ui                        # Launches UI, backend API, and MCP server together
```

### FastAPI Server

```bash
# Start the API server directly
uv run python -m cognee.api.client
```

### Testing

```bash
# Run all tests
uv run pytest

# Run specific test suites (CI mirrors these commands)
uv run pytest cognee/tests/unit/ -v
uv run pytest cognee/tests/integration/ -v
uv run pytest cognee/tests/cli_tests/ -v

# Run with coverage
uv run pytest --cov=cognee --cov-report=html
```

### Code Quality

```bash
# Lint and format (ruff)
uv run ruff check .
uv run ruff format .

# Run both via pre-commit
pre-commit run --all-files

# Type checking (mypy)
uv run mypy cognee/

# Pylint
uv run pylint cognee/
```

### MCP Server (`cognee-mcp/`)

```bash
cd cognee-mcp
uv sync --dev --all-extras --reinstall

# Run with different transports
uv run python src/server.py                        # stdio (default)
uv run python src/server.py --transport sse
uv run python src/server.py --transport http --host 127.0.0.1 --port 8000 --path /mcp

# API Mode (connect to running Cognee API)
uv run python src/server.py --transport sse --api-url http://localhost:8000 --api-token YOUR_TOKEN

# Docker
docker run -e TRANSPORT_MODE=http --env-file ./.env -p 8000:8000 --rm -it cognee/cognee-mcp:main
```

### Frontend (`cognee-frontend/`)

```bash
cd cognee-frontend
npm install
npm run dev       # Next.js dev server
npm run lint      # ESLint
npm run build && npm start
```

## Coding Style & Naming Conventions

Python:
- 4-space indentation, modules and functions in `snake_case`, classes in `PascalCase`.
- Public APIs should be type-annotated where practical.
- Use `ruff format` before committing; `ruff check` enforces import hygiene and style (line-length 100 configured in `pyproject.toml`).
- Use double quotes `"` not single quotes `'` (enforced by ruff-format).
- Prefer explicit, structured error handling. Use shared logging utilities in `cognee.shared.logging_utils`.
- Always run `pre-commit run --all-files` before committing.

MCP server and Frontend:
- Follow the local `README.md` and ESLint/TypeScript configuration in `cognee-frontend/`.

## Testing Guidelines

- Place Python tests under `cognee/tests/`.
  - Unit tests: `cognee/tests/unit/`
  - Integration tests: `cognee/tests/integration/`
  - CLI tests: `cognee/tests/cli_tests/`
  - Task tests: `cognee/tests/tasks/`
- Name test files `test_*.py`. Use `pytest.mark.asyncio` for async tests.
- Avoid external state; rely on test fixtures and the CI-provided env vars when LLM/embedding providers are required.
- When adding public APIs, provide/update targeted examples under `examples/`.

## Branching Strategy

**IMPORTANT**: Always branch from `dev`, not `main`. The `dev` branch is the active development branch.

```bash
git checkout dev
git pull origin dev
git checkout -b feature/your-feature-name
```

## Commit & Pull Request Guidelines

- Use clear, imperative subjects (≤ 72 chars) and conventional commit styling in PR titles. Our CI validates semantic PR titles (see `.github/workflows/pr_lint`). Examples:
  - `feat(graph): add temporal edge weighting`
  - `fix(api): handle missing auth cookie`
  - `docs: update installation instructions`
- Reference related issues/discussions in the PR body and provide brief context.
- PRs should describe scope, list local test commands run, and mention any impacts on MCP server or UI if applicable.
- Sign commits and affirm the DCO (see `CONTRIBUTING.md`).

## Available Installation Extras

| Extra | Description |
|-------|-------------|
| `postgres` / `postgres-binary` | PostgreSQL + PGVector support |
| `neo4j` | Neo4j graph database |
| `neptune` | AWS Neptune |
| `chromadb` | ChromaDB vector database |
| `docs` | Document processing (unstructured library) |
| `scraping` | Web scraping (Tavily, BeautifulSoup, Playwright) |
| `langchain` | LangChain integration |
| `llama-index` | LlamaIndex integration |
| `anthropic` | Anthropic Claude models |
| `ollama` | Ollama local models |
| `mistral` | Mistral AI models |
| `groq` | Groq API |
| `llama-cpp` | Llama.cpp local inference |
| `huggingface` | HuggingFace transformers |
| `aws` | S3 storage backend |
| `redis` | Redis caching |
| `graphiti` | Graphiti-core integration |
| `baml` | BAML structured output |
| `dlt` | Data load tool integration |
| `docling` | Docling document processing |
| `codegraph` | Code graph extraction |
| `evals` | Evaluation tools |
| `deepeval` | DeepEval testing framework |
| `posthog` | PostHog analytics |
| `monitoring` | Sentry + Langfuse observability |
| `tracing` | OpenTelemetry tracing |
| `distributed` | Modal distributed execution |
| `dev` | Development tools (pytest, mypy, ruff, etc.) |
| `debug` | Debugpy for debugging |

## CI Mirrors Local Commands

Our GitHub Actions run the same ruff checks and pytest suites shown above. Key workflows under `.github/workflows/`:
- `basic_tests.yml` - Unit tests and simple examples
- `integration_tests.yml` - Integration test suite
- `cli_tests.yml` - CLI command tests
- `e2e_tests.yml` - End-to-end tests
- `graph_db_tests.yml` - Graph database tests
- `vector_db_tests.yml` - Vector database tests
- `relational_db_migration_tests.yml` - Database migration tests
- `temporal_graph_tests.yml` - Temporal graph functionality
- `test_mcp.yml` - MCP server tests

Use the commands in this document locally to minimize CI surprises.

## Key SDK Entry Points

Main functions exported from `cognee/__init__.py`:
- `add(data, dataset_name)` - Ingest data
- `cognify(datasets)` - Build knowledge graph
- `search(query_text, query_type)` - Query knowledge
- `memify(extraction_tasks, enrichment_tasks)` - Enrich graph
- `run_custom_pipeline()` - Execute custom pipelines
- `delete(data_id)` - Remove data
- `config()` - Configuration management
- `datasets()` - Dataset operations
- `prune()` - Prune operations
- `update()` - Update data
- `visualize_graph()` / `start_visualization_server()` - Graph visualization
- `session()` - Session management
- `enable_tracing()` / `disable_tracing()` / `get_traces()` - Observability

All functions are async - use `await` or `asyncio.run()`.