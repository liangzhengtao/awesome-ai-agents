# Long-Term Memory and Knowledge Graphs

> Build agents that remember, learn, and accumulate knowledge over time — beyond simple RAG, using knowledge graphs, episodic memory, and semantic memory systems.

## When to Use

| Scenario | Use Long-Term Memory? |
|----------|----------------------|
| Agent needs to remember across sessions | Yes |
| Knowledge has complex relationships | Yes (knowledge graph) |
| Agent should learn user preferences over time | Yes |
| Need to track facts, not just documents | Yes |
| Short document retrieval | RAG is simpler and sufficient |
| Stateless, single-turn interactions | No memory needed |

## Memory Types

```
┌─────────────────────────────────────────────────────────┐
│                   Agent Memory Layers                     │
│                                                          │
│  ┌────────────────────────────────────────────────────┐ │
│  │  Working Memory (Context Window)                    │ │
│  │  Current conversation, active facts                 │ │
│  │  Capacity: ~128K tokens                            │ │
│  └────────────────────────┬───────────────────────────┘ │
│                           │ flush                        │
│  ┌────────────────────────▼───────────────────────────┐ │
│  │  Episodic Memory                                    │ │
│  │  Past conversations, user interactions              │ │
│  │  Storage: Vector DB + timestamps                    │ │
│  └────────────────────────┬───────────────────────────┘ │
│                           │ extract                      │
│  ┌────────────────────────▼───────────────────────────┐ │
│  │  Semantic Memory                                    │ │
│  │  Facts, relationships, learned knowledge            │ │
│  │  Storage: Knowledge Graph (entities + relations)    │ │
│  └────────────────────────┬───────────────────────────┘ │
│                           │ query                        │
│  ┌────────────────────────▼───────────────────────────┐ │
│  │  Procedural Memory                                  │ │
│  │  Learned patterns, successful strategies            │ │
│  │  Storage: Fine-tuned weights / prompt library       │ │
│  └────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

## Code Template: Knowledge Graph Memory

```python
"""Knowledge graph-based memory using NetworkX."""
import json
import networkx as nx
from openai import OpenAI

class KnowledgeGraphMemory:
    def __init__(self, persist_path: str = "knowledge_graph.json"):
        self.graph = nx.DiGraph()
        self.persist_path = persist_path
        self.client = OpenAI()
        self._load()

    def add_memory(self, text: str, source: str = "conversation"):
        """Extract entities and relations from text and add to graph."""
        extraction = self.client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[
                {
                    "role": "system",
                    "content": """Extract entities and relationships from the text.
                    Return JSON:
                    {
                        "entities": [{"name": str, "type": str, "description": str}],
                        "relations": [{"source": str, "target": str, "relation": str}]
                    }""",
                },
                {"role": "user", "content": text},
            ],
            response_format={"type": "json_object"},
        )

        data = json.loads(extraction.choices[0].message.content)

        # Add entities as nodes
        for entity in data.get("entities", []):
            if self.graph.has_node(entity["name"]):
                # Merge descriptions
                existing = self.graph.nodes[entity["name"]]
                existing["description"] = f"{existing.get('description', '')} | {entity['description']}"
                existing["mentions"] = existing.get("mentions", 0) + 1
            else:
                self.graph.add_node(
                    entity["name"],
                    type=entity["type"],
                    description=entity["description"],
                    mentions=1,
                    source=source,
                )

        # Add relations as edges
        for rel in data.get("relations", []):
            self.graph.add_edge(
                rel["source"],
                rel["target"],
                relation=rel["relation"],
                source=source,
            )

        self._save()

    def query(self, entity: str) -> dict:
        """Get everything known about an entity."""
        if entity not in self.graph:
            return {"found": False, "entity": entity}

        node = self.graph.nodes[entity]
        neighbors = []
        for neighbor in self.graph.neighbors(entity):
            edge = self.graph[entity][neighbor]
            neighbors.append({
                "entity": neighbor,
                "relation": edge.get("relation", "related_to"),
                "type": self.graph.nodes[neighbor].get("type", "unknown"),
            })
        for predecessor in self.graph.predecessors(entity):
            edge = self.graph[predecessor][entity]
            neighbors.append({
                "entity": predecessor,
                "relation": f"inverse: {edge.get('relation', 'related_to')}",
                "type": self.graph.nodes[predecessor].get("type", "unknown"),
            })

        return {
            "found": True,
            "entity": entity,
            "description": node.get("description", ""),
            "type": node.get("type", ""),
            "connections": neighbors,
            "mentions": node.get("mentions", 1),
        }

    def search(self, query: str, max_hops: int = 2) -> list[dict]:
        """Find relevant subgraph for a query."""
        # Extract entities from query
        extraction = self.client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[
                {"role": "system", "content": "Extract entity names from the query as a JSON list."},
                {"role": "user", "content": query},
            ],
            response_format={"type": "json_object"},
        )
        entities = json.loads(extraction.choices[0].message.content).get("entities", [])

        # BFS from each entity
        relevant = set()
        for entity in entities:
            if entity in self.graph:
                relevant.add(entity)
                for hop in range(max_hops):
                    neighbors = set()
                    for node in relevant:
                        neighbors.update(self.graph.neighbors(node))
                        neighbors.update(self.graph.predecessors(node))
                    relevant.update(neighbors)

        # Build subgraph
        results = []
        for node in relevant:
            info = self.query(node)
            results.append(info)
        return results

    def _save(self):
        data = nx.node_link_data(self.graph)
        with open(self.persist_path, "w") as f:
            json.dump(data, f, indent=2)

    def _load(self):
        try:
            with open(self.persist_path) as f:
                data = json.load(f)
                self.graph = nx.node_link_graph(data)
        except FileNotFoundError:
            pass
