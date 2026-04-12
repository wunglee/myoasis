# Technology Stack

**Analysis Date:** 2026-04-12

## Languages

**Primary:**
- Python 3.10-3.11 (specified as `>=3.10.0,<3.12` in `pyproject.toml`)

**Secondary:**
- SQL (SQLite schema definitions in `oasis/social_platform/schema/`)

## Runtime

**Environment:**
- Python 3.10+ with asyncio support
- Supports both CPU and CUDA (GPU) execution for ML models

**Package Manager:**
- Poetry (primary) - `pyproject.toml` + `poetry.lock`
- pip (fallback) - package installable via `pip install camel-oasis`
- uv (in container) - used in Dockerfile for faster installs

## Frameworks

**Core AI/ML:**
- `camel-ai` 0.2.78 - Core multi-agent framework for LLM agent orchestration
- `transformers` (HuggingFace) - For TwHIN-BERT model loading and inference
- `sentence-transformers` 3.0.0 - For text embeddings (paraphrase-MiniLM-L6-v2)
- `torch` - PyTorch for tensor operations and model inference
- `numpy` - Numerical computations
- `scikit-learn` - TF-IDF vectorization and cosine similarity

**Data Processing:**
- `pandas` 2.2.2 - Data manipulation and analysis
- `unstructured` 0.13.7 - Document processing

**Graph/Network:**
- `igraph` 0.11.6 - Social network graph operations
- `cairocffi` 1.7.1 - Graph visualization

**Database:**
- SQLite3 (built-in) - Primary database for social platform data
- `neo4j` 5.23.0 - Optional graph database for visualization

**Image Processing:**
- `pillow` 10.3.0 - Image handling

## Key Dependencies

**Critical:**
- `camel-ai` 0.2.78 - The entire agent system is built on CAMEL framework
  - Provides `ChatAgent`, `ModelFactory`, `BaseMessage`, `FunctionTool`
  - Handles LLM interactions and tool calling
- `torch` - Required for embedding generation in recommendation system
- `transformers` - Required for TwHIN-BERT model (Twitter/X-specific embeddings)

**Infrastructure:**
- `pytest` 8.2.0 + `pytest-asyncio` 0.23.6 - Testing framework
- `pre-commit` 3.7.1 - Git hooks for code quality
- `slack_sdk` 3.31.0 - Slack integration (optional)
- `requests_oauthlib` 2.0.0 - OAuth authentication

**API/Documentation:**
- `prance` 23.6.21.0 - OpenAPI/Swagger parsing
- `openapi-spec-validator` 0.7.1 - OpenAPI spec validation

## Configuration

**Environment:**
- Environment variables for API keys (see `.container/.env.example`):
  - `OPENAI_API_KEY` - Required for OpenAI models
  - `MOONSHOT_API_KEY` - For Moonshot/Kimi models
  - `QWEN_API_KEY` - For Alibaba Qwen models
  - `JINA_API_KEY` - For Jina URL Reader
  - `COHERE_API_KEY` - For Cohere Rerank
  - `GOOGLE_API_KEY` + `SEARCH_ENGINE_ID` - For Google Search
  - `OPENWEATHERMAP_API_KEY` - For weather data
  - `OASIS_DB_PATH` - Optional custom database path

**Build:**
- `pyproject.toml` - Poetry configuration, dependencies, package metadata
- `poetry.lock` - Locked dependency versions
- No `setup.py` - uses Poetry build system

**Code Quality:**
- `.pre-commit-config.yaml` - Pre-commit hooks configuration
  - `isort` 5.13.2 - Import sorting
  - `flake8` 7.0.0 - PEP8 linting
  - `ruff` v0.3.5 - Fast Python linter/formatter
  - `mdformat` 0.7.17 - Markdown formatting
  - `yamllint` v1.35.1 - YAML linting
  - Custom license checker

## Platform Requirements

**Development:**
- Python 3.10 or 3.11
- Poetry for dependency management
- Pre-commit hooks installed
- SQLite3 (included with Python)

**Production:**
- Docker support (`.container/Dockerfile`)
- Base image: `python:3.10-bookworm`
- Optional: CUDA-enabled PyTorch for GPU acceleration
- Optional: Neo4j for graph visualization

**Scaling:**
- Supports up to 1 million agents (as per project claims)
- Recommendation system supports batch processing
- Database: SQLite (single-node) or can be extended

## AI/ML Stack

**LLM Integration (via CAMEL):**
- OpenAI GPT models (GPT-4, GPT-4o-mini)
- Moonshot/Kimi models (K2.5, K2, V1 series)
- Alibaba Qwen models
- Any CAMEL-supported model platform

**Embedding Models:**
- OpenAI Embeddings (text-embedding-3-small)
- TwHIN-BERT (Twitter/X-specific BERT model)
- paraphrase-MiniLM-L6-v2 (SentenceTransformer)

**Recommendation System:**
- Interest-based (cosine similarity on embeddings)
- Hot-score-based (Reddit-style ranking algorithm)
- Random (baseline)

---

*Stack analysis: 2026-04-12*
