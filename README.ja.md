[English](README.md) | [中文](README.zh.md) | [日本語](README.ja.md) | [Français](README.fr.md) | [Español](README.es.md) | [العربية](README.ar.md) | [한국어](README.ko.md) | [Português](README.pt.md) | [Русский](README.ru.md) | [Deutsch](README.de.md)
n<div align="center">

<img src=".banner.svg" width="100%" alt="banner">

</div>


# Awesome AI Agents

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![Stars](https://img.shields.io/github/stars/liangzhengtao/awesome-ai-agents?style=social)](https://github.com/liangzhengtao/awesome-ai-agents)

---

> **基本的なチャットボットの構築はもうやめよう。本番環境対応の AI エージェントを構築するための 12 のスキル。**

AI エージェント構築のための実績あるスキルコレクション——単一エージェントのツール呼び出しからマルチエージェントのオーケストレーションまで。各スキルは、アーキテクチャ図、コードテンプレート、パターン、落とし穴を含む、コピペでそのまま使える完全なガイドです。

---

## このプロジェクトについて

2026年の AI エージェントエコシステムは断片化しています——LangChain、AutoGen、CrewAI、MCP、カスタムソリューション——それぞれ異なる API とパターンを持っています。本プロジェクトは、フレームワークの選択に関係なく、すべてのエージェント開発者に必要な**コアスキル**を抽出しています。

---

## スキル一覧

### エージェントフレームワーク

| # | スキル | 説明 | パス |
|---|-------|------|------|
| 1 | **LangChain Agents** | LangChain と LangGraph でエージェントを構築 | [`skills/Agent框架/langchain-agents.md`](skills/Agent框架/langchain-agents.md) |
| 2 | **AutoGen Multi-Agent** | Microsoft AutoGen によるマルチエージェント対話 | [`skills/Agent框架/autogen-multiagent.md`](skills/Agent框架/autogen-multiagent.md) |
| 3 | **CrewAI Agents** | CrewAI によるロールベースのエージェントオーケストレーション | [`skills/Agent框架/crewai-agents.md`](skills/Agent框架/crewai-agents.md) |
| 4 | **Custom Agents** | フレームワークなしでエージェントをゼロから構築 | [`skills/Agent框架/custom-agents.md`](skills/Agent框架/custom-agents.md) |

### ツール使用

| # | スキル | 説明 | パス |
|---|-------|------|------|
| 5 | **Function Calling** | プロバイダー横断の関数呼び出しとツール使用 | [`skills/工具调用/function-calling.md`](skills/工具调用/function-calling.md) |
| 6 | **MCP Integration** | Model Context Protocol サーバー統合 | [`skills/工具调用/mcp-integration.md`](skills/工具调用/mcp-integration.md) |
| 7 | **API Orchestration** | 複数の API をエージェントワークフローに統合 | [`skills/工具调用/api-orchestration.md`](skills/工具调用/api-orchestration.md) |

### 記憶システム

| # | スキル | 説明 | パス |
|---|-------|------|------|
| 8 | **RAG Memory** | エージェント向けの RAG 記憶 | [`skills/记忆系统/rag-memory.md`](skills/记忆系统/rag-memory.md) |
| 9 | **Long-Term Memory** | ナレッジグラフと永続記憶 | [`skills/记忆系统/long-term-memory.md`](skills/记忆系统/long-term-memory.md) |

### マルチエージェント

| # | スキル | 説明 | パス |
|---|-------|------|------|
| 10 | **Agent Communication** | エージェント間メッセージングのプロトコル | [`skills/多Agent协作/agent-communication.md`](skills/多Agent协作/agent-communication.md) |
| 11 | **Task Decomposition** | 複雑なタスクをサブタスクに分解 | [`skills/多Agent协作/task-decomposition.md`](skills/多Agent协作/task-decomposition.md) |
| 12 | **Agent Orchestration** | ワークフロー内で複数のエージェントを協調 | [`skills/多Agent协作/agent-orchestration.md`](skills/多Agent协作/agent-orchestration.md) |

---

## クイックスタート

### Cursor で使用する

1. このリポジトリをクローン：
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. Cursor を開く → 設定 → ルール → `awesome-ai-agents/skills/` をコンテキストとして追加
3. Cursor に依頼：*"langchain-agents スキルを読んで、Web 検索付きのエージェントを構築して"*

### Claude Code で使用する

1. このリポジトリをクローン：
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. プロジェクトでスキルを参照：
   ```bash
   # 必要なスキルをコピー
   cp awesome-ai-agents/skills/Agent框架/custom-agents.md .claude/agents/
   ```
3. または直接参照：
   ```
   awesome-ai-agents/skills/工具调用/function-calling.md を読んで、私のプロジェクトに実装して
   ```

### Kimi Code で使用する

1. このリポジトリをクローン：
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. スキルディレクトリとして追加：
   ```bash
   cp -r awesome-ai-agents/skills/ ~/.agents/skills/ai-agents/
   ```
3. または会話で参照：
   ```
   awesome-ai-agents/skills/记忆系统/rag-memory.md のスキルを読んで、プロジェクトに RAG をセットアップして
   ```

### 参考資料として使用する

各スキルファイルは独立しています。`skills/` ディレクトリを閲覧して、必要なスキルを選び、アーキテクチャ図、コードテンプレート、パターンに従ってください。

---

## 各スキルの内容

すべてのスキルファイルは同じ構造に従います：

- **使用するタイミング** — このスキルが適用される条件の明確な判断基準
- **アーキテクチャ** — システム設計を示す ASCII 図
- **コードテンプレート** — 本番環境対応の Python コード、コピペでそのまま使える
- **パターン** — コード付きの一般的な設計パターン
- **落とし穴** — ハマりやすいポイント（対策付き）
- **実績あるパターン** — 本番環境から得られた貴重な知恵
- **関連スキル** — 関連スキルへのクロスリファレンス

---

## 学習パス

### 初級
```
Custom Agents → Function Calling → RAG Memory
```

### 中級
```
LangChain Agents → MCP Integration → Long-Term Memory → Task Decomposition
```

### 上級
```
AutoGen Multi-Agent → Agent Communication → Agent Orchestration → API Orchestration
```

---

## コントリビューション

コントリビューションを歓迎します！ガイドラインは [CONTRIBUTING.md](CONTRIBUTING.md) をご覧ください。

---

## ライセンス

本プロジェクトは MIT ライセンスで提供されています。詳細は [LICENSE](LICENSE) をご覧ください。

---

## 関連プロジェクト

他の `awesome-*` プロジェクトもぜひご覧ください：

| プロジェクト | 説明 |
|-------------|------|
| [awesome-ai-agents](https://github.com/liangzhengtao/awesome-ai-agents) | AI エージェント構築スキル |
| [awesome-llm-apps](https://github.com/liangzhengtao/awesome-llm-apps) | LLM を活用したアプリケーション集 |
| [awesome-rag](https://github.com/liangzhengtao/awesome-rag) | RAG 技術と実装 |
| [awesome-prompt-engineering](https://github.com/liangzhengtao/awesome-prompt-engineering) | プロンプトエンジニアリングガイドとテンプレート |
| [awesome-ai-coding](https://github.com/liangzhengtao/awesome-ai-coding) | AI コーディングツールとプラクティス |
| [awesome-gpt-store](https://github.com/liangzhengtao/awesome-gpt-store) | 厳選 GPT Store アプリケーション |
| [awesome-ai-api](https://github.com/liangzhengtao/awesome-ai-api) | AI API 統合ガイド |
| [awesome-llm-fine-tuning](https://github.com/liangzhengtao/awesome-llm-fine-tuning) | LLM ファインチューニング技術 |
| [awesome-vector-database](https://github.com/liangzhengtao/awesome-vector-database) | ベクトルデータベースの比較とガイド |
| [awesome-ai-safety](https://github.com/liangzhengtao/awesome-ai-safety) | AI セキュリティとアラインメントのリソース |
| [awesome-multimodal-ai](https://github.com/liangzhengtao/awesome-multimodal-ai) | マルチモーダル AI 技術 |
| [awesome-open-source-llm](https://github.com/liangzhengtao/awesome-open-source-llm) | オープンソース LLM コレクション |

---

<div align="center">

**AI エージェントコミュニティのために心を込めて構築しました。**

[⬆ トップに戻る](#awesome-ai-agents)

</div>
