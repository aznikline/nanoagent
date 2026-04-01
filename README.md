# nanoagent

[中文](./README_CN.md)

`nanoagent` is a source-first learning repository about building agents step by step in Python.

The codebase is organized as a progression:

- `01-essence`: minimal agent loop with tool calling
- `02-memory`: persistent memory and optional planning
- `03-skills-mcp`: rules, skills, MCP config loading, and plan-as-tool
- `04-subagent`: subagent as a callable tool
- `05-teams`: persistent multi-agent collaboration with inbox-style messaging
- `06-compact`: context compaction for long-running conversations
- `07-safety`: command blacklist, confirmation, and output truncation
- `full`: one-file integration of the full stack

It is not a production framework. It is a compact codebase for understanding how agent capabilities accumulate over time.

## Repository Layout

### Core chapters

- `01-essence/agent-essence.py`
- `02-memory/agent-memory.py`
- `03-skills-mcp/agent-skills-mcp.py`
- `04-subagent/agent-subagent.py`
- `05-teams/agent-teams.py`
- `06-compact/agent-compact.py`
- `07-safety/agent-safe.py`
- `full/agent-full.py`

### Supporting material

- `*.md` files beside each chapter explain the design and code
- `bonus/` contains extra agent patterns such as commands and preset agents
- `real-mcp/` contains a minimal HTTP-based MCP example
- `nano-skill/` contains skill-focused notes and examples
- `tech-sharing/tech-sharing.md` is a long-form presentation-style summary

### Tests

- `tests/test_compact.py`
- `tests/test_subagent.py`
- `tests/test_agent.py`

The compact and subagent tests are mock-driven examples. `tests/test_agent.py` references `agent.py` and `agent-plus.py`, which are not present in the current tree, so treat that file as a regression artifact rather than a reliable green test target.

## What Each Stage Adds

### 1. Minimal loop

`01-essence/agent-essence.py` defines three tools:

- `execute_bash`
- `read_file`
- `write_file`

The agent loop is the essential pattern:

1. send messages and tool schemas to the model
2. receive tool calls or a final answer
3. execute tools in Python
4. append tool outputs back into the conversation
5. repeat until the model stops

### 2. Memory and planning

`02-memory/agent-memory.py` adds:

- `agent_memory.md` persistence
- `save_memory()` and `load_memory()`
- `create_plan()` for 3-5 step decomposition
- `run_agent_step()` and `run_agent_plus()`

This is the first point where the agent can remember prior work and optionally break tasks into multiple steps with `--plan`.

### 3. Rules, skills, MCP

`03-skills-mcp/agent-skills-mcp.py` expands the toolset and loads external configuration from:

- `.agent/rules/*.md`
- `.agent/skills/*.json`
- `.agent/mcp.json`

It adds these base tools:

- `read`
- `write`
- `edit`
- `glob`
- `grep`
- `bash`
- `plan`

This version is the closest to a small Claude Code style harness: it loads local rules, local skill descriptors, and MCP tool schemas.

### 4. Subagents

`04-subagent/agent-subagent.py` introduces `subagent(role, task)`, where delegation itself becomes a tool.

The subagent runs with:

- its own system prompt
- its own message history
- a restricted tool list that excludes recursive `subagent`

This is a focused delegation model, not a persistent team model.

### 5. Teams

`05-teams/agent-teams.py` moves from temporary delegation to persistent collaboration.

It introduces:

- `Agent` objects with durable `messages`
- `Team` for hire / send / broadcast / disband
- inbox-style communication between agents
- a simple planning step that creates 2-4 team members

### 6. Context compaction

`06-compact/agent-compact.py` keeps long tasks alive by summarizing old messages.

Important constants:

- `COMPACT_THRESHOLD = 20`
- `KEEP_RECENT = 6`

When the message list grows too large, it summarizes old conversation history and keeps only:

- the system message
- a summary pair
- the most recent messages

### 7. Safety

`07-safety/agent-safe.py` adds three practical safeguards:

- dangerous command blacklist via regex patterns
- explicit user confirmation before read / write / bash execution
- output truncation with `MAX_OUTPUT_LENGTH = 5000`

This is still simple, but it is the first version that seriously constrains tool execution.

### Full integration

`full/agent-full.py` combines the chapter features:

- file and shell tools
- memory
- rules / skills / MCP loading
- subagents
- team mode
- compaction
- safety hooks

Entry examples:

- `python full/agent-full.py "your task"`
- `python full/agent-full.py --auto "your task"`
- `python full/agent-full.py --team "your task"`

## Installation

The only declared dependency is:

```bash
pip install -r requirements.txt
```

`requirements.txt` currently contains:

```txt
openai
```

Set environment variables before running any chapter:

```bash
export OPENAI_API_KEY="your-key"
export OPENAI_BASE_URL="https://api.openai.com/v1"
export OPENAI_MODEL="gpt-4o-mini"
```

## Running The Code

### Minimal agent

```bash
python 01-essence/agent-essence.py "list all Python files in the current directory"
```

### Memory + planning

```bash
python 02-memory/agent-memory.py --plan "analyze this repository and write a short summary"
```

### Rules / skills / MCP

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

### Full integrated agent

```bash
python full/agent-full.py "refactor hello.py and add tests"
```

## Local Configuration Files

Some chapters expect optional local files:

- `.agent/rules/*.md`
- `.agent/skills/*.json`
- `.agent/mcp.json`
- `agent_memory.md`

If they do not exist, the code generally falls back cleanly and continues without them.

## Validation Notes

What was verified locally:

- the repository structure matches the chapter progression above
- the top-level scripts are present
- the README now reflects actual source files instead of prior integration wording

What was not fully verified in this environment:

- `pytest` is not installed, so `python3 -m pytest -q tests` does not run here
- `tests/test_agent.py` targets files that are not present in the current repository state

## Recommended Reading Order

If you want the shortest path to understanding the code:

1. `01-essence`
2. `02-memory`
3. `03-skills-mcp`
4. `04-subagent`
5. `05-teams`
6. `06-compact`
7. `07-safety`
8. `full`

If you want the fastest path to the most complete implementation:

1. `full/agent-full.py`
2. then backtrack into the chapter files for individual concepts

## License

MIT. See [LICENSE](./LICENSE).