```

## Code Template: Episodic Memory with Vector Store

```python
"""Store and retrieve past conversation episodes."""
import chromadb
from datetime import datetime
from openai import OpenAI

class EpisodicMemory:
    def __init__(self, user_id: str):
        self.user_id = user_id
        self.client = OpenAI()
        self.chroma = chromadb.PersistentClient(path="./episodic_memory")
        self.collection = self.chroma.get_or_create_collection(
            name=f"episodes_{user_id}",
            metadata={"hnsw:space": "cosine"},
        )

    def store_episode(self, messages: list[dict], summary: str = None):
        """Store a conversation episode."""
        if not summary:
            summary = self._summarize(messages)

        # Embed the summary
        embedding = self._embed(summary)

        episode_id = f"ep_{datetime.now().isoformat()}"
        self.collection.add(
            ids=[episode_id],
            documents=[summary],
            embeddings=[embedding],
            metadatas=[{
                "user_id": self.user_id,
                "timestamp": datetime.now().isoformat(),
                "turn_count": len(messages),
                "raw_messages": json.dumps(messages[-6:]),  # Last 3 turns
            }],
        )
        return episode_id

    def recall(self, query: str, top_k: int = 3) -> list[dict]:
        """Recall relevant past episodes."""
        embedding = self._embed(query)
        results = self.collection.query(
            query_embeddings=[embedding],
            n_results=top_k,
            include=["documents", "metadatas", "distances"],
        )

        episodes = []
        for i in range(len(results["ids"][0])):
            episodes.append({
                "summary": results["documents"][0][i],
                "timestamp": results["metadatas"][0][i]["timestamp"],
                "relevance": 1 - results["distances"][0][i],
                "recent_messages": json.loads(
                    results["metadatas"][0][i].get("raw_messages", "[]")
                ),
            })
        return episodes

    def _summarize(self, messages: list[dict]) -> str:
        response = self.client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[
                {"role": "system", "content": "Summarize this conversation in 2-3 sentences, focusing on key topics, decisions, and outcomes."},
                {"role": "user", "content": json.dumps(messages)},
            ],
        )
        return response.choices[0].message.content

    def _embed(self, text: str) -> list[float]:
        response = self.client.embeddings.create(
            model="text-embedding-3-small", input=[text]
        )
        return response.data[0].embedding
```

## Code Template: Unified Memory System

```python
"""Combine all memory types into a single agent memory system."""
class AgentMemory:
    def __init__(self, user_id: str):
        self.kg = KnowledgeGraphMemory(persist_path=f"kg_{user_id}.json")
        self.episodic = EpisodicMemory(user_id)
        self.working_memory: list[dict] = []  # Current context

    def observe(self, message: dict):
        """Process a new message through all memory layers."""
        self.working_memory.append(message)

        # Extract facts for knowledge graph
        self.kg.add_memory(message["content"], source="conversation")

        # Flush working memory to episodic if too long
        if len(self.working_memory) > 20:
            self.episodic.store_episode(self.working_memory)
            self.working_memory = self.working_memory[-6:]  # Keep last 3 turns

    def recall(self, query: str) -> dict:
        """Gather relevant context from all memory layers."""
        # Search knowledge graph for facts
        kg_results = self.kg.search(query)

        # Search episodic memory for past conversations
        episodes = self.episodic.recall(query, top_k=2)

        # Format for LLM context
        context_parts = []

        if kg_results:
            facts = "\n".join(
                f"- {r['entity']}: {r.get('description', 'N/A')}"
                for r in kg_results[:5]
            )
            context_parts.append(f"Known Facts:\n{facts}")

        if episodes:
            past = "\n".join(
                f"- [{e['timestamp'][:10]}] {e['summary']}"
                for e in episodes
            )
            context_parts.append(f"Past Conversations:\n{past}")

        return {
            "context": "\n\n".join(context_parts),
            "kg_entities": len(kg_results),
            "episodes_found": len(episodes),
        }

    def get_context_window(self) -> list[dict]:
        """Get working memory formatted for LLM input."""
        return self.working_memory
