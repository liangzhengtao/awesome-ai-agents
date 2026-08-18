[中文版](README.zh.md)

# Awesome AI Agents

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Stars](https://img.shields.io/github/stars/liangzhengtao/awesome-ai-agents?style=social)](https://github.com/liangzhengtao/awesome-ai-agents)

---

> **Stop building basic chatbots. 12 skills to build production-ready AI agents.**

A curated collection of battle-tested skills for building AI agents — from single-agent tool calling to multi-agent orchestration. Each skill is a complete, copy-paste-ready guide with architecture diagrams, code templates, patterns, and pitfalls.

---

## Why This Project?

The AI agent ecosystem in 2026 is fragmented — LangChain, AutoGen, CrewAI, MCP, custom solutions — each with different APIs and patterns. This project distills the **core skills** every agent developer needs, independent of framework choice.

---

## Skills Table

### Agent Frameworks

| # | Skill | Description | Path |
|---|-------|-------------|------|
| 1 | **LangChain Agents** | Build agents with LangChain and LangGraph | [`skills/Agent框架/langchain-agents.md`](skills/Agent框架/langchain-agents.md) |
| 2 | **AutoGen Multi-Agent** | Multi-agent conversations with Microsoft AutoGen | [`skills/Agent框架/autogen-multiagent.md`](skills/Agent框架/autogen-multiagent.md) |
| 3 | **CrewAI Agents** | Role-based agent orchestration with CrewAI | [`skills/Agent框架/crewai-agents.md`](skills/Agent框架/crewai-agents.md) |
| 4 | **Custom Agents** | Build agents from scratch without frameworks | [`skills/Agent框架/custom-agents.md`](skills/Agent框架/custom-agents.md) |

### Tool Use

| # | Skill | Description | Path |
|---|-------|-------------|------|
| 5 | **Function Calling** | Function calling and tool use across providers | [`skills/工具调用/function-calling.md`](skills/工具调用/function-calling.md) |
| 6 | **MCP Integration** | Model Context Protocol server integration | [`skills/工具调用/mcp-integration.md`](skills/工具调用/mcp-integration.md) |
| 7 | **API Orchestration** | Combining multiple APIs into agent workflows | [`skills/工具调用/api-orchestration.md`](skills/工具调用/api-orchestration.md) |

### Memory Systems

| # | Skill | Description | Path |
|---|-------|-------------|------|
| 8 | **RAG Memory** | Retrieval-Augmented Generation memory for agents | [`skills/记忆系统/rag-memory.md`](skills/记忆系统/rag-memory.md) |
| 9 | **Long-Term Memory** | Knowledge graphs and persistent memory | [`skills/记忆系统/long-term-memory.md`](skills/记忆系统/long-term-memory.md) |

### Multi-Agent

| # | Skill | Description | Path |
|---|-------|-------------|------|
| 10 | **Agent Communication** | Protocols for agent-to-agent messaging | [`skills/多Agent协作/agent-communication.md`](skills/多Agent协作/agent-communication.md) |
| 11 | **Task Decomposition** | Breaking complex tasks into subtasks | [`skills/多Agent协作/task-decomposition.md`](skills/多Agent协作/task-decomposition.md) |
| 12 | **Agent Orchestration** | Coordinating multiple agents in workflows | [`skills/多Agent协作/agent-orchestration.md`](skills/多Agent协作/agent-orchestration.md) |

---

## Quick Start

### Using with Cursor

1. Clone this repo:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. Open Cursor → Settings → Rules → Add `awesome-ai-agents/skills/` as context
3. Ask Cursor: *"Read the langchain-agents skill and build me an agent with web search"*

### Using with Claude Code

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

### Using with Kimi Code

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

### Using as Reference

Each skill file is self-contained. Browse the skills directory, pick the one you need, and follow the architecture diagram, code templates, and patterns.

---

## What Each Skill Contains

Every skill file follows the same structure:

- **When to Use** — Clear decision matrix for when this skill applies
- **Architecture** — ASCII diagram showing the system design
- **Code Templates** — Production-ready Python code, copy-paste ready
- **Patterns** — Common design patterns with code
- **Pitfalls** — Things that will bite you (with solutions)
- **Proven Patterns** — Hard-won production wisdom
- **See Also** — Cross-references to related skills

---

## Learning Path

### Beginner
```
Custom Agents → Function Calling → RAG Memory
```

### Intermediate
```
LangChain Agents → MCP Integration → Long-Term Memory → Task Decomposition
```

### Advanced
```
AutoGen Multi-Agent → Agent Communication → Agent Orchestration → API Orchestration
```

---

## Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

---

## See Also

Check out our other `awesome-*` projects:

| Project | Description |
|---------|-------------|
| [awesome-ai-agents](https://github.com/liangzhengtao/awesome-ai-agents) | Skills for building AI agents |
| [awesome-llm-apps](https://github.com/liangzhengtao/awesome-llm-apps) | Collection of LLM-powered applications |
| [awesome-rag](https://github.com/liangzhengtao/awesome-rag) | RAG techniques and implementations |
| [awesome-prompt-engineering](https://github.com/liangzhengtao/awesome-prompt-engineering) | Prompt engineering guides and templates |
| [awesome-ai-coding](https://github.com/liangzhengtao/awesome-ai-coding) | AI-powered coding tools and practices |
| [awesome-gpt-store](https://github.com/liangzhengtao/awesome-gpt-store) | Curated GPT store applications |
| [awesome-ai-api](https://github.com/liangzhengtao/awesome-ai-api) | AI API integration guides |
| [awesome-llm-fine-tuning](https://github.com/liangzhengtao/awesome-llm-fine-tuning) | LLM fine-tuning techniques |
| [awesome-vector-database](https://github.com/liangzhengtao/awesome-vector-database) | Vector database comparisons and guides |
| [awesome-ai-safety](https://github.com/liangzhengtao/awesome-ai-safety) | AI safety and alignment resources |
| [awesome-multimodal-ai](https://github.com/liangzhengtao/awesome-multimodal-ai) | Multimodal AI techniques |
| [awesome-open-source-llm](https://github.com/liangzhengtao/awesome-open-source-llm) | Open-source LLM collection |

---

<div align="center">

**Built with care for the AI agent community.**

[⬆ Back to top](#awesome-ai-agents)

</div>
