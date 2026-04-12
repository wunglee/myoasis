# Codebase Structure

**Analysis Date:** 2026-04-12

## Directory Layout

```
/Users/wangli/projects/myoasis/
├── oasis/                      # Main package source code
│   ├── __init__.py             # Package exports and version
│   ├── clock/                  # Time management for simulation
│   ├── environment/            # Environment orchestration (PettingZoo-style API)
│   ├── social_agent/           # Agent implementations and graph management
│   └── social_platform/        # Platform backend simulation
│       ├── config/             # User info and Neo4j configuration
│       └── schema/             # SQL schema definitions
├── examples/                   # Usage examples and tutorials
│   ├── experiment/             # Large-scale experiment guides
│   ├── quick_start.py          # Basic usage example
│   ├── twitter_simulation_openai.py
│   ├── reddit_simulation_openai.py
│   └── ...                     # Other simulation examples
├── data/                       # Sample datasets
│   ├── twitter_dataset/        # Twitter user profiles
│   ├── reddit/                 # Reddit user profiles
│   └── emall/                  # E-commerce product data
├── test/                       # Test suite
│   ├── infra/                  # Infrastructure tests
│   │   └── database/           # Database operation tests
│   └── agent/                  # Agent-related tests
├── docs/                       # Documentation (Mintlify format)
│   ├── api-reference/          # API documentation
│   ├── cookbooks/              # Tutorial guides
│   ├── key_modules/            # Module explanations
│   └── *.mdx                   # Documentation pages
├── visualization/              # Analysis and visualization scripts
│   ├── twitter_simulation/     # Twitter-specific analysis
│   ├── reddit_simulation_*/    # Reddit-specific analysis
│   └── dynamic_follow_network/ # Network visualization
├── generator/                  # Data generation utilities
├── models/                     # ML model cache directory
├── log/                        # Runtime logs (generated)
├── pyproject.toml              # Poetry package configuration
├── poetry.lock                 # Locked dependencies
└── README.md                   # Project overview
```

## Directory Purposes

**oasis/ (Main Package):**
- Purpose: Core simulation framework
- Contains: All source code for the OASIS environment
- Key files: `__init__.py` exports public API

**oasis/environment/:**
- Purpose: Environment lifecycle management
- Contains: `env.py` (OasisEnv class), `make.py` (factory), `env_action.py` (action dataclasses)
- Key files: `oasis/environment/env.py` - main environment class

**oasis/social_agent/:**
- Purpose: Agent definitions and social graph
- Contains: `agent.py` (SocialAgent), `agent_graph.py` (AgentGraph with igraph/Neo4j), `agents_generator.py`, `agent_action.py`, `agent_environment.py`
- Key files: `agent.py`, `agent_graph.py`, `agents_generator.py`

**oasis/social_platform/:**
- Purpose: Social media backend simulation
- Contains: `platform.py` (main Platform class), `channel.py` (async messaging), `database.py` (SQLite ops), `recsys.py` (recommendation algorithms), `typing.py` (enums), `platform_utils.py`
- Key files: `platform.py`, `channel.py`, `database.py`, `recsys.py`

**oasis/social_platform/config/:**
- Purpose: Configuration classes
- Contains: `user.py` (UserInfo dataclass), `neo4j.py` (Neo4jConfig)
- Key files: `user.py` - agent profile configuration

**oasis/social_platform/schema/:**
- Purpose: SQL table definitions
- Contains: 17 `.sql` files defining database schema
- Key files: `user.sql`, `post.sql`, `follow.sql`, `trace.sql`, `rec.sql`

**oasis/clock/:**
- Purpose: Simulation time management
- Contains: `clock.py` (Clock class with time scaling)
- Key files: `clock.py`

**oasis/testing/:**
- Purpose: Debugging utilities
- Contains: `show_db.py` (database inspection)
- Key files: `show_db.py`

**examples/:**
- Purpose: Demonstration scripts and tutorials
- Contains: Various simulation examples and experiment documentation
- Key files: `quick_start.py`, `twitter_simulation_openai.py`, `reddit_simulation_openai.py`

