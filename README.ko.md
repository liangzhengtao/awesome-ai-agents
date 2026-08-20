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

> **기본적인 챗봇을 그만 만드세요. 프로덕션 AI 에이전트를 구축하기 위한 12가지 스킬.**

AI 에이전트 구축을 위한 검증된 스킬 모음 — 단일 에이전트 도구 호출부터 멀티 에이전트 오케스트레이션까지. 각 스킬은 아키텍처 다이어그램, 코드 템플릿, 패턴, 함정을 포함한 완전한 복사-붙여넣기 가이드입니다.

---

## 이 프로젝트가 필요한 이유

2026년 AI 에이전트 생태계는 파편화되어 있습니다 — LangChain, AutoGen, CrewAI, MCP, 커스텀 솔루션 등 각각 다른 API와 패턴을 가지고 있습니다. 이 프로젝트는 프레임워크 선택과 관계없이 모든 에이전트 개발자에게 필요한 **핵심 스킬**을 정제합니다.

---

## 스킬 테이블

### 에이전트 프레임워크

| # | 스킬 | 설명 | 경로 |
|---|------|------|------|
| 1 | **LangChain Agents** | LangChain과 LangGraph로 에이전트 구축 | [`skills/Agent框架/langchain-agents.md`](skills/Agent框架/langchain-agents.md) |
| 2 | **AutoGen Multi-Agent** | Microsoft AutoGen으로 멀티 에이전트 대화 | [`skills/Agent框架/autogen-multiagent.md`](skills/Agent框架/autogen-multiagent.md) |
| 3 | **CrewAI Agents** | CrewAI로 역할 기반 에이전트 오케스트레이션 | [`skills/Agent框架/crewai-agents.md`](skills/Agent框架/crewai-agents.md) |
| 4 | **Custom Agents** | 프레임워크 없이 에이전트를 처음부터 구축 | [`skills/Agent框架/custom-agents.md`](skills/Agent框架/custom-agents.md) |

### 도구 사용

| # | 스킬 | 설명 | 경로 |
|---|------|------|------|
| 5 | **Function Calling** | 프로바이더 간 함수 호출 및 도구 사용 | [`skills/工具调用/function-calling.md`](skills/工具调用/function-calling.md) |
| 6 | **MCP Integration** | 모델 컨텍스트 프로토콜 서버 통합 | [`skills/工具调用/mcp-integration.md`](skills/工具调用/mcp-integration.md) |
| 7 | **API Orchestration** | 에이전트 워크플로우에 여러 API 결합 | [`skills/工具调用/api-orchestration.md`](skills/工具调用/api-orchestration.md) |

### 메모리 시스템

| # | 스킬 | 설명 | 경로 |
|---|------|------|------|
| 8 | **RAG Memory** | 에이전트용 검색 증강 생성 메모리 | [`skills/记忆系统/rag-memory.md`](skills/记忆系统/rag-memory.md) |
| 9 | **Long-Term Memory** | 지식 그래프 및 영구 메모리 | [`skills/记忆系统/long-term-memory.md`](skills/记忆系统/long-term-memory.md) |

### 멀티 에이전트

| # | 스킬 | 설명 | 경로 |
|---|------|------|------|
| 10 | **Agent Communication** | 에이전트 간 메시징 프로토콜 | [`skills/多Agent协作/agent-communication.md`](skills/多Agent协作/agent-communication.md) |
| 11 | **Task Decomposition** | 복잡한 작업을 하위 작업으로 분해 | [`skills/多Agent协作/task-decomposition.md`](skills/多Agent协作/task-decomposition.md) |
| 12 | **Agent Orchestration** | 워크플로우에서 여러 에이전트 조정 | [`skills/多Agent协作/agent-orchestration.md`](skills/多Agent协作/agent-orchestration.md) |

---

## 빠른 시작

### Cursor에서 사용

1. 이 저장소를 클론합니다:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. Cursor 열기 → 설정 → 규칙 → `awesome-ai-agents/skills/`를 컨텍스트로 추가
3. Cursor에게 요청: *"langchain-agents 스킬을 읽고 웹 검색이 가능한 에이전트를 만들어줘"*

### Claude Code에서 사용

