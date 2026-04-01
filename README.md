# nanoagent

这是一个对上游 [GitHubxsy/nanoAgent](https://github.com/GitHubxsy/nanoAgent) 的本地整合仓库。

目标不是重写上游，而是在完整保留其教程结构的前提下，补一层更适合二次学习和归档的本地整理内容。

## 这份整合做了什么

- 保留上游章节目录和代码文件，方便按原教程路径阅读
- 保留上游中英文 README 作为对照资料
- 新增本地中文学习笔记和架构总结
- 明确记录已观察到的上游一致性问题，而不是直接改写原始内容

## 仓库结构

### 上游原始层

- `01-essence/`
- `02-memory/`
- `03-skills-mcp/`
- `04-subagent/`
- `05-teams/`
- `06-compact/`
- `07-safety/`
- `full/`
- `bonus/`
- `nano-skill/`
- `real-mcp/`
- `tech-sharing/`
- `tests/`

### 本地整理层

- `docs/summary/nanoagent-study-notes.zh-CN.md`
- `docs/summary/nanoagent-architecture.zh-CN.md`
- `docs/superpowers/specs/2026-04-01-nanoagent-integration-design.md`
- `docs/superpowers/plans/2026-04-01-nanoagent-integration.md`

## 阅读路径

### 路径一：按上游章节学习

适合想顺着原作者思路，从最小 Agent 一路看到完整集成版的人。

1. `01-essence/`
2. `02-memory/`
3. `03-skills-mcp/`
4. `04-subagent/`
5. `05-teams/`
6. `06-compact/`
7. `07-safety/`
8. `full/`

### 路径二：按本地总结理解

适合想先建立整体心智模型，再回头看具体代码的人。

1. `docs/summary/nanoagent-architecture.zh-CN.md`
2. `docs/summary/nanoagent-study-notes.zh-CN.md`
3. 再回到各章节代码和文章做细读

## 快速运行

先安装依赖：

```bash
pip install -r requirements.txt
```

再设置环境变量：

```bash
export OPENAI_API_KEY="your-key"
export OPENAI_BASE_URL="https://api.openai.com/v1"
export OPENAI_MODEL="gpt-4o-mini"
```

运行示例：

```bash
python 01-essence/agent-essence.py "列出当前目录下所有 Python 文件"
python 02-memory/agent-memory.py "统计代码行数并记住结果"
python full/agent-full.py "重构 hello.py，添加类型注解和单元测试"
```

## 对照资料

- 上游英文导读：`README.upstream.md`
- 上游中文说明：`README.upstream.zh-CN.md`

## 已知事项

- 上游仓库以教学演示为主，不等同于生产级 Agent 框架
- 当前保留了上游测试文件，但已观察到测试与文件树存在不完全一致的情况
- 当前环境里如果没有 `pytest`，测试命令不会直接通过，需要先补安装

## 许可证

沿用上游 `LICENSE`，即 MIT。
