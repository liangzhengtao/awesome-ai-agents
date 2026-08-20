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

> **Hören Sie auf, einfache Chatbots zu bauen. 12 Fähigkeiten für produktionsreife KI-Agenten.**

Eine kuratierte Sammlung erprobter Fähigkeiten zum Aufbau von KI-Agenten — von Tool-Aufrufen einzelner Agenten bis hin zur Multi-Agent-Orchestrierung. Jede Fähigkeit ist eine vollständige, kopierbereite Anleitung mit Architekturdiagrammen, Code-Templates, Mustern und Fallstricken.

---

## Warum dieses Projekt?

Das KI-Agenten-Ökosystem im Jahr 2026 ist fragmentiert — LangChain, AutoGen, CrewAI, MCP, individuelle Lösungen — jede mit eigenen APIs und Mustern. Dieses Projekt destilliert die **Kernfähigkeiten**, die jeder Agent-Entwickler braucht, unabhängig von der Framework-Wahl.

---

## Fähigkeiten-Übersicht

### Agent-Frameworks

| # | Fähigkeit | Beschreibung | Pfad |
|---|----------|-------------|------|
| 1 | **LangChain Agents** | Agenten mit LangChain und LangGraph erstellen | [`skills/Agent框架/langchain-agents.md`](skills/Agent框架/langchain-agents.md) |
| 2 | **AutoGen Multi-Agent** | Multi-Agent-Gespräche mit Microsoft AutoGen | [`skills/Agent框架/autogen-multiagent.md`](skills/Agent框架/autogen-multiagent.md) |
| 3 | **CrewAI Agents** | Rollenbasierte Agent-Orchestrierung mit CrewAI | [`skills/Agent框架/crewai-agents.md`](skills/Agent框架/crewai-agents.md) |
| 4 | **Custom Agents** | Agenten von Grund auf ohne Frameworks erstellen | [`skills/Agent框架/custom-agents.md`](skills/Agent框架/custom-agents.md) |

### Tool-Nutzung

| # | Fähigkeit | Beschreibung | Pfad |
|---|----------|-------------|------|
| 5 | **Function Calling** | Funktionsaufrufe und Tool-Nutzung über Anbieter hinweg | [`skills/工具调用/function-calling.md`](skills/工具调用/function-calling.md) |
| 6 | **MCP Integration** | Model Context Protocol Server-Integration | [`skills/工具调用/mcp-integration.md`](skills/工具调用/mcp-integration.md) |
| 7 | **API Orchestration** | Mehrere APIs in Agent-Workflows kombinieren | [`skills/工具调用/api-orchestration.md`](skills/工具调用/api-orchestration.md) |

### Speichersysteme

| # | Fähigkeit | Beschreibung | Pfad |
|---|----------|-------------|------|
| 8 | **RAG Memory** | Retrieval-Augmented-Generation-Speicher für Agenten | [`skills/记忆系统/rag-memory.md`](skills/记忆系统/rag-memory.md) |
| 9 | **Long-Term Memory** | Wissensgraphen und persistenter Speicher | [`skills/记忆系统/long-term-memory.md`](skills/记忆系统/long-term-memory.md) |

### Multi-Agent

| # | Fähigkeit | Beschreibung | Pfad |
|---|----------|-------------|------|
| 10 | **Agent Communication** | Protokolle für Agent-zu-Agent-Nachrichten | [`skills/多Agent协作/agent-communication.md`](skills/多Agent协作/agent-communication.md) |
| 11 | **Task Decomposition** | Komplexe Aufgaben in Teilaufgaben zerlegen | [`skills/多Agent协作/task-decomposition.md`](skills/多Agent协作/task-decomposition.md) |
| 12 | **Agent Orchestration** | Mehrere Agenten in Workflows koordinieren | [`skills/多Agent协作/agent-orchestration.md`](skills/多Agent协作/agent-orchestration.md) |

---

## Schnellstart

### Mit Cursor verwenden

1. Klonen Sie dieses Repository:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. Cursor öffnen → Einstellungen → Regeln → `awesome-ai-agents/skills/` als Kontext hinzufügen
3. Cursor fragen: *"Lies die langchain-agents Fähigkeit und baue mir einen Agenten mit Websuche"*

### Mit Claude Code verwenden

