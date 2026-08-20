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

> **Хватит создавать простых чат-ботов. 12 навыков для создания продакшен-агентов на основе ИИ.**

Тщательно подобранная коллекция проверенных навыков для создания ИИ-агентов — от вызова инструментов в одиночном агенте до оркестрации мультиагентных систем. Каждый навык представляет собой полное руководство, готовое к копированию, с архитектурными схемами, шаблонами кода, паттернами и типичными ошибками.

---

## Зачем этот проект?

Экосистема ИИ-агентов в 2026 году фрагментирована — LangChain, AutoGen, CrewAI, MCP, пользовательские решения — каждый со своими API и паттернами. Этот проект концентрирует **ключевые навыки**, необходимые каждому разработчику агентов, независимо от выбора фреймворка.

---

## Таблица навыков

### Фреймворки агентов

| # | Навык | Описание | Путь |
|---|-------|----------|------|
| 1 | **LangChain Agents** | Создание агентов с LangChain и LangGraph | [`skills/Agent框架/langchain-agents.md`](skills/Agent框架/langchain-agents.md) |
| 2 | **AutoGen Multi-Agent** | Мультиагентные беседы с Microsoft AutoGen | [`skills/Agent框架/autogen-multiagent.md`](skills/Agent框架/autogen-multiagent.md) |
| 3 | **CrewAI Agents** | Оркестрация агентов на основе ролей с CrewAI | [`skills/Agent框架/crewai-agents.md`](skills/Agent框架/crewai-agents.md) |
| 4 | **Custom Agents** | Создание агентов с нуля без фреймворков | [`skills/Agent框架/custom-agents.md`](skills/Agent框架/custom-agents.md) |

### Использование инструментов

| # | Навык | Описание | Путь |
|---|-------|----------|------|
| 5 | **Function Calling** | Вызов функций и использование инструментов у разных провайдеров | [`skills/工具调用/function-calling.md`](skills/工具调用/function-calling.md) |
| 6 | **MCP Integration** | Интеграция серверов Model Context Protocol | [`skills/工具调用/mcp-integration.md`](skills/工具调用/mcp-integration.md) |
| 7 | **API Orchestration** | Объединение нескольких API в рабочие процессы агентов | [`skills/工具调用/api-orchestration.md`](skills/工具调用/api-orchestration.md) |

### Системы памяти

| # | Навык | Описание | Путь |
|---|-------|----------|------|
| 8 | **RAG Memory** | Память генерации с дополнением из поиска для агентов | [`skills/记忆系统/rag-memory.md`](skills/记忆系统/rag-memory.md) |
| 9 | **Long-Term Memory** | Графы знаний и постоянная память | [`skills/记忆系统/long-term-memory.md`](skills/记忆系统/long-term-memory.md) |

### Мультиагентные системы

| # | Навык | Описание | Путь |
|---|-------|----------|------|
| 10 | **Agent Communication** | Протоколы обмена сообщениями между агентами | [`skills/多Agent协作/agent-communication.md`](skills/多Agent协作/agent-communication.md) |
| 11 | **Task Decomposition** | Разбиение сложных задач на подзадачи | [`skills/多Agent协作/task-decomposition.md`](skills/多Agent协作/task-decomposition.md) |
| 12 | **Agent Orchestration** | Координация нескольких агентов в рабочих процессах | [`skills/多Agent协作/agent-orchestration.md`](skills/多Agent协作/agent-orchestration.md) |

---

## Быстрый старт

### Использование с Cursor

1. Клонируйте этот репозиторий:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. Откройте Cursor → Настройки → Правила → Добавьте `awesome-ai-agents/skills/` как контекст
3. Спросите Cursor: *"Прочитай навык langchain-agents и создай агента с веб-поиском"*

### Использование с Claude Code

1. Клонируйте этот репозиторий:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. В вашем проекте сослитесь на навык:
   ```bash
   # Скопируйте нужный навык
   cp awesome-ai-agents/skills/Agent框架/custom-agents.md .claude/agents/
   ```
