[English](README.md) | [中文](README.zh.md) | [日本語](README.ja.md) | [Français](README.fr.md) | [Español](README.es.md)
n<div align="center">

<img src=".banner.svg" width="100%" alt="banner">

</div>


# Awesome AI Agents

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Stars](https://img.shields.io/github/stars/liangzhengtao/awesome-ai-agents?style=social)](https://github.com/liangzhengtao/awesome-ai-agents)

---

> **Deja de construir chatbots básicos. 12 habilidades para construir agentes de IA de producción.**

Una colección de habilidades probadas para construir agentes de IA — desde la llamada a herramientas de un solo agente hasta la orquestación multi-agente. Cada habilidad es una guía completa, lista para copiar y pegar, con diagramas de arquitectura, plantillas de código, patrones y errores comunes.

---

## ¿Por qué este proyecto?

El ecosistema de agentes de IA en 2026 está fragmentado — LangChain, AutoGen, CrewAI, MCP, soluciones personalizadas — cada uno con diferentes APIs y patrones. Este proyecto destila las **habilidades fundamentales** que todo desarrollador de agentes necesita, independientemente de la elección del framework.

---

## Tabla de habilidades

### Frameworks de agentes

| # | Habilidad | Descripción | Ruta |
|---|----------|-------------|------|
| 1 | **LangChain Agents** | Construir agentes con LangChain y LangGraph | [`skills/Agent框架/langchain-agents.md`](skills/Agent框架/langchain-agents.md) |
| 2 | **AutoGen Multi-Agent** | Conversaciones multi-agente con Microsoft AutoGen | [`skills/Agent框架/autogen-multiagent.md`](skills/Agent框架/autogen-multiagent.md) |
| 3 | **CrewAI Agents** | Orquestación de agentes basada en roles con CrewAI | [`skills/Agent框架/crewai-agents.md`](skills/Agent框架/crewai-agents.md) |
| 4 | **Custom Agents** | Construir agentes desde cero sin frameworks | [`skills/Agent框架/custom-agents.md`](skills/Agent框架/custom-agents.md) |

### Uso de herramientas

| # | Habilidad | Descripción | Ruta |
|---|----------|-------------|------|
| 5 | **Function Calling** | Llamadas a funciones y uso de herramientas multi-proveedor | [`skills/工具调用/function-calling.md`](skills/工具调用/function-calling.md) |
| 6 | **MCP Integration** | Integración de servidores Model Context Protocol | [`skills/工具调用/mcp-integration.md`](skills/工具调用/mcp-integration.md) |
| 7 | **API Orchestration** | Combinar múltiples APIs en flujos de trabajo de agentes | [`skills/工具调用/api-orchestration.md`](skills/工具调用/api-orchestration.md) |

### Sistemas de memoria

| # | Habilidad | Descripción | Ruta |
|---|----------|-------------|------|
| 8 | **RAG Memory** | Memoria RAG para agentes | [`skills/记忆系统/rag-memory.md`](skills/记忆系统/rag-memory.md) |
| 9 | **Long-Term Memory** | Grafos de conocimiento y memoria persistente | [`skills/记忆系统/long-term-memory.md`](skills/记忆系统/long-term-memory.md) |

### Multi-Agente

| # | Habilidad | Descripción | Ruta |
|---|----------|-------------|------|
| 10 | **Agent Communication** | Protocolos de mensajería entre agentes | [`skills/多Agent协作/agent-communication.md`](skills/多Agent协作/agent-communication.md) |
| 11 | **Task Decomposition** | Dividir tareas complejas en subtareas | [`skills/多Agent协作/task-decomposition.md`](skills/多Agent协作/task-decomposition.md) |
| 12 | **Agent Orchestration** | Coordinar múltiples agentes en flujos de trabajo | [`skills/多Agent协作/agent-orchestration.md`](skills/多Agent协作/agent-orchestration.md) |

---

## Inicio rápido

### Uso con Cursor

1. Clonar este repositorio:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. Abrir Cursor → Configuración → Reglas → Agregar `awesome-ai-agents/skills/` como contexto
3. Pedir a Cursor: *"Lee la habilidad langchain-agents y construye un agente con búsqueda web"*

### Uso con Claude Code

1. Clonar este repositorio:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. En tu proyecto, referenciar la habilidad:
   ```bash
   # Copiar la habilidad que necesitas
   cp awesome-ai-agents/skills/Agent框架/custom-agents.md .claude/agents/
   ```
3. O referenciar directamente:
   ```
   Lee awesome-ai-agents/skills/工具调用/function-calling.md e impleméntalo en mi proyecto
   ```

### Uso con Kimi Code

1. Clonar este repositorio:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. Agregar como directorio de habilidades:
   ```bash
   cp -r awesome-ai-agents/skills/ ~/.agents/skills/ai-agents/
   ```
3. O referenciar en una conversación:
   ```
   Lee la habilidad awesome-ai-agents/skills/记忆系统/rag-memory.md y configura RAG para mi proyecto
   ```

### Uso como referencia

Cada archivo de habilidad es autónomo. Explora el directorio de habilidades, elige el que necesites, y sigue el diagrama de arquitectura, las plantillas de código y los patrones.

---

## Contenido de cada habilidad

Cada archivo de habilidad sigue la misma estructura:

- **Cuándo usarlo** — Matriz de decisión clara para saber cuándo aplica esta habilidad
- **Arquitectura** — Diagrama ASCII mostrando el diseño del sistema
- **Plantillas de código** — Código Python de producción, listo para copiar y pegar
- **Patrones** — Patrones de diseño comunes con código
- **Errores comunes** — Lo que puede salir mal (con soluciones)
- **Patrones probados** — Sabiduría de producción ganada en batalla
- **Ver también** — Referencias cruzadas a habilidades relacionadas

---

## Ruta de aprendizaje

### Principiante
```
Custom Agents → Function Calling → RAG Memory
```

### Intermedio
```
LangChain Agents → MCP Integration → Long-Term Memory → Task Decomposition
```

### Avanzado
```
AutoGen Multi-Agent → Agent Communication → Agent Orchestration → API Orchestration
```

---

## Contribuir

¡Las contribuciones son bienvenidas! Consulta [CONTRIBUTING.md](CONTRIBUTING.md) para las directrices.

---

## Licencia

Este proyecto está licenciado bajo la licencia MIT — ver [LICENSE](LICENSE) para más detalles.

---

## Ver también

Consulta nuestros otros proyectos `awesome-*`:

| Proyecto | Descripción |
|----------|-------------|
| [awesome-ai-agents](https://github.com/liangzhengtao/awesome-ai-agents) | Habilidades para construir agentes de IA |
| [awesome-llm-apps](https://github.com/liangzhengtao/awesome-llm-apps) | Colección de aplicaciones impulsadas por LLM |
| [awesome-rag](https://github.com/liangzhengtao/awesome-rag) | Técnicas e implementaciones RAG |
| [awesome-prompt-engineering](https://github.com/liangzhengtao/awesome-prompt-engineering) | Guías y plantillas de prompt engineering |
| [awesome-ai-coding](https://github.com/liangzhengtao/awesome-ai-coding) | Herramientas y prácticas de codificación con IA |
| [awesome-gpt-store](https://github.com/liangzhengtao/awesome-gpt-store) | Aplicaciones seleccionadas de GPT Store |
| [awesome-ai-api](https://github.com/liangzhengtao/awesome-ai-api) | Guías de integración de APIs de IA |
| [awesome-llm-fine-tuning](https://github.com/liangzhengtao/awesome-llm-fine-tuning) | Técnicas de fine-tuning de LLM |
| [awesome-vector-database](https://github.com/liangzhengtao/awesome-vector-database) | Comparaciones y guías de bases de datos vectoriales |
| [awesome-ai-safety](https://github.com/liangzhengtao/awesome-ai-safety) | Recursos de seguridad y alineación de IA |
| [awesome-multimodal-ai](https://github.com/liangzhengtao/awesome-multimodal-ai) | Técnicas de IA multimodal |
| [awesome-open-source-llm](https://github.com/liangzhengtao/awesome-open-source-llm) | Colección de LLM de código abierto |

---

<div align="center">

**Construido con dedicación para la comunidad de agentes de IA.**

[⬆ Volver arriba](#awesome-ai-agents)

</div>
