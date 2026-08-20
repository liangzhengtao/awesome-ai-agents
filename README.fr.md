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

> **Arrêtez de construire des chatbots basiques. 12 compétences pour construire des agents IA de production.**

Une collection de compétences éprouvées pour la construction d'agents IA — de l'appel d'outil mono-agent à l'orchestration multi-agents. Chaque compétence est un guide complet, prêt à copier-coller, avec des diagrammes d'architecture, des modèles de code, des patterns et des pièges à éviter.

---

## Pourquoi ce projet ?

L'écosystème des agents IA en 2026 est fragmenté — LangChain, AutoGen, CrewAI, MCP, solutions personnalisées — chacun avec ses propres API et patterns. Ce projet distille les **compétences fondamentales** dont tout développeur d'agents a besoin, indépendamment du choix de framework.

---

## Tableau des compétences

### Frameworks d'agents

| # | Compétence | Description | Chemin |
|---|-----------|-------------|--------|
| 1 | **LangChain Agents** | Construire des agents avec LangChain et LangGraph | [`skills/Agent框架/langchain-agents.md`](skills/Agent框架/langchain-agents.md) |
| 2 | **AutoGen Multi-Agent** | Conversations multi-agents avec Microsoft AutoGen | [`skills/Agent框架/autogen-multiagent.md`](skills/Agent框架/autogen-multiagent.md) |
| 3 | **CrewAI Agents** | Orchestration d'agents basée sur les rôles avec CrewAI | [`skills/Agent框架/crewai-agents.md`](skills/Agent框架/crewai-agents.md) |
| 4 | **Custom Agents** | Construire des agents de zéro sans framework | [`skills/Agent框架/custom-agents.md`](skills/Agent框架/custom-agents.md) |

### Utilisation d'outils

| # | Compétence | Description | Chemin |
|---|-----------|-------------|--------|
| 5 | **Function Calling** | Appels de fonctions et utilisation d'outils multi-fournisseurs | [`skills/工具调用/function-calling.md`](skills/工具调用/function-calling.md) |
| 6 | **MCP Integration** | Intégration de serveurs Model Context Protocol | [`skills/工具调用/mcp-integration.md`](skills/工具调用/mcp-integration.md) |
| 7 | **API Orchestration** | Combinaison de multiples APIs en workflows d'agents | [`skills/工具调用/api-orchestration.md`](skills/工具调用/api-orchestration.md) |

### Systèmes de mémoire

| # | Compétence | Description | Chemin |
|---|-----------|-------------|--------|
| 8 | **RAG Memory** | Mémoire RAG pour agents | [`skills/记忆系统/rag-memory.md`](skills/记忆系统/rag-memory.md) |
| 9 | **Long-Term Memory** | Graphes de connaissances et mémoire persistante | [`skills/记忆系统/long-term-memory.md`](skills/记忆系统/long-term-memory.md) |

### Multi-Agents

| # | Compétence | Description | Chemin |
|---|-----------|-------------|--------|
| 10 | **Agent Communication** | Protocoles de messagerie inter-agents | [`skills/多Agent协作/agent-communication.md`](skills/多Agent协作/agent-communication.md) |
| 11 | **Task Decomposition** | Découper des tâches complexes en sous-tâches | [`skills/多Agent协作/task-decomposition.md`](skills/多Agent协作/task-decomposition.md) |
| 12 | **Agent Orchestration** | Coordonner plusieurs agents dans des workflows | [`skills/多Agent协作/agent-orchestration.md`](skills/多Agent协作/agent-orchestration.md) |

---

## Démarrage rapide

### Utilisation avec Cursor

1. Cloner ce dépôt :
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. Ouvrir Cursor → Paramètres → Règles → Ajouter `awesome-ai-agents/skills/` comme contexte
3. Demander à Cursor : *"Lis la compétence langchain-agents et construis-moi un agent avec recherche web"*

### Utilisation avec Claude Code

1. Cloner ce dépôt :
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. Dans votre projet, référencer la compétence :
   ```bash
   # Copier la compétence dont vous avez besoin
   cp awesome-ai-agents/skills/Agent框架/custom-agents.md .claude/agents/
   ```
