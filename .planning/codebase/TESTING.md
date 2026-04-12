# Testing Patterns

**Analysis Date:** 2026-04-12

## Test Framework

**Primary Framework:** pytest 8.2.0

**Additional Dependencies:**
- `pytest-asyncio` 0.23.6 - For async test support

**Configuration Location:** `pyproject.toml` (Poetry dependencies)

## Test File Organization

### Directory Structure

```
test/
├── conftest.py              # Test configuration and shared fixtures
├── agent/                   # Agent-related tests
│   ├── test_action_docstring.py
│   ├── test_agent_custom_prompt.py
│   ├── test_agent_generator.py
│   ├── test_agent_graph.py
│   ├── test_agent_tools.py
│   ├── test_interview_action.py
│   ├── test_multi_agent_signup_create.py
│   └── test_twitter_user_agent_all_actions.py
├── infra/                   # Infrastructure tests
│   └── database/
│       ├── test_comment.py
│       ├── test_create_fetch_database.py
│       ├── test_multi_create_post.py
│       ├── test_multi_user_signup.py
│       ├── test_post.py
│       ├── test_search.py
│       ├── test_user.py
│       ├── test_user_signup.py
│       └── ... (18 total database tests)
└── test_data/              # Test data files
```

### Naming Conventions

- **Test files:** `test_*.py` prefix
- **Test functions:** `test_*` prefix with descriptive names
- **Test classes:** Not commonly used; prefer module-level functions

## Test Structure

### Basic Test Pattern

```python
# =========== Copyright 2023 @ CAMEL-AI.org. All Rights Reserved. ===========
# ... license header ...
import os
import os.path as osp
import sqlite3

import pytest

from oasis.social_platform.platform import Platform

parent_folder = osp.dirname(osp.abspath(__file__))
test_db_filepath = osp.join(parent_folder, "test.db")


class MockChannel:
    """Mock channel for testing platform operations."""

    def __init__(self):
        self.call_count = 0
        self.messages = []

    async def receive_from(self):
        # Return test commands
        if self.call_count == 0:
            self.call_count += 1
            return ("id_", (1, 2, "follow"))
        else:
            return ("id_", (None, None, "exit"))

    async def send_to(self, message):
        self.messages.append(message)


@pytest.fixture
def setup_platform():
    # Cleanup before test
    if os.path.exists(test_db_filepath):
        os.remove(test_db_filepath)

    mock_channel = MockChannel()
    instance = Platform(db_path=test_db_filepath, channel=mock_channel)
    return instance


@pytest.mark.asyncio
async def test_follow_user(setup_platform):
    try:
        platform = setup_platform

        # Setup test data
        conn = sqlite3.connect(test_db_filepath)
        cursor = conn.cursor()
        cursor.execute(
            ("INSERT INTO user "
             "(user_id, agent_id, user_name, num_followings, num_followers) "
             "VALUES (?, ?, ?, ?, ?)"),
            (1, 1, "user1", 0, 0),
        )
        conn.commit()

        # Execute
        await platform.running()

        # Assert
        cursor.execute(
            "SELECT * FROM follow WHERE follower_id=1 AND followee_id=2")
        assert cursor.fetchone() is not None, "Follow record not found"

    finally:
        # Cleanup
        conn.close()
        if os.path.exists(test_db_filepath):
            os.remove(test_db_filepath)
```

### Async Test Pattern

All async tests use `@pytest.mark.asyncio` decorator:

```python
@pytest.mark.asyncio
async def test_agents_posting(setup_platform):
    agents = []
    channel = Channel()
    infra = Platform(db_path=test_db_filepath,
                     channel=channel,
                     recsys_type='reddit')
    task = asyncio.create_task(infra.running())

    # Create agents and perform actions
    for i in range(2):
        agent = SocialAgent(agent_id=0, user_info=user_info, channel=channel)
        await agent.env.action.sign_up(f"user{i}", f"User{i}", "A bio.")
        agents.append(agent)

    # Test assertions
    response = await agents[1].perform_action_by_llm()
    assert any(tool_call.tool_name in ["multiply", "math_multiply"]
               for tool_call in response.info['tool_calls'])

    # Cleanup
    await channel.write_to_receive_queue((None, None, "exit"))
    await task
```

### Conditional Test Skipping

Skip tests when environment variables are not set:

```python
def neo4j_vars_set() -> bool:
    return (os.getenv("NEO4J_URI") is not None
           and os.getenv("NEO4J_USERNAME") is not None
           and os.getenv("NEO4J_PASSWORD") is not None)


@pytest.mark.skipif(
    not neo4j_vars_set(),
    reason="One or more neo4j env variables are not set",
)
def test_agent_neo4j_graph():
    # Test requires Neo4j connection
    pass
```

