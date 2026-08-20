[English](README.md) | [中文](README.zh.md) | [日本語](README.ja.md) | [Français](README.fr.md) | [Español](README.es.md) | [العربية](README.ar.md) | [한국어](README.ko.md) | [Português](README.pt.md) | [Русский](README.ru.md) | [Deutsch](README.de.md)

<div align="center">

<img src=".banner.svg" width="100%" alt="banner">

</div>


# Awesome AI Agents

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Stars](https://img.shields.io/github/stars/liangzhengtao/awesome-ai-agents?style=social)](https://github.com/liangzhengtao/awesome-ai-agents)

---

> **Pare de construir chatbots básicos. 12 habilidades para construir agentes de IA em produção.**

Uma coleção curada de habilidades comprovadas para construir agentes de IA — desde chamadas de ferramentas em agente único até orquestração multi-agente. Cada habilidade é um guia completo, pronto para copiar e colar, com diagramas de arquitetura, templates de código, padrões e armadilhas.

---

## Por que este Projeto?

O ecossistema de agentes de IA em 2026 é fragmentado — LangChain, AutoGen, CrewAI, MCP, soluções personalizadas — cada um com APIs e padrões diferentes. Este projeto destila as **habilidades essenciais** que todo desenvolvedor de agentes precisa, independente da escolha de framework.

---

## Tabela de Habilidades

### Frameworks de Agentes

| # | Habilidade | Descrição | Caminho |
|---|-----------|-----------|---------|
| 1 | **LangChain Agents** | Construa agentes com LangChain e LangGraph | [`skills/Agent框架/langchain-agents.md`](skills/Agent框架/langchain-agents.md) |
| 2 | **AutoGen Multi-Agent** | Conversas multi-agente com Microsoft AutoGen | [`skills/Agent框架/autogen-multiagent.md`](skills/Agent框架/autogen-multiagent.md) |
| 3 | **CrewAI Agents** | Orquestração de agentes baseada em papéis com CrewAI | [`skills/Agent框架/crewai-agents.md`](skills/Agent框架/crewai-agents.md) |
| 4 | **Custom Agents** | Construa agentes do zero sem frameworks | [`skills/Agent框架/custom-agents.md`](skills/Agent框架/custom-agents.md) |

### Uso de Ferramentas

| # | Habilidade | Descrição | Caminho |
|---|-----------|-----------|---------|
| 5 | **Function Calling** | Chamadas de função e uso de ferramentas entre provedores | [`skills/工具调用/function-calling.md`](skills/工具调用/function-calling.md) |
| 6 | **MCP Integration** | Integração de servidores Model Context Protocol | [`skills/工具调用/mcp-integration.md`](skills/工具调用/mcp-integration.md) |
| 7 | **API Orchestration** | Combinando múltiplas APIs em fluxos de agente | [`skills/工具调用/api-orchestration.md`](skills/工具调用/api-orchestration.md) |

### Sistemas de Memória

| # | Habilidade | Descrição | Caminho |
|---|-----------|-----------|---------|
| 8 | **RAG Memory** | Memória de geração aumentada por recuperação para agentes | [`skills/记忆系统/rag-memory.md`](skills/记忆系统/rag-memory.md) |
| 9 | **Long-Term Memory** | Grafos de conhecimento e memória persistente | [`skills/记忆系统/long-term-memory.md`](skills/记忆系统/long-term-memory.md) |

### Multi-Agente

| # | Habilidade | Descrição | Caminho |
|---|-----------|-----------|---------|
| 10 | **Agent Communication** | Protocolos de mensagens entre agentes | [`skills/多Agent协作/agent-communication.md`](skills/多Agent协作/agent-communication.md) |
| 11 | **Task Decomposition** | Decompondo tarefas complexas em subtarefas | [`skills/多Agent协作/task-decomposition.md`](skills/多Agent协作/task-decomposition.md) |
| 12 | **Agent Orchestration** | Coordenando múltiplos agentes em fluxos de trabalho | [`skills/多Agent协作/agent-orchestration.md`](skills/多Agent协作/agent-orchestration.md) |

---

## Início Rápido

### Usando com o Cursor

1. Clone este repositório:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. Abra o Cursor → Configurações → Regras → Adicione `awesome-ai-agents/skills/` como contexto
3. Pergunte ao Cursor: *"Leia a habilidade langchain-agents e construa um agente com pesquisa na web"*

### Usando com o Claude Code

1. Clone este repositório:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. No seu projeto, referencie a habilidade:
   ```bash
   # Copie a habilidade necessária
   cp awesome-ai-agents/skills/Agent框架/custom-agents.md .claude/agents/
   ```
3. Ou referencie diretamente:
   ```
   Read awesome-ai-agents/skills/工具调用/function-calling.md and implement it in my project
   ```

### Usando com o Kimi Code

1. Clone este repositório:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. Adicione como diretório de habilidades:
   ```bash
   cp -r awesome-ai-agents/skills/ ~/.agents/skills/ai-agents/
   ```
3. Ou referencie na conversa:
   ```
   Read the skill at awesome-ai-agents/skills/记忆系统/rag-memory.md and set up RAG for my project
   ```

### Como Referência

Cada arquivo de habilidade é autônomo. Navegue pelo diretório de habilidades, escolha a que precisa e siga o diagrama de arquitetura, os templates de código e os padrões.

---

## O Que Cada Habilidade Contém

Todo arquivo de habilidade segue a mesma estrutura:

- **Quando Usar** — Matriz de decisão clara para quando a habilidade se aplica
- **Arquitetura** — Diagrama ASCII mostrando o design do sistema
- **Templates de Código** — Código Python para produção, pronto para copiar e colar
- **Padrões** — Padrões de design comuns com código
- **Armadilhas** — Coisas que vão te prejudicar (com soluções)
- **Padrões Comprovados** — Sabedoria de produção conquistada com esforço
- **Veja Também** — Referências cruzadas para habilidades relacionadas

---

## Caminho de Aprendizado

### Iniciante
```
Custom Agents → Function Calling → RAG Memory
```

### Intermediário
```
LangChain Agents → MCP Integration → Long-Term Memory → Task Decomposition
```

### Avançado
```
AutoGen Multi-Agent → Agent Communication → Agent Orchestration → API Orchestration
```

---

## Contribuição

Aceitamos contribuições! Consulte [CONTRIBUTING.md](CONTRIBUTING.md) para diretrizes.

---

## Licença

Este projeto está licenciado sob a licença MIT — veja [LICENSE](LICENSE) para detalhes.

---

## Veja Também

Confira nossos outros projetos `awesome-*`:

| Projeto | Descrição |
|---------|-----------|
| [awesome-ai-agents](https://github.com/liangzhengtao/awesome-ai-agents) | Habilidades para construir agentes de IA |
| [awesome-llm-apps](https://github.com/liangzhengtao/awesome-llm-apps) | Coleção de aplicações com LLM |
| [awesome-rag](https://github.com/liangzhengtao/awesome-rag) | Técnicas e implementações RAG |
| [awesome-prompt-engineering](https://github.com/liangzhengtao/awesome-prompt-engineering) | Guias e templates de engenharia de prompts |
| [awesome-ai-coding](https://github.com/liangzhengtao/awesome-ai-coding) | Ferramentas e práticas de programação com IA |
| [awesome-gpt-store](https://github.com/liangzhengtao/awesome-gpt-store) | Aplicações curadas da GPT Store |
| [awesome-ai-api](https://github.com/liangzhengtao/awesome-ai-api) | Guias de integração de APIs de IA |
| [awesome-llm-fine-tuning](https://github.com/liangzhengtao/awesome-llm-fine-tuning) | Técnicas de fine-tuning de LLM |
| [awesome-vector-database](https://github.com/liangzhengtao/awesome-vector-database) | Comparações e guias de bancos de dados vetoriais |
| [awesome-ai-safety](https://github.com/liangzhengtao/awesome-ai-safety) | Recursos de segurança e alinhamento de IA |
| [awesome-multimodal-ai](https://github.com/liangzhengtao/awesome-multimodal-ai) | Técnicas de IA multimodal |
| [awesome-open-source-llm](https://github.com/liangzhengtao/awesome-open-source-llm) | Coleção de LLMs open-source |

---

<div align="center">

**Feito com carinho para a comunidade de agentes de IA.**

[⬆ Voltar ao topo](#awesome-ai-agents)

</div>
