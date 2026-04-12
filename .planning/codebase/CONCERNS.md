# Codebase Concerns

**Analysis Date:** 2026-04-12

## Tech Debt

### Hardcoded Secrets and Placeholder Values

**Issue:** Hardcoded API keys and placeholder credentials exist in the codebase.

**Files:**
- `generator/reddit/user_generate.py:22` - Hardcoded OpenAI API key: `client = OpenAI(api_key='sk-xxx')`
- `visualization/twitter_simulation/group_polarization/group_polarization_eval.py:65-66` - Placeholder values for `Baseurl = "XXXXX"` and `Skey = "XXXXXX"`

**Impact:** Security risk if accidentally committed with real credentials; unclear configuration for external users.

**Fix approach:** Use environment variables or configuration files for all secrets. Add `.env.example` files to document required variables.

### TODO Comments Without Tracking

**Issue:** Multiple TODO comments scattered throughout the codebase without issue tracking.

**Files:**
- `oasis/social_agent/agent.py:160-161` - TODO to rewrite test/interview functions
- `oasis/social_agent/agent_environment.py:60,69,86` - TODOs for format changes and unimplemented environments
- `oasis/social_agent/agents_generator.py:44,103,188,206,238,262,286` - Multiple TODOs including scalability concerns
- `examples/experiment/twitter_simulation/group_polarization/twitter_simulation_group_polar.py:156` - TODO for adaptive test agents
- `examples/twitter_simulation_vllm.py:30,36` - TODOs for vLLM server URLs

**Impact:** Technical debt accumulation; unclear prioritization of improvements.

**Fix approach:** Create GitHub issues for each TODO and reference them in comments. Remove completed TODOs.

### Scalability Limitations in AgentGraph

**Issue:** AgentGraph class is noted to be too slow for 1 million agents.

**Files:**
- `oasis/social_agent/agents_generator.py:206-208` - Comment: "TODO when setting 100w agents, the agentgraph class is too slow. I use the list."
- `oasis/social_agent/agents_generator.py:286-287` - Comment: "TODO If we simulate 1 million agents, we can not use agent_graph class. It is not scalble."

**Impact:** Limits simulation scale; requires workarounds using plain lists instead of proper graph structure.

**Fix approach:** Profile AgentGraph performance bottlenecks. Consider using more efficient data structures or database-backed storage for large-scale simulations.

## Known Bugs

### Silent Exception Handling

**Issue:** Bare except clauses that silently swallow exceptions.

**Files:**
- `oasis/social_agent/agent_graph.py:207-210` - `add_edge` method silently passes on any exception:
  ```python
  try:
      self.graph.add_edge(agent_id_0, agent_id_1)
  except Exception:
      pass
  ```

**Impact:** Edge addition failures are invisible; debugging is difficult.

**Fix approach:** Log exceptions with appropriate context before passing, or use specific exception types.

### Debug Code in Production

**Issue:** PDB debugger calls left in production code.

**Files:**
- `oasis/social_platform/recsys.py:564-565,579-580` - Active `pdb.set_trace()` calls in recommendation system
- `visualization/twitter_simulation/align_with_real_world/code/graph.py:160,206,245,287` - Multiple `pdb.set_trace()` calls
- `oasis/social_platform/platform.py:72` - Commented but present `pdb.set_trace()`

**Impact:** Code execution will pause unexpectedly in production environments.

**Fix approach:** Remove all `pdb.set_trace()` calls. Use proper logging for debugging.

## Security Considerations

### SQL Injection Risk

**Issue:** SQL queries use string formatting with f-strings in some places.

**Files:**
- `oasis/social_platform/platform.py:307-311` - Uses f-string for IN clause:
  ```python
  placeholders = ", ".join("?" for _ in selected_post_ids)
  post_query = (
      f"SELECT post_id, user_id, original_post_id, content, "
      f"quote_content, created_at, num_likes, num_dislikes, "
      f"num_shares FROM post WHERE post_id IN ({placeholders})")
  ```

**Impact:** While this specific instance uses parameterized placeholders, the pattern is risky and could lead to injection if modified incorrectly.

**Current mitigation:** Uses proper parameterization with `?` placeholders.

