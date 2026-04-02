# nanoagent

[English](./README.md)

`nanoagent` 是一个按源码递进组织的 Agent 学习仓库。它的价值不在于“拿来即上生产”，而在于把 Agent 的关键能力拆成一层层可阅读、可运行、可对照的 Python 实现。

这个仓库适合两种读法：

- 顺着章节推进，直接看能力是怎么一层层长出来的
- 先读 [`docs/integration-thinking/`](./docs/integration-thinking/)，先建立整体判断，再回到源码细节

如果你想理解一个 Agent 为什么需要循环、记忆、委派、协作、压缩和安全边界，这个仓库已经足够清楚。如果你想直接把它当成成熟框架，它还不完整。

## 阅读路径

### 路线 A：按实现路径读源码

如果你希望沿着“功能是怎么长出来的”这条线阅读，建议顺序是：

1. `01-essence`
2. `02-memory`
3. `03-skills-mcp`
4. `04-subagent`
5. `05-teams`
6. `06-compact`
7. `07-safety`
8. `full`

### 路线 B：先看全景，再回到章节

如果你已经熟悉基本概念，更适合先看：

1. `full/agent-full.py`
2. [`docs/integration-thinking/README.md`](./docs/integration-thinking/README.md)
3. 再回到各章节拆开看每一层能力

## 仓库结构

### 核心脚本

- `01-essence/agent-essence.py`：最小 Agent 循环与工具调用
- `02-memory/agent-memory.py`：持久记忆与可选规划
- `03-skills-mcp/agent-skills-mcp.py`：Rules、Skills、MCP 配置加载，以及 plan-as-tool
- `04-subagent/agent-subagent.py`：通过可调用的子智能体完成委派
- `05-teams/agent-teams.py`：持久多智能体协作与收件箱式通信
- `06-compact/agent-compact.py`：长任务场景下的上下文压缩
- `07-safety/agent-safe.py`：命令过滤、人工确认、输出截断
- `full/agent-full.py`：把整条能力链收拢到一个文件中

### 配套文档

- `01-essence/agent-essence.md` 到 `07-safety/agent-safe.md`：逐章解释源码和设计取舍
- `tech-sharing/tech-sharing.md`：整套内容的 Markdown 版分享稿
- [`docs/integration-thinking/README.md`](./docs/integration-thinking/README.md)：整合思考目录
- [`docs/integration-thinking/chapter-notes.zh-CN.md`](./docs/integration-thinking/chapter-notes.zh-CN.md)：按章节拆解的学习笔记
- [`docs/integration-thinking/architecture.zh-CN.md`](./docs/integration-thinking/architecture.zh-CN.md)：从整体能力演进角度做的架构总结

### 辅助目录

- `bonus/`：补充模式，例如 commands 和 preset agents
- `real-mcp/`：一个最小 HTTP MCP 示例
- `nano-skill/`：Skill 相关说明与示例
- `tests/`：小型示例测试和遗留测试资产

## 每一阶段到底增加了什么

### 1. 最小循环

`01-essence/agent-essence.py` 只定义了 3 个工具：

- `execute_bash`
- `read_file`
- `write_file`

核心循环是：

1. 把消息和工具 schema 发给模型
2. 接收 tool call 或最终回答
3. 在 Python 里执行工具
4. 把工具结果追加回消息历史
5. 重复直到模型停止

这是整套仓库最底层的运行模式。

### 2. 记忆与规划

`02-memory/agent-memory.py` 增加了：

- `agent_memory.md` 持久化
- `save_memory()` 和 `load_memory()`
- `create_plan()` 负责 3 到 5 步拆解
- `run_agent_step()` 和 `run_agent_plus()`

从这一版开始，Agent 不再只是单轮执行器，而是可以跨任务保留上下文，并在需要时先做任务拆分。

### 3. Rules、Skills 与 MCP

`03-skills-mcp/agent-skills-mcp.py` 会读取这些外部配置：

- `.agent/rules/*.md`
- `.agent/skills/*.json`
- `.agent/mcp.json`

基础工具也扩展为：

- `read`
- `write`
- `edit`
- `glob`
- `grep`
- `bash`
- `plan`

