# nanoagent

[English](./README.md)

`nanoagent` 是一个按源码递进组织的 Agent 学习仓库，核心价值不是“直接上线”，而是把 Agent 的能力拆成一层层能读懂、能运行、能对照的 Python 实现。

仓库主线分成 7 个阶段，再加一个 `full` 集成版：

- `01-essence`：最小 Agent 循环与工具调用
- `02-memory`：持久记忆与可选规划
- `03-skills-mcp`：Rules、Skills、MCP 配置加载，以及 plan-as-tool
- `04-subagent`：把 subagent 作为一个工具来调用
- `05-teams`：持久多智能体协作与收件箱通信
- `06-compact`：长上下文压缩
- `07-safety`：命令黑名单、人工确认、输出截断
- `full`：把整条能力链集成到一个文件里

它不是生产级框架，而是一套足够小、足够清楚的源码样本，用来理解 Agent 能力是怎么一层层长出来的。

## 仓库结构

### 核心章节

- `01-essence/agent-essence.py`
- `02-memory/agent-memory.py`
- `03-skills-mcp/agent-skills-mcp.py`
- `04-subagent/agent-subagent.py`
- `05-teams/agent-teams.py`
- `06-compact/agent-compact.py`
- `07-safety/agent-safe.py`
- `full/agent-full.py`

### 配套内容

- 每个章节旁边的 `*.md` 用来解释该章节的设计和源码
- `bonus/` 放了一些额外模式，例如 command 和 preset agent
- `real-mcp/` 放了一个最小 HTTP MCP 示例
- `nano-skill/` 放了 Skill 相关的说明和例子
- `tech-sharing/tech-sharing.md` 是偏分享稿风格的总览材料

### 测试

- `tests/test_compact.py`
- `tests/test_subagent.py`
- `tests/test_agent.py`

其中 `test_compact.py` 和 `test_subagent.py` 是基于 mock 的示例测试。`tests/test_agent.py` 依赖 `agent.py` 和 `agent-plus.py`，但这两个文件当前不在仓库里，所以这份测试更像遗留回归资产，不适合直接当成稳定通过的测试入口。

## 各阶段到底加了什么

### 1. 最小循环

`01-essence/agent-essence.py` 只有 3 个工具：

- `execute_bash`
- `read_file`
- `write_file`

核心循环就是：

1. 把消息和工具 schema 发给模型
2. 接收 tool call 或最终回答
3. 在 Python 里执行工具
4. 把工具输出追加回消息历史
5. 重复直到模型停止

这是整套仓库最底层的模式。

### 2. 记忆与规划

`02-memory/agent-memory.py` 增加了：

- `agent_memory.md` 持久化
- `save_memory()` / `load_memory()`
- `create_plan()` 生成 3-5 步拆解
- `run_agent_step()` / `run_agent_plus()`

从这一版开始，Agent 能记住之前做过什么，也能通过 `--plan` 先拆任务再执行。

### 3. Rules、Skills、MCP

`03-skills-mcp/agent-skills-mcp.py` 会读取：

- `.agent/rules/*.md`
- `.agent/skills/*.json`
- `.agent/mcp.json`

基础工具也扩成了：

- `read`
- `write`
- `edit`
- `glob`
- `grep`
- `bash`
- `plan`

这一版已经很接近一个小型的 Claude Code 风格 harness：本地规则、本地技能描述、MCP 工具 schema 都能装进上下文。

### 4. Subagent

`04-subagent/agent-subagent.py` 引入了 `subagent(role, task)`。

它的特点是：

- 有独立 system prompt
- 有独立消息历史
- 工具列表里排除了递归 `subagent`

这是“临时委派”的模型，不是“持久团队”的模型。

### 5. Teams

`05-teams/agent-teams.py` 把一次性委派升级成持久协作，新增了：

- 带持久 `messages` 的 `Agent`
- 负责 `hire / send / broadcast / disband` 的 `Team`
- 基于 inbox 的智能体间通信
- 一个简单的团队规划流程，会创建 2-4 个成员

### 6. 上下文压缩

`06-compact/agent-compact.py` 通过摘要来防止长任务把上下文撑爆。

关键常量：

- `COMPACT_THRESHOLD = 20`
- `KEEP_RECENT = 6`

消息过长时，会把旧消息压成摘要，只保留：

- system message
- 摘要消息对
- 最近的若干条消息

### 7. 安全

`07-safety/agent-safe.py` 增加了三道约束：

- 基于正则的危险命令黑名单
- 执行前人工确认
- `MAX_OUTPUT_LENGTH = 5000` 的输出截断

它依然很简化，但已经开始认真限制工具执行边界。

### Full 集成版

`full/agent-full.py` 把前面章节能力收拢到一个文件里：

- 文件与 shell 工具
- memory
- rules / skills / MCP
- subagent
- team mode
- compaction
- safety hooks

常用入口：

- `python full/agent-full.py "你的任务"`
- `python full/agent-full.py --auto "你的任务"`
- `python full/agent-full.py --team "你的任务"`

## 安装

当前声明的依赖只有：

```bash
pip install -r requirements.txt
```

`requirements.txt` 目前只有一项：

```txt
openai
```

运行任一章节前需要设置环境变量：

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

### 记忆 + 规划

```bash
python 02-memory/agent-memory.py --plan "分析当前仓库并写一份简短总结"
```

### Rules / Skills / MCP

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

### 完整版

```bash
python full/agent-full.py "重构 hello.py，添加测试"
```

## 本地配置文件

有些章节会读取可选本地文件：

- `.agent/rules/*.md`
- `.agent/skills/*.json`
- `.agent/mcp.json`
- `agent_memory.md`

这些文件不存在时，大多数代码会回退到默认行为继续执行。

## 验证说明

当前本地确认过的事实：

- 仓库结构和章节递进关系与源码一致
- 顶层入口脚本都存在
- README 现在是按源码能力重写的，而不是沿用之前的整合说明

当前环境里没有完整确认的部分：

- 没有安装 `pytest`，所以 `python3 -m pytest -q tests` 不能直接运行
- `tests/test_agent.py` 依赖当前仓库里不存在的文件

## 推荐阅读顺序

如果你想最快理解源码：

1. `01-essence`
2. `02-memory`
3. `03-skills-mcp`
4. `04-subagent`
5. `05-teams`
6. `06-compact`
7. `07-safety`
8. `full`

如果你想最快看到完整能力：

1. 先读 `full/agent-full.py`
2. 再回头拆各章节看每个能力是怎么加进来的

## License

MIT，见 [LICENSE](./LICENSE)。