3. Или сослитесь напрямую:
   ```
   Read awesome-ai-agents/skills/工具调用/function-calling.md and implement it in my project
   ```

### Использование с Kimi Code

1. Клонируйте этот репозиторий:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. Добавьте как директорию навыков:
   ```bash
   cp -r awesome-ai-agents/skills/ ~/.agents/skills/ai-agents/
   ```
3. Или сослитесь в разговоре:
   ```
   Read the skill at awesome-ai-agents/skills/记忆系统/rag-memory.md and set up RAG for my project
   ```

### Как справочник

Каждый файл навыка является самостоятельным. Просмотрите директорию навыков, выберите нужный и следуйте архитектурной схеме, шаблонам кода и паттернам.

---

## Что содержит каждый навык

Каждый файл навыка следует одинаковой структуре:

- **Когда использовать** — Чёткая матрица решений для определения применимости навыка
- **Архитектура** — ASCII-диаграмма, показывающая дизайн системы
- **Шаблоны кода** — Продакшен-код на Python, готовый к копированию
- **Паттерны** — Типичные паттерны проектирования с кодом
- **Подводные камни** — Что может вас подстерегать (с решениями)
- **Проверенные паттерны** — Продакшен-мудрость, добытая опытом
- **Смотрите также** — Перекрёстные ссылки на связанные навыки

---

## Путь обучения

### Начинающий
```
Custom Agents → Function Calling → RAG Memory
```

### Средний
```
LangChain Agents → MCP Integration → Long-Term Memory → Task Decomposition
```

### Продвинутый
```
AutoGen Multi-Agent → Agent Communication → Agent Orchestration → API Orchestration
```

---

## Участие в проекте

Мы приветствуем вклады! См. [CONTRIBUTING.md](CONTRIBUTING.md) для руководящих принципов.

---

## Лицензия

Проект распространяется под лицензией MIT — подробности в [LICENSE](LICENSE).

---

## Смотрите также

Ознакомьтесь с нашими другими проектами `awesome-*`:

| Проект | Описание |
|--------|----------|
| [awesome-ai-agents](https://github.com/liangzhengtao/awesome-ai-agents) | Навыки для создания ИИ-агентов |
| [awesome-llm-apps](https://github.com/liangzhengtao/awesome-llm-apps) | Коллекция приложений на основе LLM |
| [awesome-rag](https://github.com/liangzhengtao/awesome-rag) | Техники и реализации RAG |
| [awesome-prompt-engineering](https://github.com/liangzhengtao/awesome-prompt-engineering) | Руководства и шаблоны промпт-инженерии |
| [awesome-ai-coding](https://github.com/liangzhengtao/awesome-ai-coding) | Инструменты и практики ИИ-программирования |
| [awesome-gpt-store](https://github.com/liangzhengtao/awesome-gpt-store) | Избранные приложения GPT Store |
| [awesome-ai-api](https://github.com/liangzhengtao/awesome-ai-api) | Руководства по интеграции ИИ-API |
| [awesome-llm-fine-tuning](https://github.com/liangzhengtao/awesome-llm-fine-tuning) | Техники дообучения LLM |
| [awesome-vector-database](https://github.com/liangzhengtao/awesome-vector-database) | Сравнения и руководства по векторным БД |
| [awesome-ai-safety](https://github.com/liangzhengtao/awesome-ai-safety) | Ресурсы по безопасности и выравниванию ИИ |
| [awesome-multimodal-ai](https://github.com/liangzhengtao/awesome-multimodal-ai) | Техники мультимодального ИИ |
| [awesome-open-source-llm](https://github.com/liangzhengtao/awesome-open-source-llm) | Коллекция LLM с открытым кодом |

---

<div align="center">

**Создано с заботой для сообщества ИИ-агентов.**

[⬆ Наверх](#awesome-ai-agents)

</div>