```

## Patterns

### Pattern: Automatic Fact Extraction
```python
"""Passively extract facts from every user message."""
def extract_and_store(memory: AgentMemory, user_msg: str):
    # Use a cheap model for extraction
    facts = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{
            "role": "system",
            "content": "Extract factual claims from this message. Return JSON list. "
                       "Only extract new information, not questions.",
        }, {"role": "user", "content": user_msg}],
    )
    for fact in json.loads(facts.choices[0].message.content):
        memory.kg.add_memory(fact, source="user_conversation")
```

### Pattern: Memory Consolidation
```python
"""Periodically merge and deduplicate knowledge graph nodes."""
def consolidate_memory(memory: AgentMemory):
    G = memory.kg.graph
    # Find similar nodes
    nodes = list(G.nodes(data=True))
    for i, (n1, d1) in enumerate(nodes):
        for n2, d2 in nodes[i+1:]:
            if d1.get("type") == d2.get("type"):
                # Check similarity
                sim = cosine_similarity(embed(n1), embed(n2))
                if sim > 0.9:
                    # Merge n2 into n1
                    for _, _, edge_data in G.edges(n2, data=True):
                        G.add_edge(n1, _, **edge_data)
                    G.remove_node(n2)
```

### Pattern: Forgetting Old Memories
```python
"""Remove memories that haven't been accessed recently."""
def forget_old_memories(memory: AgentMemory, days: int = 90):
    cutoff = datetime.now() - timedelta(days=days)
    to_remove = []
    for node, data in memory.kg.graph.nodes(data=True):
        last_accessed = data.get("last_accessed")
        if last_accessed and datetime.fromisoformat(last_accessed) < cutoff:
            if data.get("mentions", 0) < 3:  # Only forget rarely-mentioned
                to_remove.append(node)
    for node in to_remove:
        memory.kg.graph.remove_node(node)
```

## Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| Entity extraction errors | LLM extracts wrong entities | Validate and deduplicate; use structured output |
| Graph explosion | Too many nodes to query efficiently | Prune low-mention nodes; use graph partitioning |
| Memory conflicts | Contradictory facts stored | Track timestamps; prefer recent facts |
| Privacy concerns | Sensitive data in memory | Implement data retention policies and deletion |
| Retrieval latency | Graph queries are slow | Cache frequent queries; limit search depth |
| Context window overflow | Too much memory in prompt | Prioritize by recency and relevance |

## Proven Patterns

1. **Extract facts, not just store text** — knowledge graphs are more useful than raw text
2. **Track provenance** — where each fact came from and when
3. **Implement forgetting** — not all memories are worth keeping forever
4. **Use cheap models for extraction** — `gpt-4o-mini` is sufficient for entity extraction
5. **Consolidate periodically** — merge duplicate entities and resolve conflicts
6. **Respect privacy** — allow users to view and delete their memory

## Dependencies

```bash
pip install networkx chromadb openai
# For graph visualization
pip install pyvis
```

## See Also

- [RAG Memory](./rag-memory.md) — Document retrieval-based memory
- [Function Calling](../工具调用/function-calling.md) — Connect memory to agents
- [Custom Agents](../Agent框架/custom-agents.md) — Building agents with memory
- [Agent Communication](../多Agent协作/agent-communication.md) — Shared memory between agents

---

## 中文版本

### 使用场景

- Agent 需要跨会话记忆
- 知识具有复杂关系（知识图谱）
- Agent 应随时间学习用户偏好
- 需要追踪事实而非仅文档

> 短文档检索使用 RAG 更简单；无状态单轮交互不需要记忆系统。

### 核心步骤

1. **知识图谱记忆** — 使用 NetworkX 构建实体-关系图，LLM 从对话中提取实体和关系并添加到图中
2. **情景记忆** — 使用向量数据库存储对话摘要，通过语义搜索回忆相关历史对话
3. **统一记忆系统** — 组合知识图谱（事实）+ 情景记忆（对话）+ 工作记忆（当前上下文）
4. **自动事实提取** — 从每条用户消息中被动提取事实，存入知识图谱
5. **记忆整合** — 定期合并重复实体、删除长期未访问的记忆

### 模板说明

- KnowledgeGraphMemory — NetworkX 知识图谱，支持实体添加、查询、子图搜索
- EpisodicMemory — ChromaDB 存储对话摘要，支持语义搜索回忆
- AgentMemory — 统一记忆系统，observe 处理新消息，recall 从所有层收集上下文
- 记忆模式 — 自动事实提取、记忆整合、遗忘旧记忆

### 常见陷阱

1. **实体提取错误** — LLM 提取错误实体，使用结构化输出验证并去重
2. **图爆炸** — 节点过多查询效率下降，剪枝低提及节点，使用图分区
3. **记忆冲突** — 存储矛盾事实，追踪时间戳，优先使用最新事实
4. **隐私问题** — 记忆中包含敏感数据，实现数据保留策略和删除机制
5. **检索延迟** — 图查询较慢，缓存频繁查询，限制搜索深度
