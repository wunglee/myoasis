# Architecture

**Analysis Date:** 2026-04-12

## Pattern Overview

**Overall:** Multi-Agent Social Simulation Framework with Environment-Agent Pattern

**Key Characteristics:**
- PettingZoo-style API (`make()`, `reset()`, `step()`, `close()`)
- Asynchronous agent execution with semaphore-based concurrency control
- Channel-based message passing between agents and platform
- Pluggable recommendation systems (Twitter-style, Reddit-style, Random)
- Graph-based agent relationship management (igraph or Neo4j backends)
- LLM-driven agent behavior via tool-calling interface

## Layers

**Environment Layer:**
- Purpose: Orchestrates the simulation lifecycle and manages agent-platform interactions
- Location: `oasis/environment/`
- Contains: `OasisEnv` class, environment factory (`make.py`), action definitions
- Depends on: Agent graph, Platform, Clock
- Used by: End-user scripts, examples

**Agent Layer:**
- Purpose: Encapsulates individual LLM-powered social agents with their behavior and state
- Location: `oasis/social_agent/`
- Contains: `SocialAgent`, `AgentGraph`, agent generation utilities, action wrappers
- Depends on: CAMEL framework, Platform channel, Environment actions
- Used by: Environment layer

**Platform Layer:**
- Purpose: Simulates social media backend (Twitter/Reddit) with database persistence
- Location: `oasis/social_platform/`
- Contains: `Platform` class, `Channel`, database operations, recommendation systems, config
- Depends on: SQLite, Neo4j (optional), sentence-transformers, PyTorch
- Used by: Environment layer, Agents

**Clock Layer:**
- Purpose: Time management for simulation (real-time vs accelerated)
- Location: `oasis/clock/`
- Contains: `Clock` class with time scaling factor
- Depends on: datetime
- Used by: Platform, Agents

**Testing/Utilities Layer:**
- Purpose: Database inspection and testing utilities
- Location: `oasis/testing/`
- Contains: `print_db_contents()` for debugging
- Used by: Development and debugging

## Data Flow

**Simulation Lifecycle:**

1. **Setup Phase:**
   - User creates `AgentGraph` and populates with `SocialAgent` instances
   - User calls `oasis.make()` to create `OasisEnv`
   - Environment initializes Platform with database and channel

2. **Reset Phase:**
   - `env.reset()` starts platform async task (`platform.running()`)
   - Agents are registered in database via `generate_custom_agents()`

3. **Step Phase:**
   - User provides action dict: `{agent: ManualAction | LLMAction}`
   - `env.step()` updates recommendation table via `update_rec_table()`
   - Actions are dispatched:
     - `ManualAction`: Direct function call via `perform_action_by_data()`
     - `LLMAction`: Async LLM inference via `perform_action_by_llm()` with semaphore
   - Platform processes actions through Channel message passing
   - Results returned to agents

4. **Teardown Phase:**
   - `env.close()` sends EXIT signal to platform
   - Platform closes database connections

**Agent-Platform Communication:**

```
Agent.perform_action() 
  -> SocialAction.perform_action()
    -> Channel.write_to_receive_queue()
      -> Platform.running() receives message
        -> Platform action method executes SQL
          -> Channel.send_to() returns result
    -> Channel.read_from_send_queue() retrieves result
```

## Key Abstractions

**SocialAgent:**
- Purpose: LLM-powered agent that can perform social media actions
- Location: `oasis/social_agent/agent.py`
- Pattern: Extends CAMEL's `ChatAgent` with social environment
- Key methods: `perform_action_by_llm()`, `perform_action_by_data()`, `perform_interview()`

**AgentGraph:**
- Purpose: Manages social network topology (follow relationships)
- Location: `oasis/social_agent/agent_graph.py`
- Pattern: Adapter pattern supporting igraph (local) or Neo4j (distributed)
- Key methods: `add_agent()`, `add_edge()`, `remove_edge()`, `get_agents()`

**Platform:**
- Purpose: Social media backend simulation
- Location: `oasis/social_platform/platform.py`
- Pattern: Event loop with action dispatch via `getattr`
- Key methods: `running()`, `create_post()`, `like_post()`, `follow()`, `update_rec_table()`

**Channel:**
- Purpose: Async message passing between agents and platform
- Location: `oasis/social_platform/channel.py`
- Pattern: Producer-consumer with async queue and safe dictionary
- Components: `receive_queue` (asyncio.Queue), `send_dict` (AsyncSafeDict)

**Recommendation System:**
- Purpose: Content recommendation algorithms
- Location: `oasis/social_platform/recsys.py`
- Pattern: Strategy pattern with multiple implementations
- Types: `rec_sys_random`, `rec_sys_reddit`, `rec_sys_personalized_twh`, `rec_sys_personalized_with_trace`

## Entry Points

**Main Entry Point:**
- Location: `oasis/__init__.py`
- Exports: `make`, `Platform`, `SocialAgent`, `AgentGraph`, `ActionType`, etc.

**Environment Factory:**
- Location: `oasis/environment/make.py`
- Function: `make(*args, **kwargs)` - creates `OasisEnv` instance

**Example Scripts:**
- Location: `examples/quick_start.py`, `examples/twitter_simulation_openai.py`, `examples/reddit_simulation_openai.py`
- Purpose: Demonstrate usage patterns

## Error Handling

**Strategy:** Try-catch with structured error returns

**Patterns:**
- All platform actions return `{"success": bool, ...}` dictionaries
- Exceptions caught and returned as `{"success": False, "error": str(e)}`
- Logging to file handlers in `log/` directory
- Agent errors logged but don't crash simulation

## Cross-Cutting Concerns

**Logging:** 
- Module-specific loggers (`social.agent`, `social.twitter`, `oasis.env`)
- File handlers with timestamps in `log/` directory
- Format: `%(levelname)s - %(asctime)s - %(name)s - %(message)s`

**Database:**
- SQLite3 with WAL mode (`PRAGMA synchronous = OFF`)
- Schema files in `oasis/social_platform/schema/`
- Tables: user, post, follow, like, dislike, comment, trace, rec, etc.

**Concurrency:**
- `asyncio.Semaphore(128)` limits concurrent LLM requests
- Async/await throughout agent and platform layers
- Thread-safe `AsyncSafeDict` for channel message storage

---

*Architecture analysis: 2026-04-12*