**test/:**
- Purpose: pytest test suite
- Contains: `conftest.py`, `infra/database/` (DB tests), `agent/` (agent tests)
- Key files: `conftest.py`

**data/:**
- Purpose: Sample datasets for simulations
- Contains: JSON/CSV files with user profiles and agent data
- Key files: `reddit/user_data_36.json`, `twitter_dataset/*.csv`

**docs/:**
- Purpose: Documentation website content
- Contains: `.mdx` files for Mintlify documentation
- Key files: `quickstart.mdx`, `introduction.mdx`

**visualization/:**
- Purpose: Post-simulation analysis scripts
- Contains: Python scripts for generating figures and analysis
- Key files: Various `analysis_*.py` and `vis_*.py` scripts

## Key File Locations

**Entry Points:**
- `oasis/__init__.py` - Package entry point, exports public API
- `oasis/environment/make.py` - Environment factory function
- `examples/quick_start.py` - Quick start example

**Configuration:**
- `pyproject.toml` - Poetry package config, dependencies
- `.pre-commit-config.yaml` - Pre-commit hooks

**Core Logic:**
- `oasis/environment/env.py` - OasisEnv class (main environment)
- `oasis/social_agent/agent.py` - SocialAgent class (LLM agent)
- `oasis/social_agent/agent_graph.py` - AgentGraph class (social network)
- `oasis/social_platform/platform.py` - Platform class (social media backend)
- `oasis/social_platform/channel.py` - Channel class (async messaging)
- `oasis/social_platform/recsys.py` - Recommendation algorithms

**Database:**
- `oasis/social_platform/database.py` - Database operations
- `oasis/social_platform/schema/*.sql` - Table schemas

**Testing:**
- `test/conftest.py` - pytest configuration
- `test/infra/database/test_*.py` - Database tests

## Naming Conventions

**Files:**
- Python modules: `snake_case.py`
- SQL schemas: `snake_case.sql`
- Examples: `snake_case.py` or `descriptive_name_simulation.py`

**Directories:**
- Package modules: `snake_case/` (e.g., `social_agent/`, `social_platform/`)
- Documentation: `kebab-case/` (e.g., `api-reference/`)

**Classes:**
- PascalCase: `OasisEnv`, `SocialAgent`, `AgentGraph`, `Platform`, `Channel`

**Functions/Methods:**
- snake_case: `make()`, `perform_action()`, `update_rec_table()`

## Where to Add New Code

**New Platform Action:**
- Implementation: `oasis/social_platform/platform.py` - add async method
- Registration: `oasis/social_agent/agent_action.py` - add to `get_openai_function_list()`
- Typing: `oasis/social_platform/typing.py` - add to `ActionType` enum
- Schema: `oasis/social_platform/schema/` - add SQL table if needed

**New Agent Type:**
- Implementation: `oasis/social_agent/agent.py` or new file
- Registration: `oasis/social_agent/__init__.py` - add to exports

**New Recommendation Algorithm:**
- Implementation: `oasis/social_platform/recsys.py` - add function
- Integration: `oasis/social_platform/platform.py` - add case in `update_rec_table()`
- Typing: `oasis/social_platform/typing.py` - add to `RecsysType` enum

**New Example:**
- Location: `examples/` directory
- Naming: `descriptive_simulation.py`

**Tests:**
- Unit tests: `test/agent/` or `test/infra/`
- Database tests: `test/infra/database/`
- Naming: `test_feature.py`

## Special Directories

**log/:**
- Purpose: Runtime log files
- Generated: Yes (created at runtime)
- Committed: No (in `.gitignore`)
- Pattern: `social.agent-YYYY-MM-DD_HH-MM-SS.log`, `social.twitter-*.log`, `oasis-*.log`

**models/:**
- Purpose: ML model cache (sentence-transformers)
- Generated: Yes (downloaded at runtime)
- Committed: No

**.venv/:**
- Purpose: Python virtual environment
- Generated: Yes
- Committed: No

**__pycache__/:**
- Purpose: Python bytecode cache
- Generated: Yes
- Committed: No

---

*Structure analysis: 2026-04-12*