3. Ou référencer directement :
   ```
   Lis awesome-ai-agents/skills/工具调用/function-calling.md et implémente-le dans mon projet
   ```

### Utilisation avec Kimi Code

1. Cloner ce dépôt :
   ```bash
   git clone https://github.com/liangzhengtao/awesome-ai-agents.git
   ```
2. Ajouter comme répertoire de compétences :
   ```bash
   cp -r awesome-ai-agents/skills/ ~/.agents/skills/ai-agents/
   ```
3. Ou référencer dans une conversation :
   ```
   Lis la compétence awesome-ai-agents/skills/记忆系统/rag-memory.md et configure la RAG pour mon projet
   ```

### Utilisation comme référence

Chaque fichier de compétence est autonome. Parcourez le répertoire skills, choisissez celui dont vous avez besoin, et suivez le diagramme d'architecture, les modèles de code et les patterns.

---

## Contenu de chaque compétence

Chaque fichier de compétence suit la même structure :

- **Quand l'utiliser** — Matrice de décision claire pour savoir quand cette compétence s'applique
- **Architecture** — Diagramme ASCII montrant la conception du système
- **Modèles de code** — Code Python de production, prêt à copier-coller
- **Patterns** — Patterns de conception courants avec du code
- **Pièges** — Ce qui peut vous poser problème (avec des solutions)
- **Patterns éprouvés** — Sagesse de production acquise sur le terrain
- **Voir aussi** — Références croisées vers des compétences connexes

---

## Parcours d'apprentissage

### Débutant
```
Custom Agents → Function Calling → RAG Memory
```

### Intermédiaire
```
LangChain Agents → MCP Integration → Long-Term Memory → Task Decomposition
```

### Avancé
```
AutoGen Multi-Agent → Agent Communication → Agent Orchestration → API Orchestration
```

---

## Contribuer

Les contributions sont les bienvenues ! Consultez [CONTRIBUTING.md](CONTRIBUTING.md) pour les directives.

---

## Licence

Ce projet est sous licence MIT — voir [LICENSE](LICENSE) pour les détails.

---

## Voir aussi

Découvrez nos autres projets `awesome-*` :

| Projet | Description |
|--------|-------------|
| [awesome-ai-agents](https://github.com/liangzhengtao/awesome-ai-agents) | Compétences pour construire des agents IA |
| [awesome-llm-apps](https://github.com/liangzhengtao/awesome-llm-apps) | Collection d'applications alimentées par des LLM |
| [awesome-rag](https://github.com/liangzhengtao/awesome-rag) | Techniques et implémentations RAG |
| [awesome-prompt-engineering](https://github.com/liangzhengtao/awesome-prompt-engineering) | Guides et modèles de prompt engineering |
| [awesome-ai-coding](https://github.com/liangzhengtao/awesome-ai-coding) | Outils et pratiques de codage IA |
| [awesome-gpt-store](https://github.com/liangzhengtao/awesome-gpt-store) | Applications GPT Store sélectionnées |
| [awesome-ai-api](https://github.com/liangzhengtao/awesome-ai-api) | Guides d'intégration d'APIs IA |
| [awesome-llm-fine-tuning](https://github.com/liangzhengtao/awesome-llm-fine-tuning) | Techniques de fine-tuning LLM |
| [awesome-vector-database](https://github.com/liangzhengtao/awesome-vector-database) | Comparaisons et guides de bases de données vectorielles |
| [awesome-ai-safety](https://github.com/liangzhengtao/awesome-ai-safety) | Ressources sur la sécurité et l'alignement IA |
| [awesome-multimodal-ai](https://github.com/liangzhengtao/awesome-multimodal-ai) | Techniques d'IA multimodale |
| [awesome-open-source-llm](https://github.com/liangzhengtao/awesome-open-source-llm) | Collection de LLM open-source |

---

<div align="center">

**Construit avec soin pour la communauté des agents IA.**

[⬆ Retour en haut](#awesome-ai-agents)

</div>