## Fixtures

### conftest.py

The `test/conftest.py` file configures the Python path:

```python
import os
import sys

# Add the project root directory to sys.path
root_path = os.path.abspath(os.path.join(os.path.dirname(__file__), "../"))
sys.path.insert(0, root_path)
```

### Common Fixture Patterns

Database test fixtures follow this pattern:

```python
@pytest.fixture
def setup_platform():
    # Ensure clean state
    if os.path.exists(test_db_filepath):
        os.remove(test_db_filepath)

    # Create instance
    mock_channel = MockChannel()
    instance = Platform(db_path, mock_channel)
    return instance
```

## Mocking Strategies

### Mock Classes

Create mock implementations for external dependencies:

```python
class MockChannel:
    def __init__(self):
        self.call_count = 0
        self.messages = []

    async def receive_from(self):
        # Return predetermined responses based on call count
        if self.call_count == 0:
            self.call_count += 1
            return ("id_", (1, 2, "follow"))
        # ... more conditions

    async def send_to(self, message):
        self.messages.append(message)
        # Assert on message content
```

### SQLite for Integration Tests

Tests use SQLite in-memory or file-based databases:

```python
test_db_filepath = osp.join(parent_folder, "test.db")

# Cleanup before and after
if os.path.exists(test_db_filepath):
    os.remove(test_db_filepath)

# Use raw sqlite3 for verification
conn = sqlite3.connect(test_db_filepath)
cursor = conn.cursor()
```

## Running Tests

### Commands

```bash
# Run all tests
pytest

# Run specific test file
pytest test/agent/test_agent_graph.py

# Run with verbose output
pytest -v

# Run specific test
pytest test/infra/database/test_user.py::test_follow_user
```

### Environment Variables

Some tests require environment variables:

```bash
export NEO4J_URI="bolt://localhost:7687"
export NEO4J_USERNAME="neo4j"
export NEO4J_PASSWORD="password"
export OPENAI_API_KEY="sk-..."
export OPENAI_BASE_URL="https://api.openai.com/v1"
```

## CI/CD Integration

### GitHub Actions

File: `.github/workflows/python-app.yml`

```yaml
name: Python application CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Set up Python 3.10.11
        uses: actions/setup-python@v2
        with:
          python-version: 3.10.11

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -e .

      - name: Install pre-commit
        run: pip install pre-commit

      - name: Run pre-commit
        run: pre-commit run --all-files

      - name: Run tests
        env:
          OPENAI_BASE_URL: ${{ secrets.OPENAI_BASE_URL }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
        run: pytest
```

### Pre-commit Hooks

Tests are not run as part of pre-commit, but code quality checks are:

- flake8 (PEP 8 compliance)
- ruff (formatting and linting)
- isort (import sorting)

## Test Coverage Areas

### Agent Tests (`test/agent/`)

| File | Coverage |
|------|----------|
| `test_agent_graph.py` | Agent graph operations, Neo4j backend |
| `test_agent_tools.py` | Agent tool usage with external toolkits |
| `test_agent_generator.py` | Agent generation from profiles |
| `test_twitter_user_agent_all_actions.py` | All Twitter actions |
| `test_interview_action.py` | Interview functionality |
| `test_multi_agent_signup_create.py` | Multi-agent scenarios |

### Database Tests (`test/infra/database/`)

| File | Coverage |
|------|----------|
| `test_user.py` | User operations, follow/unfollow, mute/unmute |
| `test_post.py` | Post creation, likes, dislikes |
| `test_comment.py` | Comment operations |
| `test_search.py` | Search functionality |
| `test_create_fetch_database.py` | Database creation |
| `test_trend.py` | Trending posts |
| `test_refresh.py` | Recommendation refresh |
| `test_group_chat.py` | Group messaging |
| `test_product.py` | Product/purchase operations |

## Testing Best Practices

1. **Always clean up** test databases after tests (try/finally blocks)
2. **Use fixtures** for common setup
3. **Mock external services** (OpenAI, Neo4j when not available)
4. **Test both success and failure** cases
5. **Use descriptive assertion messages** for clarity
6. **Skip tests** that require unavailable services with clear reasons

## Coverage

No explicit coverage target is configured, but the test suite covers:
- Database operations (CRUD for all entities)
- Agent actions and behaviors
- Platform operations
- Graph operations (in-memory and Neo4j)
- Recommendation systems

---

*Testing analysis: 2026-04-12*