**Recommendations:** Audit all SQL queries to ensure consistent use of parameterized queries. Never use f-strings for SQL.

### Missing Input Validation

**Issue:** User inputs are not consistently validated before processing.

**Files:**
- `oasis/social_agent/agent_action.py:258-275` - `perform_action_by_hci` uses `int(input(...))` without validation
- `oasis/social_agent/agents_generator.py:370-372` - Direct `input()` calls without validation

**Impact:** Type conversion errors or unexpected input could crash the application.

**Fix approach:** Add input validation and error handling for all user inputs.

## Performance Bottlenecks

### Global State in Recsys Module

**Issue:** Heavy use of global variables for caching in recommendation system.

**Files:**
- `oasis/social_platform/recsys.py:39-61` - Multiple global variables:
  ```python
  model = None
  twhin_tokenizer = None
  twhin_model = None
  user_previous_post_all = {}
  user_previous_post = {}
  user_profiles = []
  t_items = {}
  u_items = {}
  date_score = []
  ```

**Impact:** Global state makes the code hard to test and thread-unsafe. Memory usage grows unbounded with simulation size.

**Fix approach:** Encapsulate state in a class. Implement proper cache eviction policies.

### Synchronous Database Operations

**Issue:** Database operations in async context use synchronous SQLite.

**Files:**
- `oasis/social_platform/platform.py` - All database operations are synchronous
- `oasis/social_platform/database.py` - Uses standard `sqlite3` module

**Impact:** Blocks the event loop during database operations, reducing concurrency.

**Fix approach:** Consider using `aiosqlite` for async database operations, or run DB operations in thread pool.

### Inefficient String Concatenation in Loops

**Issue:** Profile string building uses repeated concatenation.

**Files:**
- `oasis/social_platform/recsys.py:509-520` - String concatenation in loop for profile updates

**Impact:** O(n^2) complexity for string operations.

**Fix approach:** Use list join pattern or StringIO for building large strings.

## Code Quality Issues

### Large Files and Functions

**Issue:** Several files exceed recommended line counts and have complex functions.

**Files:**
- `oasis/social_platform/platform.py` - 1,642 lines (exceeds 800-line recommendation)
- `oasis/social_agent/agent_action.py` - 758 lines
- `oasis/social_platform/recsys.py` - 797 lines

**Functions with high complexity:**
- `oasis/social_platform/platform.py:328-398` - `update_rec_table()` - Complex branching for different recsys types
- `oasis/social_platform/recsys.py:419-606` - `rec_sys_personalized_twh()` - 187 lines, multiple responsibilities

**Fix approach:** Refactor large files into modules. Extract smaller, focused functions.

### Commented Code Blocks

**Issue:** Large blocks of commented code remain in the codebase.

**Files:**
- `oasis/social_platform/platform.py:200-203,420-424,908-911` - Commented logging code
- `oasis/social_agent/agent.py:167-168,185-186` - Commented memory operations
- `oasis/social_platform/platform.py:227-256` - Commented exception handling

**Impact:** Clutters the codebase; unclear if code is temporarily disabled or permanently removed.

**Fix approach:** Remove commented code. Use version control for history.

### Type Inconsistencies

**Issue:** Type annotations are inconsistent or missing in some places.

**Files:**
- `oasis/social_platform/platform.py:67` - `following_post_count=3` lacks type annotation
- `oasis/social_platform/platform.py:56-69` - Mix of type annotated and non-annotated parameters
- `oasis/social_agent/agents_generator.py:179-186` - Function parameters lack type annotations

**Fix approach:** Add comprehensive type annotations. Enable stricter mypy checking.

## Architectural Limitations

### Tight Coupling Between Components

**Issue:** Platform and agent components are tightly coupled.

**Files:**
- `oasis/social_platform/platform.py` - Platform class knows about agent actions
- `oasis/social_agent/agent.py` - Agent directly accesses platform internals via `env.action`

**Impact:** Difficult to test components in isolation. Changes to one component often require changes to others.

**Fix approach:** Define clear interfaces between components. Use dependency injection.

### Database Schema Management

**Issue:** Database schema is managed through individual SQL files executed in code.