1. 이 저장소를 클론합니다:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. 프로젝트에서 스킬을 참조합니다:
   ```bash
   # 필요한 스킬을 복사
   cp awesome-ai-agents/skills/Agent框架/custom-agents.md .claude/agents/
   ```
3. 또는 직접 참조:
   ```
   Read awesome-ai-agents/skills/工具调用/function-calling.md and implement it in my project
   ```

### Kimi Code에서 사용

1. 이 저장소를 클론합니다:
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. 스킬 디렉토리로 추가:
   ```bash
   cp -r awesome-ai-agents/skills/ ~/.agents/skills/ai-agents/
   ```
3. 또는 대화에서 참조:
   ```
   Read the skill at awesome-ai-agents/skills/记忆系统/rag-memory.md and set up RAG for my project
   ```

### 참고 자료로 사용

각 스킬 파일은 독립적입니다. 스킬 디렉토리를 탐색하고, 필요한 것을 선택하고, 아키텍처 다이어그램, 코드 템플릿, 패턴을 따르세요.

---

## 각 스킬의 구성 요소

모든 스킬 파일은 동일한 구조를 따릅니다:

- **언제 사용하는가** — 이 스킬이 적용되는 시점의 명확한 의사결정 매트릭스
- **아키텍처** — 시스템 설계를 보여주는 ASCII 다이어그램
- **코드 템플릿** — 프로덕션용 Python 코드, 복사-붙여넣기 가능
- **패턴** — 코드와 함께하는 일반적인 설계 패턴
- **함정** — 주의해야 할 사항 (해결책 포함)
- **검증된 패턴** — 값비싼 경험으로 얻은 프로덕션 지혜
- **함께 보기** — 관련 스킬에 대한 교차 참조

---

## 학습 경로

### 초급
```
Custom Agents → Function Calling → RAG Memory
```

### 중급
```
LangChain Agents → MCP Integration → Long-Term Memory → Task Decomposition
```

### 고급
```
AutoGen Multi-Agent → Agent Communication → Agent Orchestration → API Orchestration
```

---

## 기여하기

기여를 환영합니다! 가이드라인은 [CONTRIBUTING.md](CONTRIBUTING.md)를 참조하세요.

---

## 라이선스

이 프로젝트는 MIT 라이선스에 따라 사용이 허가됩니다 — 자세한 내용은 [LICENSE](LICENSE)를 참조하세요.

---

## 함께 보기

다른 `awesome-*` 프로젝트도 확인해보세요:

| 프로젝트 | 설명 |
|---------|------|
| [awesome-ai-agents](https://github.com/liangzhengtao/awesome-ai-agents) | AI 에이전트 구축 스킬 |
| [awesome-llm-apps](https://github.com/liangzhengtao/awesome-llm-apps) | LLM 기반 애플리케이션 모음 |
| [awesome-rag](https://github.com/liangzhengtao/awesome-rag) | RAG 기술 및 구현 |
| [awesome-prompt-engineering](https://github.com/liangzhengtao/awesome-prompt-engineering) | 프롬프트 엔지니어링 가이드 및 템플릿 |
| [awesome-ai-coding](https://github.com/liangzhengtao/awesome-ai-coding) | AI 기반 코딩 도구 및 관행 |
| [awesome-gpt-store](https://github.com/liangzhengtao/awesome-gpt-store) | 엄선된 GPT Store 애플리케이션 |
| [awesome-ai-api](https://github.com/liangzhengtao/awesome-ai-api) | AI API 통합 가이드 |
| [awesome-llm-fine-tuning](https://github.com/liangzhengtao/awesome-llm-fine-tuning) | LLM 미세 조정 기술 |
| [awesome-vector-database](https://github.com/liangzhengtao/awesome-vector-database) | 벡터 데이터베이스 비교 및 가이드 |
| [awesome-ai-safety](https://github.com/liangzhengtao/awesome-ai-safety) | AI 안전 및 정렬 자료 |
| [awesome-multimodal-ai](https://github.com/liangzhengtao/awesome-multimodal-ai) | 멀티모달 AI 기술 |
| [awesome-open-source-llm](https://github.com/liangzhengtao/awesome-open-source-llm) | 오픈소스 LLM 모음 |

---

<div align="center">

**AI 에이전트 커뮤니티를 위해 정성껏 제작되었습니다.**

[⬆ 맨 위로 돌아가기](#awesome-ai-agents)

</div>
