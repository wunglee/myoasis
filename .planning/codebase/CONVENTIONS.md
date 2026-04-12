# Coding Conventions

**Analysis Date:** 2026-04-12

## Overview

This codebase follows Python conventions with strict linting and formatting enforced through pre-commit hooks. The project uses Poetry for dependency management and targets Python 3.10-3.11.

## Code Style

### Formatting Tools

The project uses multiple tools for code quality, configured in `.pre-commit-config.yaml`:

| Tool | Purpose | Version |
|------|---------|---------|
| **ruff** | Fast Python linter and formatter | v0.3.5 |
| **flake8** | PEP 8 compliance checking | 7.0.0 |
| **isort** | Import sorting | 5.13.2 |
| **mdformat** | Markdown formatting | 0.7.17 |
| **yamllint** | YAML linting | v1.35.1 |

### Pre-commit Hooks

All hooks run automatically on commit via `.pre-commit-config.yaml`:

```yaml
- End-of-file fixer
- Mixed line ending fixer (enforces LF)
- Trailing whitespace removal
- TOML/YAML validation
- Import sorting (isort)
- PEP 8 checking (flake8)
- Ruff formatting with auto-fix
- License header checking
```

Run manually: `pre-commit run --all-files`

## Naming Conventions

### Files
- **Modules**: `snake_case.py` (e.g., `agent_action.py`, `platform_utils.py`)
- **Test files**: `test_*.py` prefix (e.g., `test_user.py`, `test_agent_graph.py`)
- **Schema files**: `*.sql` for database schemas

### Classes
- **PascalCase** for all class names
- Examples from `oasis/social_agent/agent.py`:
  ```python
  class SocialAgent(ChatAgent):
  class SocialAction:
  class AgentGraph:
  ```

### Functions and Methods
- **snake_case** for all functions and methods
- Examples:
  ```python
  async def perform_action_by_llm(self):
  async def create_post(self, agent_id: int, content: str):
  def get_openai_function_list(self) -> list[FunctionTool]:
  ```

### Variables
- **snake_case** for variables
- Private methods/attributes use single underscore prefix: `_execute_db_command`
- Constants at module level use **UPPER_SNAKE_CASE**:
  ```python
  SCHEMA_DIR = "social_platform/schema"
  DB_NAME = "social_media.db"
  USER_SCHEMA_SQL = "user.sql"
  ```

### Type Variables
- Use descriptive names with type hints
- Union types use `|` syntax (Python 3.10+): `str | None`
- Optional values: `Optional[Type]` or `Type | None`

## Import Organization

### Import Order (enforced by isort)

1. **Standard library** imports
2. **Third-party** imports
3. **Local application** imports

Example from `oasis/social_agent/agent.py`:

```python
# Standard library
from __future__ import annotations
import inspect
import logging
import sys
from datetime import datetime
from typing import TYPE_CHECKING, Any, Callable, List, Optional, Union

# Third-party
camel.agents import ChatAgent
camel.messages import BaseMessage
camel.models import BaseModelBackend, ModelManager

# Local imports
from oasis.social_agent.agent_action import SocialAction
from oasis.social_platform import Channel
from oasis.social_platform.config import UserInfo
```

### Import Patterns

- Use `from __future__ import annotations` for postponed evaluation of annotations
- Use `TYPE_CHECKING` for imports only needed during type checking:
  ```python
  from typing import TYPE_CHECKING
  if TYPE_CHECKING:
      from oasis.social_agent import AgentGraph
  ```
- Relative imports within packages: `from .neo4j import Neo4jConfig`

## Type Hints

### Function Signatures

All functions use type annotations:

```python
async def create_post(self, agent_id: int, content: str) -> dict:
async def refresh(self, agent_id: int) -> dict[str, Any]:
def get_db_path() -> str:
```

### Return Types

- Explicit return type annotations required
- Async functions return `Coroutine` or specific types
- Dictionary returns specify key/value types when possible: `dict[str, Any]`

### Special Types

- Union types: `str | None` (preferred) or `Optional[str]`
- Lists with element types: `list[FunctionTool]`
- Callable types for function parameters

## Error Handling

### Exception Patterns

Try/except blocks wrap database operations and external calls:

```python
try:
    self.pl_utils._execute_db_command(query, params, commit=True)
    result = self.db_cursor.fetchone()
    return {"success": True, "data": result}
except Exception as e:
    return {"success": False, "error": str(e)}
```

### Error Return Format

Functions return dictionaries with consistent structure:
- Success: `{"success": True, ...}`
- Failure: `{"success": False, "error": "message"}`

## Documentation

### License Headers

Every Python file starts with the Apache 2.0 license header:

```python
# =========== Copyright 2023 @ CAMEL-AI.org. All Rights Reserved. ===========
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# =========== Copyright 2023 @ CAMEL-AI.org. All Rights Reserved. ===========
```

### Docstrings

- Use triple quotes for docstrings
- Google-style or Sphinx-style docstrings for public APIs
- Type information in docstrings when not in signature

Example:
```python
async def sign_up(self, user_name: str, name: str, bio: str):
    r"""Signs up a new user with the provided username, name, and bio.

    Args:
        user_name (str): The username for the new user.
        name (str): The full name of the new user.
        bio (str): A brief biography of the new user.

    Returns:
        dict: A dictionary with success status and user_id.
    """
```

### Comments

- Use `#` for inline comments
- Prefix TODOs with `TODO:`
- Comment complex logic or business rules

## Code Patterns

### Dataclasses

Use `@dataclass` for configuration and data containers:

```python
from dataclasses import dataclass

@dataclass
class UserInfo:
    user_name: str | None = None
    name: str | None = None
    description: str | None = None
    profile: dict[str, Any] | None = None
    recsys_type: str = "twitter"
```

### Async/Await

Heavy use of async patterns for I/O operations:

```python
async def perform_action_by_llm(self):
    env_prompt = await self.env.to_text_prompt()
    response = await self.astep(user_msg)
    return response
```

### String Formatting

- Use f-strings for string interpolation
- Multi-line strings use triple quotes
- SQL queries use triple quotes for readability

## Logging

Use the standard `logging` module with module-level loggers:

```python
import logging

if "sphinx" not in sys.modules:
    twitter_log = logging.getLogger(name="social.twitter")
    twitter_log.setLevel("DEBUG")
    file_handler = logging.FileHandler(f"./log/social.twitter-{now}.log")
    file_handler.setFormatter(
        logging.Formatter(
            "%(levelname)s - %(asctime)s - %(name)s - %(message)s"))
    twitter_log.addHandler(file_handler)
```

Log files are written to `./log/` directory with timestamps.

## Configuration

### Environment Variables

- Check for environment variables at runtime
- Use `os.environ.get()` with defaults or raise errors for required vars

### Database Paths

Support both environment-based and default paths:

```python
def get_db_path() -> str:
    env_db_path = os.environ.get("OASIS_DB_PATH")
    if env_db_path:
        return env_db_path
    # Fall back to default path
    return default_db_path
```

---

*Convention analysis: 2026-04-12*
