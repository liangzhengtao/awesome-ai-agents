# Awesome AI Agents

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Stars](https://img.shields.io/github/stars/liangzhengtao/awesome-ai-agents?style=social)](https://github.com/liangzhengtao/awesome-ai-agents)

<p align="center">
  <a href="#english">English</a> | <a href="#中文">中文</a>
</p>

---

<a name="english"></a>

> **Stop building basic chatbots. 12 skills to build production-ready AI agents.**

A curated collection of battle-tested skills for building AI agents — from single-agent tool calling to multi-agent orchestration. Each skill is a complete, copy-paste-ready guide with architecture diagrams, code templates, patterns, and pitfalls.

一系列经过实战检验的 AI 智能体构建技能合集——从单智能体工具调用到多智能体编排。每项技能都是完整的、可直接复制使用的指南，包含架构图、代码模板、设计模式和常见陷阱。

---

## Why This Project? / 为什么选择这个项目？

The AI agent ecosystem in 2026 is fragmented — LangChain, AutoGen, CrewAI, MCP, custom solutions — each with different APIs and patterns. This project distills the **core skills** every agent developer needs, independent of framework choice.

2026 年的 AI 智能体生态高度碎片化——LangChain、AutoGen、CrewAI、MCP、自定义方案——每个都有不同的 API 和模式。本项目提炼了每位智能体开发者都需要的**核心技能**，不依赖于特定框架选择。

---

## Skills Table / 技能总览

### Agent Frameworks / 智能体框架

| # | Skill / 技能 | Description / 描述 | Path / 路径 |
|---|-------------|-------------------|------------|
| 1 | **LangChain Agents** | Build agents with LangChain and LangGraph / 使用 LangChain 和 LangGraph 构建智能体 | [`skills/Agent框架/langchain-agents.md`](skills/Agent框架/langchain-agents.md) |
| 2 | **AutoGen Multi-Agent** | Multi-agent conversations with Microsoft AutoGen / 使用微软 AutoGen 构建多智能体对话 | [`skills/Agent框架/autogen-multiagent.md`](skills/Agent框架/autogen-multiagent.md) |
| 3 | **CrewAI Agents** | Role-based agent orchestration with CrewAI / 使用 CrewAI 进行角色化智能体编排 | [`skills/Agent框架/crewai-agents.md`](skills/Agent框架/crewai-agents.md) |
| 4 | **Custom Agents** | Build agents from scratch without frameworks / 从零构建无框架依赖的智能体 | [`skills/Agent框架/custom-agents.md`](skills/Agent框架/custom-agents.md) |

### Tool Use / 工具调用

| # | Skill / 技能 | Description / 描述 | Path / 路径 |
|---|-------------|-------------------|------------|
| 5 | **Function Calling** | Function calling and tool use across providers / 跨模型提供商的函数调用与工具使用 | [`skills/工具调用/function-calling.md`](skills/工具调用/function-calling.md) |
| 6 | **MCP Integration** | Model Context Protocol server integration / MCP 服务器集成 | [`skills/工具调用/mcp-integration.md`](skills/工具调用/mcp-integration.md) |
| 7 | **API Orchestration** | Combining multiple APIs into agent workflows / 将多个 API 组合成智能体工作流 | [`skills/工具调用/api-orchestration.md`](skills/工具调用/api-orchestration.md) |

### Memory Systems / 记忆系统

| # | Skill / 技能 | Description / 描述 | Path / 路径 |
|---|-------------|-------------------|------------|
| 8 | **RAG Memory** | Retrieval-Augmented Generation memory for agents / 基于 RAG 的智能体记忆 | [`skills/记忆系统/rag-memory.md`](skills/记忆系统/rag-memory.md) |
| 9 | **Long-Term Memory** | Knowledge graphs and persistent memory / 知识图谱与持久化记忆 | [`skills/记忆系统/long-term-memory.md`](skills/记忆系统/long-term-memory.md) |

### Multi-Agent / 多智能体协作

| # | Skill / 技能 | Description / 描述 | Path / 路径 |
|---|-------------|-------------------|------------|
| 10 | **Agent Communication** | Protocols for agent-to-agent messaging / 智能体间消息传递协议 | [`skills/多Agent协作/agent-communication.md`](skills/多Agent协作/agent-communication.md) |
| 11 | **Task Decomposition** | Breaking complex tasks into subtasks / 将复杂任务分解为子任务 | [`skills/多Agent协作/task-decomposition.md`](skills/多Agent协作/task-decomposition.md) |
| 12 | **Agent Orchestration** | Coordinating multiple agents in workflows / 在工作流中协调多个智能体 | [`skills/多Agent协作/agent-orchestration.md`](skills/多Agent协作/agent-orchestration.md) |

---

## Quick Start / 快速开始

### Using with Cursor / 在 Cursor 中使用

1. Clone this repo:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. Open Cursor → Settings → Rules → Add `awesome-ai-agents/skills/` as context
3. Ask Cursor: *"Read the langchain-agents skill and build me an agent with web search"*

### Using with Claude Code / 在 Claude Code 中使用

1. Clone this repo:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. In your project, reference the skill:
   ```bash
   # Copy the skill you need
   cp awesome-ai-agents/skills/Agent框架/custom-agents.md .claude/agents/
   ```
3. Or reference directly:
   ```
   Read awesome-ai-agents/skills/工具调用/function-calling.md and implement it in my project
   ```

### Using with Kimi Code / 在 Kimi Code 中使用

1. Clone this repo:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. Add as a skill directory:
   ```bash
   cp -r awesome-ai-agents/skills/ ~/.agents/skills/ai-agents/
   ```
3. Or reference in conversation:
   ```
   Read the skill at awesome-ai-agents/skills/记忆系统/rag-memory.md and set up RAG for my project
   ```

### Using as Reference / 作为参考资料使用

Each skill file is self-contained. Browse the skills directory, pick the one you need, and follow the architecture diagram, code templates, and patterns.

每个技能文件都是自包含的。浏览 `skills/` 目录，选择你需要的技能，按照架构图、代码模板和设计模式操作即可。

---

## What Each Skill Contains / 每项技能包含的内容

Every skill file follows the same structure:

每项技能文件遵循统一结构：

- **When to Use** — Clear decision matrix for when this skill applies / 何时使用——清晰的决策矩阵
- **Architecture** — ASCII diagram showing the system design / 架构——ASCII 图表展示系统设计
- **Code Templates** — Production-ready Python code, copy-paste ready / 代码模板——生产级 Python 代码，可直接复制使用
- **Patterns** — Common design patterns with code / 设计模式——附带代码的常见设计模式
- **Pitfalls** — Things that will bite you (with solutions) / 常见陷阱——会坑你的问题（附解决方案）
- **Best Practices** — Hard-won production wisdom / 最佳实践——来自生产环境的经验总结
- **See Also** — Cross-references to related skills / 相关技能——交叉引用

---

## Learning Path / 学习路径

### Beginner / 初学者
```
Custom Agents → Function Calling → RAG Memory
从零构建智能体 → 函数调用 → RAG 记忆
```

### Intermediate / 中级
```
LangChain Agents → MCP Integration → Long-Term Memory → Task Decomposition
LangChain 智能体 → MCP 集成 → 长期记忆 → 任务分解
```

### Advanced / 高级
```
AutoGen Multi-Agent → Agent Communication → Agent Orchestration → API Orchestration
AutoGen 多智能体 → 智能体通信 → 智能体编排 → API 编排
```

---

## Contributing / 参与贡献

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

欢迎贡献！请参阅 [CONTRIBUTING.md](CONTRIBUTING.md) 了解贡献指南。

---

## License / 许可证

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

本项目采用 MIT 许可证——详情请参阅 [LICENSE](LICENSE)。

---

## See Also / 相关项目

Check out our other `awesome-*` projects:

查看我们的其他 `awesome-*` 项目：

| Project / 项目 | Description / 描述 |
|---------------|-------------------|
| [awesome-ai-agents](https://github.com/liangzhengtao/awesome-ai-agents) | Skills for building AI agents / 构建 AI 智能体的技能 |
| [awesome-llm-apps](https://github.com/liangzhengtao/awesome-llm-apps) | Collection of LLM-powered applications / LLM 驱动的应用集合 |
| [awesome-rag](https://github.com/liangzhengtao/awesome-rag) | RAG techniques and implementations / RAG 技术与实现 |
| [awesome-prompt-engineering](https://github.com/liangzhengtao/awesome-prompt-engineering) | Prompt engineering guides and templates / 提示工程指南与模板 |
| [awesome-ai-coding](https://github.com/liangzhengtao/awesome-ai-coding) | AI-powered coding tools and practices / AI 编程工具与实践 |
| [awesome-gpt-store](https://github.com/liangzhengtao/awesome-gpt-store) | Curated GPT store applications / 精选 GPT 商店应用 |
| [awesome-ai-api](https://github.com/liangzhengtao/awesome-ai-api) | AI API integration guides / AI API 集成指南 |
| [awesome-llm-fine-tuning](https://github.com/liangzhengtao/awesome-llm-fine-tuning) | LLM fine-tuning techniques / LLM 微调技术 |
| [awesome-vector-database](https://github.com/liangzhengtao/awesome-vector-database) | Vector database comparisons and guides / 向量数据库对比与指南 |
| [awesome-ai-safety](https://github.com/liangzhengtao/awesome-ai-safety) | AI safety and alignment resources / AI 安全与对齐资源 |
| [awesome-multimodal-ai](https://github.com/liangzhengtao/awesome-multimodal-ai) | Multimodal AI techniques / 多模态 AI 技术 |
| [awesome-open-source-llm](https://github.com/liangzhengtao/awesome-open-source-llm) | Open-source LLM collection / 开源大模型合集 |

---

<div align="center">

**Built with care for the AI agent community.**

**为 AI 智能体社区用心构建。**

[⬆ Back to top / 回到顶部](#awesome-ai-agents)

</div>

---

<a name="中文"></a>

# 优秀的 AI 智能体技能

> **停止构建基础聊天机器人。12 项技能助你构建生产级 AI 智能体。**

一系列经过实战检验的 AI 智能体构建技能合集——从单智能体工具调用到多智能体编排。每项技能都是完整的、可直接复制使用的指南，包含架构图、代码模板、设计模式和常见陷阱。

---

## 为什么选择这个项目？

2026 年的 AI 智能体生态高度碎片化——LangChain、AutoGen、CrewAI、MCP、自定义方案——每个都有不同的 API 和模式。本项目提炼了每位智能体开发者都需要的**核心技能**，不依赖于特定框架选择。

---

## 技能总览

### 智能体框架

| # | 技能 | 描述 | 路径 |
|---|------|------|------|
| 1 | **LangChain 智能体** | 使用 LangChain 和 LangGraph 构建智能体 | [`skills/Agent框架/langchain-agents.md`](skills/Agent框架/langchain-agents.md) |
| 2 | **AutoGen 多智能体** | 使用微软 AutoGen 构建多智能体对话 | [`skills/Agent框架/autogen-multiagent.md`](skills/Agent框架/autogen-multiagent.md) |
| 3 | **CrewAI 智能体** | 使用 CrewAI 进行角色化智能体编排 | [`skills/Agent框架/crewai-agents.md`](skills/Agent框架/crewai-agents.md) |
| 4 | **自定义智能体** | 从零构建无框架依赖的智能体 | [`skills/Agent框架/custom-agents.md`](skills/Agent框架/custom-agents.md) |

### 工具调用

| # | 技能 | 描述 | 路径 |
|---|------|------|------|
| 5 | **函数调用** | 跨模型提供商的函数调用与工具使用 | [`skills/工具调用/function-calling.md`](skills/工具调用/function-calling.md) |
| 6 | **MCP 集成** | MCP 服务器集成 | [`skills/工具调用/mcp-integration.md`](skills/工具调用/mcp-integration.md) |
| 7 | **API 编排** | 将多个 API 组合成智能体工作流 | [`skills/工具调用/api-orchestration.md`](skills/工具调用/api-orchestration.md) |

### 记忆系统

| # | 技能 | 描述 | 路径 |
|---|------|------|------|
| 8 | **RAG 记忆** | 基于 RAG 的智能体记忆 | [`skills/记忆系统/rag-memory.md`](skills/记忆系统/rag-memory.md) |
| 9 | **长期记忆** | 知识图谱与持久化记忆 | [`skills/记忆系统/long-term-memory.md`](skills/记忆系统/long-term-memory.md) |

### 多智能体协作

| # | 技能 | 描述 | 路径 |
|---|------|------|------|
| 10 | **智能体通信** | 智能体间消息传递协议 | [`skills/多Agent协作/agent-communication.md`](skills/多Agent协作/agent-communication.md) |
| 11 | **任务分解** | 将复杂任务分解为子任务 | [`skills/多Agent协作/task-decomposition.md`](skills/多Agent协作/task-decomposition.md) |
| 12 | **智能体编排** | 在工作流中协调多个智能体 | [`skills/多Agent协作/agent-orchestration.md`](skills/多Agent协作/agent-orchestration.md) |

---

## 快速开始

### 在 Cursor 中使用

1. 克隆仓库：
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. 打开 Cursor → 设置 → 规则 → 添加 `awesome-ai-agents/skills/` 作为上下文
3. 向 Cursor 提问：*"读取 langchain-agents 技能，为我构建一个带网络搜索的智能体"*

### 在 Claude Code 中使用

1. 克隆仓库：
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. 在项目中引用技能：
   ```bash
   # 复制你需要的技能
   cp awesome-ai-agents/skills/Agent框架/custom-agents.md .claude/agents/
   ```
3. 或直接引用：
   ```
   读取 awesome-ai-agents/skills/工具调用/function-calling.md 并在我的项目中实现
   ```

### 在 Kimi Code 中使用

1. 克隆仓库：
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. 添加为技能目录：
   ```bash
   cp -r awesome-ai-agents/skills/ ~/.agents/skills/ai-agents/
   ```
3. 或在对话中引用：
   ```
   读取 awesome-ai-agents/skills/记忆系统/rag-memory.md 并为我的项目设置 RAG
   ```

### 作为参考资料使用

每个技能文件都是自包含的。浏览 `skills/` 目录，选择你需要的技能，按照架构图、代码模板和设计模式操作即可。

---

## 每项技能包含的内容

每项技能文件遵循统一结构：

- **何时使用** — 清晰的决策矩阵
- **架构** — ASCII 图表展示系统设计
- **代码模板** — 生产级 Python 代码，可直接复制使用
- **设计模式** — 附带代码的常见设计模式
- **常见陷阱** — 会坑你的问题（附解决方案）
- **最佳实践** — 来自生产环境的经验总结
- **相关技能** — 交叉引用

---

## 学习路径

### 初学者
```
从零构建智能体 → 函数调用 → RAG 记忆
```

### 中级
```
LangChain 智能体 → MCP 集成 → 长期记忆 → 任务分解
```

### 高级
```
AutoGen 多智能体 → 智能体通信 → 智能体编排 → API 编排
```

---

## 参与贡献

欢迎贡献！请参阅 [CONTRIBUTING.md](CONTRIBUTING.md) 了解贡献指南。

---

## 许可证

本项目采用 MIT 许可证——详情请参阅 [LICENSE](LICENSE)。

---

## 相关项目

查看我们的其他 `awesome-*` 项目：

| 项目 | 描述 |
|------|------|
| [awesome-ai-agents](https://github.com/liangzhengtao/awesome-ai-agents) | 构建 AI 智能体的技能 |
| [awesome-llm-apps](https://github.com/liangzhengtao/awesome-llm-apps) | LLM 驱动的应用集合 |
| [awesome-rag](https://github.com/liangzhengtao/awesome-rag) | RAG 技术与实现 |
| [awesome-prompt-engineering](https://github.com/liangzhengtao/awesome-prompt-engineering) | 提示工程指南与模板 |
| [awesome-ai-coding](https://github.com/liangzhengtao/awesome-ai-coding) | AI 编程工具与实践 |
| [awesome-gpt-store](https://github.com/liangzhengtao/awesome-gpt-store) | 精选 GPT 商店应用 |
| [awesome-ai-api](https://github.com/liangzhengtao/awesome-ai-api) | AI API 集成指南 |
| [awesome-llm-fine-tuning](https://github.com/liangzhengtao/awesome-llm-fine-tuning) | LLM 微调技术 |
| [awesome-vector-database](https://github.com/liangzhengtao/awesome-vector-database) | 向量数据库对比与指南 |
| [awesome-ai-safety](https://github.com/liangzhengtao/awesome-ai-safety) | AI 安全与对齐资源 |
| [awesome-multimodal-ai](https://github.com/liangzhengtao/awesome-multimodal-ai) | 多模态 AI 技术 |
| [awesome-open-source-llm](https://github.com/liangzhengtao/awesome-open-source-llm) | 开源大模型合集 |

---

<div align="center">

**为 AI 智能体社区用心构建。**

[⬆ 回到顶部](#awesome-ai-agents)

</div>