**Files:**
- `oasis/social_platform/database.py:84-201` - Schema creation reads and executes multiple SQL files
- `oasis/social_platform/schema/` - Multiple .sql files for each table

**Impact:** No migration system; schema changes are difficult to track and apply.

**Fix approach:** Consider using a migration tool like Alembic or Django migrations.

### Channel Polling Pattern

**Issue:** Channel uses polling with sleep for message retrieval.

**Files:**
- `oasis/social_platform/channel.py:62-71`:
  ```python
  async def read_from_send_queue(self, message_id):
      while True:
          if message_id in await self.send_dict.keys():
              message = await self.send_dict.pop(message_id, None)
              if message:
                  return message
          await asyncio.sleep(0.1)  # set a large one to reduce the workload of cpu
  ```

**Impact:** Inefficient CPU usage; message latency depends on polling interval.

**Fix approach:** Use asyncio Events or Conditions for notification-based waiting.

## Missing Documentation

### Incomplete Docstrings

**Issue:** Several functions have incomplete or outdated docstrings.

**Files:**
- `oasis/social_agent/agents_generator.py:34-61` - Docstring says "TODO: need update the description of args"
- `oasis/social_agent/agents_generator.py:179-203` - Docstring says "TODO: need update the description of args"

**Impact:** API documentation is misleading or incomplete.

**Fix approach:** Update all docstrings to match actual parameters and behavior.

### Missing Architecture Documentation

**Issue:** No high-level documentation explaining the system architecture.

**Impact:** New contributors struggle to understand the codebase structure.

**Fix approach:** Add ARCHITECTURE.md and STRUCTURE.md documents.

## Dependencies at Risk

### Outdated Dependencies

**Issue:** Some dependencies may be outdated based on the poetry.lock file date.

**Files:**
- `pyproject.toml` - camel-ai pinned to 0.2.78 (check for newer versions)
- `pyproject.toml` - pandas pinned to 2.2.2

**Impact:** Missing security patches and bug fixes.

**Fix approach:** Regularly update dependencies and run tests. Consider using Dependabot.

### Heavy ML Dependencies

**Issue:** Heavy dependencies on ML libraries may cause deployment issues.

**Files:**
- `pyproject.toml` - sentence-transformers, torch (via camel-ai)
- `oasis/social_platform/recsys.py` - Direct imports of torch, transformers

**Impact:** Large Docker images; difficult deployment in resource-constrained environments.

**Fix approach:** Make ML components optional with feature flags.

## Test Coverage Gaps

### Missing Unit Tests for Core Logic

**Issue:** Test directory structure suggests limited coverage.

**Files:**
- `test/` directory contains mostly integration tests for database operations
- `oasis/social_platform/recsys.py` - No dedicated unit tests for recommendation algorithms
- `oasis/social_agent/agent.py` - No unit tests for agent decision logic

**Risk:** Core algorithms may have undetected bugs.

**Fix approach:** Add unit tests for all recommendation functions and agent methods.

### No E2E Tests

**Issue:** No end-to-end tests for complete simulation workflows.

**Risk:** Integration issues between components may not be caught.

**Fix approach:** Add E2E tests for common simulation scenarios.

## Logging Issues

### Excessive Debug Logging

**Issue:** Many modules set log level to DEBUG by default.

**Files:**
- `oasis/social_agent/agent.py:40` - `agent_log.setLevel("DEBUG")`
- `oasis/social_platform/platform.py:43` - `twitter_log.setLevel("DEBUG")`
- `oasis/social_platform/recsys.py:36` - `rec_log.setLevel('DEBUG')`

**Impact:** Log files grow very large in production.

**Fix approach:** Use INFO as default level. Make log level configurable via environment variable.

### Log Directory Hardcoding

**Issue:** Log directories are hardcoded to `./log`.

**Files:**
- `oasis/social_platform/platform.py:37-39`
- `oasis/social_agent/agent.py:43-45`
- `oasis/environment/env.py:30-31`

**Impact:** Inflexible deployment; may fail in read-only filesystems.

**Fix approach:** Make log directory configurable via environment variable.

---

*Concerns audit: 2026-04-12*