这一章是仓库从“单脚本示例”走向“可配置 harness”的转折点。

### 4. Subagent

`04-subagent/agent-subagent.py` 引入 `subagent(role, task)`。

被委派出去的子智能体拥有：

- 独立的 system prompt
- 独立的消息历史
- 去掉递归 `subagent` 的受限工具集

这里解决的是“复杂任务如何分出去做”，还不是“团队如何长期协作”。

### 5. Teams

`05-teams/agent-teams.py` 把临时委派升级成持久协作，新增了：

- 带持久 `messages` 的 `Agent`
- 负责 `hire`、`send`、`broadcast`、`disband` 的 `Team`
- 基于 inbox 的智能体间通信
- 一个简单的团队规划流程，会生成 2 到 4 个成员

### 6. 上下文压缩

`06-compact/agent-compact.py` 通过摘要保住长任务的可持续执行。

关键常量：

- `COMPACT_THRESHOLD = 20`
- `KEEP_RECENT = 6`

消息列表过长时，旧消息会被压成摘要，只保留 system message、摘要消息对，以及最近若干条消息。

### 7. 安全

`07-safety/agent-safe.py` 加了三道实际可用的约束：

- 基于正则的危险命令过滤
- 读、写、执行前的人类确认
- `MAX_OUTPUT_LENGTH = 5000` 的输出截断

这一版依然很轻，但已经开始把“能执行”升级为“受约束地执行”。

### Full 集成版

`full/agent-full.py` 把前面章节的能力收拢进一个脚本：

- 文件与 shell 工具
- memory
- rules、skills、MCP
- subagents
- team mode
- compaction
- safety hooks

常见入口：

```bash
python full/agent-full.py "你的任务"
python full/agent-full.py --auto "你的任务"
python full/agent-full.py --team "你的任务"
```

## 安装

当前最小依赖只有：

```bash
pip install -r requirements.txt
```

`requirements.txt` 目前内容是：

```text
openai
```

运行任一章节前，至少需要设置这些环境变量：

```bash
export OPENAI_API_KEY="your-key"
export OPENAI_BASE_URL="https://api.openai.com/v1"
export OPENAI_MODEL="gpt-4o-mini"
```

## 运行示例

### 最小 Agent

```bash
python 01-essence/agent-essence.py "列出当前目录下所有 Python 文件"
```

### 记忆与规划

```bash
python 02-memory/agent-memory.py --plan "分析当前仓库并写一份简短总结"
```

### Rules、Skills 与 MCP

```bash
python 03-skills-mcp/agent-skills-mcp.py --plan "搜索 TODO 并总结发现"
```

### Subagent 委派

```bash
python 04-subagent/agent-subagent.py "创建一个 TODO 应用，包含 Python 后端和 HTML 前端"
```

### Team 协作

```bash
python 05-teams/agent-teams.py "创建一个 TODO 应用，包含 Python 后端和 HTML 前端"
```

### 上下文压缩

```bash
python 06-compact/agent-compact.py "找到所有 Python 文件，统计行数，按行数排序，写入 report.txt"
```

### 安全版 Agent

```bash
python 07-safety/agent-safe.py "列出当前目录文件"
```

## 本地配置文件

部分章节会读取这些可选本地文件：

- `.agent/rules/*.md`
- `.agent/skills/*.json`
- `.agent/mcp.json`
- `agent_memory.md`

它们不存在时，代码通常会回退到默认行为继续执行。

## 测试与验证说明

当前仓库状态需要注意这几点：

- `tests/test_compact.py` 和 `tests/test_subagent.py` 是基于 mock 的示例测试
- `tests/test_agent.py` 依赖 `agent.py` 和 `agent-plus.py`，但这两个文件当前不在仓库中
- 当前环境没有安装 `pytest`，因此 `python3 -m pytest -q tests` 不能直接运行

## 适用边界

这个仓库很适合理解 Agent 的结构，但没有试图完整覆盖生产环境的需求，例如：

- 长任务失败后的恢复与重试
- 工具执行的强隔离
- 动作审计与回放
- 更细粒度的权限模型
- 测试和源码的一致性治理

## License

MIT，见 [LICENSE](./LICENSE)。
