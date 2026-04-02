# nanoagent

[中文说明](./README_CN.md)

`nanoagent` is a source-first learning repository for studying how agent capabilities accumulate in small Python programs.

It is useful for two kinds of reading:

- a chapter-by-chapter code path, from minimal tool calling to safety controls
- a parallel set of local notes under [`docs/integration-thinking/`](./docs/integration-thinking/) for readers who want the integrated view first

This repository is not a production framework. It is a compact reference for understanding what an agent actually needs in order to act, remember, delegate, coordinate, compress context, and stay inside basic guardrails.

## Reading Paths

### Path A: follow the implementation path

Read in order if you want to see one capability added at a time:

1. `01-essence`
2. `02-memory`
3. `03-skills-mcp`
4. `04-subagent`
5. `05-teams`
6. `06-compact`
7. `07-safety`
8. `full`

### Path B: start from the full picture

If you already know the basics, start here:

1. read `full/agent-full.py`
2. read the integration notes in [`docs/integration-thinking/`](./docs/integration-thinking/)
3. return to individual chapters for the implementation details

## Repository Map

### Core scripts

- `01-essence/agent-essence.py`: minimal agent loop with tool calling
- `02-memory/agent-memory.py`: persistent memory and optional planning
- `03-skills-mcp/agent-skills-mcp.py`: rules, skills, MCP config loading, and plan-as-tool
- `04-subagent/agent-subagent.py`: delegation through a callable subagent tool
- `05-teams/agent-teams.py`: persistent multi-agent collaboration with inbox-style messaging
- `06-compact/agent-compact.py`: context compaction for long-running work
- `07-safety/agent-safe.py`: command filtering, confirmation, and output truncation
- `full/agent-full.py`: single-file integration of the full stack

### Companion documents

- `01-essence/agent-essence.md` to `07-safety/agent-safe.md`: chapter-by-chapter code explanations
- `tech-sharing/tech-sharing.md`: long-form Markdown talk track for the whole series
- [`docs/integration-thinking/README.md`](./docs/integration-thinking/README.md): index of local integration notes
- [`docs/integration-thinking/chapter-notes.zh-CN.md`](./docs/integration-thinking/chapter-notes.zh-CN.md): chapter-based study notes
- [`docs/integration-thinking/architecture.zh-CN.md`](./docs/integration-thinking/architecture.zh-CN.md): architecture summary of the full progression

### Supporting directories

- `bonus/`: extra patterns such as commands and preset agents
- `real-mcp/`: minimal HTTP-based MCP example
- `nano-skill/`: notes and examples focused on skills
- `tests/`: small examples plus legacy test artifacts

## What Each Stage Adds

### 1. Minimal loop

`01-essence/agent-essence.py` defines three tools:

- `execute_bash`
- `read_file`
- `write_file`

The essential loop is:

1. send messages and tool schemas to the model
2. receive either tool calls or a final answer
3. execute the requested tools in Python
4. append tool outputs back into the conversation
5. repeat until the model stops

### 2. Memory and planning

`02-memory/agent-memory.py` adds:

- `agent_memory.md` persistence
- `save_memory()` and `load_memory()`
- `create_plan()` for 3-5 step decomposition
- `run_agent_step()` and `run_agent_plus()`

This is the first point where the agent can carry work across runs and optionally decompose a larger task before execution.

### 3. Rules, skills, and MCP

`03-skills-mcp/agent-skills-mcp.py` loads external configuration from:

- `.agent/rules/*.md`
- `.agent/skills/*.json`
- `.agent/mcp.json`

It also expands the base tools to:

- `read`
- `write`
- `edit`
- `glob`
- `grep`
- `bash`
- `plan`

This chapter is the shift from a single script to a configurable harness.

### 4. Subagents

`04-subagent/agent-subagent.py` introduces `subagent(role, task)`.

The delegated agent runs with:

- its own system prompt
- its own message history
- a restricted tool list that excludes recursive `subagent`

This is explicit task delegation, not yet a persistent team model.

### 5. Teams

`05-teams/agent-teams.py` upgrades temporary delegation into durable collaboration.

It introduces:

- `Agent` objects with persistent `messages`
- `Team` methods for `hire`, `send`, `broadcast`, and `disband`
- inbox-style communication between agents
- a simple planning step that creates 2-4 team members

### 6. Context compaction

`06-compact/agent-compact.py` keeps long tasks alive by summarizing old messages.

Important constants:

- `COMPACT_THRESHOLD = 20`
- `KEEP_RECENT = 6`

When the message list grows too large, old history is summarized and only the system message, one summary pair, and recent messages are retained.

### 7. Safety

`07-safety/agent-safe.py` adds three practical guardrails:

- dangerous command filtering via regex patterns
- explicit user confirmation before read, write, and bash execution
- output truncation with `MAX_OUTPUT_LENGTH = 5000`

This is still lightweight, but it is the first version that treats tool execution as something that must be constrained.

### Full integration

`full/agent-full.py` combines the chapter features into one script:

- file and shell tools
- memory
- rules, skills, and MCP loading
- subagents
- team mode
- compaction
- safety hooks

Typical entry points:

```bash
python full/agent-full.py "your task"
python full/agent-full.py --auto "your task"
python full/agent-full.py --team "your task"
```

## Installation

Install the minimal dependency set:

```bash
pip install -r requirements.txt
```

`requirements.txt` currently contains:

```text
openai
```

Set environment variables before running any chapter:

```bash
export OPENAI_API_KEY="your-key"
export OPENAI_BASE_URL="https://api.openai.com/v1"
export OPENAI_MODEL="gpt-4o-mini"
```

## Run Examples

### Minimal agent

```bash
python 01-essence/agent-essence.py "list all Python files in the current directory"
```

### Memory and planning

```bash
python 02-memory/agent-memory.py --plan "analyze this repository and write a short summary"
```

### Rules, skills, and MCP

```bash
python 03-skills-mcp/agent-skills-mcp.py --plan "search for TODOs and summarize findings"
```

### Subagent delegation

```bash
python 04-subagent/agent-subagent.py "create a TODO app with a Python backend and HTML frontend"
```

### Team workflow

```bash
python 05-teams/agent-teams.py "create a TODO app with a Python backend and HTML frontend"
```

### Context compaction

```bash
python 06-compact/agent-compact.py "find all Python files, count lines, sort by line count, and write report.txt"
```

### Safety-enabled agent

```bash
python 07-safety/agent-safe.py "list files in the current directory"
```

## Local Configuration Files

Some chapters optionally read local files such as:

- `.agent/rules/*.md`
- `.agent/skills/*.json`
- `.agent/mcp.json`
- `agent_memory.md`

If these files do not exist, the code usually falls back cleanly and continues.

## Tests and Validation Notes

Current repository state:

- `tests/test_compact.py` and `tests/test_subagent.py` are mock-driven examples
- `tests/test_agent.py` references `agent.py` and `agent-plus.py`, which are not present in the current tree
- `pytest` is not installed in the current environment, so `python3 -m pytest -q tests` is not runnable here without extra setup

## Boundaries

This repository explains the shape of an agent system well, but it does not try to solve several production concerns:

- durable recovery and retries
- strong isolation for tool execution
- auditability and replay
- full permission modeling
- complete test-to-source consistency

## License

MIT. See [LICENSE](./LICENSE).