1. Klonen Sie dieses Repository:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. In Ihrem Projekt die Fähigkeit referenzieren:
   ```bash
   # Benötigte Fähigkeit kopieren
   cp awesome-ai-agents/skills/Agent框架/custom-agents.md .claude/agents/
   ```
3. Oder direkt referenzieren:
   ```
   Read awesome-ai-agents/skills/工具调用/function-calling.md and implement it in my project
   ```

### Mit Kimi Code verwenden

1. Klonen Sie dieses Repository:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. Als Fähigkeiten-Verzeichnis hinzufügen:
   ```bash
   cp -r awesome-ai-agents/skills/ ~/.agents/skills/ai-agents/
   ```
3. Oder im Gespräch referenzieren:
   ```
   Read the skill at awesome-ai-agents/skills/记忆系统/rag-memory.md and set up RAG for my project
   ```

### Als Referenz

Jede Fähigkeitsdatei ist eigenständig. Durchsuchen Sie das Fähigkeitsverzeichnis, wählen Sie die passende und folgen Sie dem Architekturdiagramm, den Code-Templates und Mustern.

---

## Was jede Fähigkeit enthält

Jede Fähigkeitsdatei folgt der gleichen Struktur:

- **Wann verwenden** — Klare Entscheidungsmatrix für die Anwendbarkeit der Fähigkeit
- **Architektur** — ASCII-Diagramm des Systemdesigns
- **Code-Templates** — Produktionsreifer Python-Code, kopierbereit
- **Muster** — Häufige Entwurfsmuster mit Code
- **Fallstricke** — Was Sie erwischen kann (mit Lösungen)
- **Bewährte Muster** — Hart erkämpftes Produktionswissen
- **Siehe auch** — Querverweise auf verwandte Fähigkeiten

---

## Lernpfad

### Anfänger
```
Custom Agents → Function Calling → RAG Memory
```

### Fortgeschritten
```
LangChain Agents → MCP Integration → Long-Term Memory → Task Decomposition
```

### Experte
```
AutoGen Multi-Agent → Agent Communication → Agent Orchestration → API Orchestration
```

---

## Mitwirken

Wir freuen uns über Beiträge! Siehe [CONTRIBUTING.md](CONTRIBUTING.md) für Richtlinien.

---

## Lizenz

Dieses Projekt steht unter der MIT-Lizenz — siehe [LICENSE](LICENSE) für Details.

---

## Siehe auch

Entdecken Sie unsere anderen `awesome-*` Projekte:

| Projekt | Beschreibung |
|---------|-------------|
| [awesome-ai-agents](https://github.com/liangzhengtao/awesome-ai-agents) | Fähigkeiten für den Aufbau von KI-Agenten |
| [awesome-llm-apps](https://github.com/liangzhengtao/awesome-llm-apps) | Sammlung von LLM-Anwendungen |
| [awesome-rag](https://github.com/liangzhengtao/awesome-rag) | RAG-Techniken und Implementierungen |
| [awesome-prompt-engineering](https://github.com/liangzhengtao/awesome-prompt-engineering) | Prompt-Engineering-Anleitungen und Templates |
| [awesome-ai-coding](https://github.com/liangzhengtao/awesome-ai-coding) | KI-gestützte Coding-Tools und Praktiken |
| [awesome-gpt-store](https://github.com/liangzhengtao/awesome-gpt-store) | Kuratierte GPT-Store-Anwendungen |
| [awesome-ai-api](https://github.com/liangzhengtao/awesome-ai-api) | KI-API-Integrationsleitfäden |
| [awesome-llm-fine-tuning](https://github.com/liangzhengtao/awesome-llm-fine-tuning) | LLM-Fine-Tuning-Techniken |
| [awesome-vector-database](https://github.com/liangzhengtao/awesome-vector-database) | Vektordatenbank-Vergleiche und Leitfäden |
| [awesome-ai-safety](https://github.com/liangzhengtao/awesome-ai-safety) | KI-Sicherheits- und Alignment-Ressourcen |
| [awesome-multimodal-ai](https://github.com/liangzhengtao/awesome-multimodal-ai) | Multimodale KI-Techniken |
| [awesome-open-source-llm](https://github.com/liangzhengtao/awesome-open-source-llm) | Open-Source LLM-Sammlung |

---

<div align="center">

**Mit Liebe für die KI-Agenten-Community erstellt.**

[⬆ Nach oben](#awesome-ai-agents)

</div>
