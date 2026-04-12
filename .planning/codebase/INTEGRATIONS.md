# External Integrations

**Analysis Date:** 2026-04-12

## APIs & External Services

**LLM/AI Providers:**
- **OpenAI** - Primary LLM provider
  - Models: GPT-4, GPT-4o-mini, text-embedding-3-small
  - Env var: `OPENAI_API_KEY`
  - Usage: Agent decision-making, embeddings
  - File: `oasis/social_agent/agent.py`, `oasis/social_platform/process_recsys_posts.py`

- **Moonshot (Kimi)** - Chinese LLM provider
  - Models: MOONSHOT_KIMI_K2_5, MOONSHOT_KIMI_K2, MOONSHOT_V1 series
  - Env var: `MOONSHOT_API_KEY`
  - Usage: Alternative LLM backend
  - File: `examples/kimi_native_simulation.py`, `docs/KIMI_INTEGRATION.md`

- **Alibaba Qwen** - Alternative LLM provider
  - Env var: `QWEN_API_KEY`
  - Usage: Alternative LLM backend
  - File: `examples/bailian_qwen_simulation.py`

**Search & Data:**
- **Jina URL Reader** - URL content extraction
  - Env var: `JINA_API_KEY`
  - Usage: Reading web content for agents

- **Google Custom Search** - Web search capability
  - Env vars: `GOOGLE_API_KEY`, `SEARCH_ENGINE_ID`
  - Usage: Agent search tools

- **Cohere** - Reranking service
  - Env var: `COHERE_API_KEY`
  - Usage: Search result reranking

- **OpenWeatherMap** - Weather data
  - Env var: `OPENWEATHERMAP_API_KEY`
  - Usage: Weather information for agents

**Communication:**
- **Slack** - Messaging integration
  - Package: `slack_sdk` 3.31.0
  - Usage: Notifications, bot interactions
  - OAuth via `requests_oauthlib`

## Data Storage

**Databases:**
- **SQLite** - Primary database
  - Location: `./data/social_media.db` (configurable via `OASIS_DB_PATH`)
  - Schema files: `oasis/social_platform/schema/`
  - Tables: user, post, follow, mute, like, dislike, report, trace, rec, comment, product, chat_group, group_member, group_message
  - File: `oasis/social_platform/database.py`

- **Neo4j** - Graph database (optional)
  - Version: 5.23.0
  - Usage: Social network visualization
  - Config: `oasis/social_platform/config/neo4j.py`
  - Connection: URI, username, password via `Neo4jConfig`
  - Files: `visualization/dynamic_follow_network/code/vis_neo4j_*.py`

**File Storage:**
- Local filesystem only
- Log directory: `./log/`
- Data directory: `./data/`
- Models cache: `./models/` (for sentence-transformers)

**Caching:**
- In-memory caching for recommendation system models
- Global variables for TwHIN-BERT tokenizer/model singletons
- No external cache (Redis/Memcached) detected

## Authentication & Identity

**Auth Provider:**
- Custom/OAuth-based
- `requests_oauthlib` 2.0.0 for OAuth flows
- No centralized auth provider (simulation environment)

**API Keys:**
- All API keys passed via environment variables
- No secret manager integration detected

## Monitoring & Observability

**Error Tracking:**
- None detected (no Sentry, Rollbar, etc.)

**Logging:**
- Python standard `logging` module
- Log files: `./log/social.agent-{timestamp}.log`, `./log/social.twitter-{timestamp}.log`
- Log level: DEBUG
- Format: `%(levelname)s - %(asctime)s - %(name)s - %(message)s`

**Tracing:**
- Custom trace table in SQLite database
- Records all agent actions with timestamps
- File: `oasis/social_platform/platform.py` (trace recording)

## CI/CD & Deployment

**Hosting:**
- PyPI package: `camel-oasis`
- Docker support: `.container/Dockerfile`, `.container/docker-compose.yaml`
- No cloud-specific deployment configs detected

**CI Pipeline:**
- **GitHub Actions** (`.github/workflows/python-app.yml`)
  - Triggers: Push/PR to main branch
  - Python 3.10.11
  - Steps: checkout, install deps, pre-commit, pytest
  - Secrets: `OPENAI_BASE_URL`, `OPENAI_API_KEY`

**Pre-commit Hooks:**
- End-of-file fixer
- Mixed line ending fixer
- Trailing whitespace removal
- TOML/YAML validation
- Import sorting (isort)
- PEP8 checking (flake8)
- Ruff formatting
- Markdown formatting
- License checking

## Environment Configuration

**Required env vars:**
- `OPENAI_API_KEY` - For OpenAI model access

**Optional env vars:**
- `MOONSHOT_API_KEY` - For Moonshot/Kimi models
- `QWEN_API_KEY` - For Qwen models
- `ANTHROPIC_API_KEY` - For Anthropic-compatible APIs
- `OASIS_DB_PATH` - Custom database path
- `JINA_API_KEY` - Jina URL Reader
- `COHERE_API_KEY` - Cohere reranking
- `GOOGLE_API_KEY` + `SEARCH_ENGINE_ID` - Google Search
- `OPENWEATHERMAP_API_KEY` - Weather data

**Secrets location:**
- Environment variables only
- `.env.example` shows expected variables (not real values)
- `.gitignore` excludes `.env` files

## Webhooks & Callbacks

**Incoming:**
- None detected (batch simulation system)

**Outgoing:**
- None detected (no webhook callbacks)

## Data Sources/Destinations

**Input Data:**
- JSON agent profiles (e.g., `data/reddit/user_data_36.json`)
- HuggingFace datasets (referenced in README)
  - Dataset: `echo-yiyiyi/oasis-dataset`

**Output Data:**
- SQLite database with full simulation trace
- Log files
- Neo4j graph exports (optional)

**Model Downloads:**
- HuggingFace Hub (transformers)
  - `Twitter/twhin-bert-base` - Twitter-specific BERT
  - `paraphrase-MiniLM-L6-v2` - Sentence transformer
- Models cached in `./models/` directory

## Integration Architecture

**CAMEL Framework Integration:**
- Core dependency for all LLM interactions
- Provides abstraction over multiple model providers
- Handles tool calling/function calling for agent actions
- File: All agent-related code depends on `camel.*` imports

**Platform Abstraction:**
- Support for Twitter/X-like and Reddit-like platforms
- Configurable via `DefaultPlatformType` enum
- Different recommendation algorithms per platform
- Files: `oasis/social_platform/typing.py`, `oasis/social_platform/recsys.py`

---

*Integration audit: 2026-04-12*
