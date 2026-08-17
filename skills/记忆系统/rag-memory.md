# RAG-Based Memory for Agents

> Give your agents the ability to retrieve relevant knowledge from large document collections using Retrieval-Augmented Generation — the most common memory pattern for production agents.

## When to Use

| Scenario | Use RAG Memory? |
|----------|----------------|
| Agent needs knowledge from large document sets | Yes |
| Answers must cite specific sources | Yes |
| Knowledge changes frequently | Yes (easy to update index) |
| Domain-specific expertise (legal, medical, finance) | Yes |
| Small, static knowledge base | Consider in-context prompting |
| Need reasoning over complex relationships | Consider knowledge graphs |
| Need episodic memory (past conversations) | Consider long-term memory |

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    RAG Memory System                      │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │               Ingestion Pipeline                  │   │
│  │  Documents → Chunking → Embedding → Vector Store  │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │               Query Pipeline                      │   │
│  │                                                    │   │
│  │  User Query                                        │   │
│  │      │                                             │   │
│  │      ▼                                             │   │
│  │  ┌─────────────┐   ┌───────────────────────┐     │   │
│  │  │  Embed      │──►│  Vector Similarity     │     │   │
│  │  │  Query      │   │  Search (Top-K)        │     │   │
│  │  └─────────────┘   └───────────┬───────────┘     │   │
│  │                                 │                   │   │
│  │                                 ▼                   │   │
│  │                     ┌───────────────────────┐     │   │
│  │                     │  Re-ranking            │     │   │
│  │                     │  (Cross-encoder)       │     │   │
│  │                     └───────────┬───────────┘     │   │
│  │                                 │                   │   │
│  │                                 ▼                   │   │
│  │                     ┌───────────────────────┐     │   │
│  │                     │  LLM generates answer  │     │   │
│  │                     │  with retrieved context │     │   │
│  │                     └───────────────────────┘     │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

## Code Template: Complete RAG Pipeline

```python
"""Production RAG pipeline with ChromaDB and OpenAI."""
import chromadb
from openai import OpenAI
from typing import Iterator

class RAGMemory:
    def __init__(self, collection_name: str = "agent_memory"):
        self.client = OpenAI()
        self.chroma = chromadb.PersistentClient(path="./chroma_db")
        self.collection = self.chroma.get_or_create_collection(
            name=collection_name,
            metadata={"hnsw:space": "cosine"},
        )

    def ingest(self, documents: list[dict], chunk_size: int = 500, overlap: int = 50):
        """Ingest documents into the vector store.

        Args:
            documents: List of {"id": str, "text": str, "metadata": dict}
            chunk_size: Characters per chunk
            overlap: Overlap between chunks
        """
        all_chunks = []
        for doc in documents:
            chunks = self._chunk_text(doc["text"], chunk_size, overlap)
            for i, chunk in enumerate(chunks):
                all_chunks.append({
                    "id": f"{doc['id']}_chunk_{i}",
                    "text": chunk,
                    "metadata": {**doc.get("metadata", {}), "source": doc["id"]},
                })

        # Batch embed and store
        for batch in self._batch(all_chunks, 100):
            texts = [c["text"] for c in batch]
            embeddings = self._embed(texts)
            self.collection.add(
                ids=[c["id"] for c in batch],
                documents=texts,
                embeddings=embeddings,
                metadatas=[c["metadata"] for c in batch],
            )
        print(f"Ingested {len(all_chunks)} chunks from {len(documents)} documents.")

    def query(self, question: str, top_k: int = 5) -> list[dict]:
        """Retrieve relevant chunks for a question."""
        query_embedding = self._embed([question])[0]
        results = self.collection.query(
            query_embeddings=[query_embedding],
            n_results=top_k,
            include=["documents", "metadatas", "distances"],
        )

        chunks = []
        for i in range(len(results["ids"][0])):
            chunks.append({
                "text": results["documents"][0][i],
                "metadata": results["metadatas"][0][i],
                "score": 1 - results["distances"][0][i],  # cosine similarity
            })
        return chunks

    def answer(self, question: str, top_k: int = 5) -> str:
        """Generate an answer using RAG."""
        chunks = self.query(question, top_k)
        context = "\n\n---\n\n".join(c["text"] for c in chunks)

        response = self.client.chat.completions.create(
            model="gpt-4o",
            messages=[
                {
                    "role": "system",
                    "content": "Answer based on the provided context. Cite sources. "
                               "If the context doesn't contain the answer, say so.",
                },
                {
                    "role": "user",
                    "content": f"Context:\n{context}\n\nQuestion: {question}",
                },
            ],
        )
        return response.choices[0].message.content

    def _embed(self, texts: list[str]) -> list[list[float]]:
        """Generate embeddings using OpenAI."""
        response = self.client.embeddings.create(
            model="text-embedding-3-small",
            input=texts,
        )
        return [item.embedding for item in response.data]

    def _chunk_text(self, text: str, size: int, overlap: int) -> list[str]:
        """Split text into overlapping chunks."""
        chunks = []
        start = 0
        while start < len(text):
            end = start + size
            chunk = text[start:end]
            # Try to break at sentence boundary
            if end < len(text):
                last_period = chunk.rfind(".")
                if last_period > size * 0.5:
                    chunk = chunk[:last_period + 1]
                    end = start + last_period + 1
            chunks.append(chunk.strip())
            start = end - overlap
        return [c for c in chunks if len(c) > 20]

    def _batch(self, items: list, size: int) -> Iterator[list]:
        for i in range(0, len(items), size):
            yield items[i : i + size]
```

## Code Template: RAG as an Agent Tool

```python
"""Expose RAG memory as a tool for any agent."""
from langchain_core.tools import tool

rag = RAGMemory(collection_name="company_docs")

@tool
def search_knowledge_base(query: str, top_k: int = 3) -> str:
    """Search the company knowledge base for relevant information.
    Use this when you need facts about company policies, products, or procedures."""
    chunks = rag.query(query, top_k=top_k)
    results = []
    for i, chunk in enumerate(chunks):
        source = chunk["metadata"].get("source", "unknown")
        results.append(f"[{i+1}] (source: {source}, score: {chunk['score']:.2f})\n{chunk['text']}")
    return "\n\n".join(results)

# Use in any agent
from langgraph.prebuilt import create_react_agent
agent = create_react_agent(llm, tools=[search_knowledge_base])
```

## Code Template: Hybrid Search (Vector + Keyword)

```python
"""Combine vector similarity with BM25 keyword search."""
from rank_bm25 import BM25Okapi
import numpy as np

class HybridRAG:
    def __init__(self, rag_memory: RAGMemory):
        self.rag = rag_memory
        self.corpus: list[str] = []
        self.bm25: BM25Okapi | None = None

    def build_index(self):
        """Build BM25 index from the vector store."""
        all_docs = self.rag.collection.get(include=["documents"])
        self.corpus = all_docs["documents"]
        tokenized = [doc.lower().split() for doc in self.corpus]
        self.bm25 = BM25Okapi(tokenized)

    def query(self, question: str, top_k: int = 5, alpha: float = 0.7) -> list[dict]:
        """Hybrid search: alpha * vector + (1-alpha) * BM25."""
        # Vector search
        vector_results = self.rag.query(question, top_k=top_k * 2)

        # BM25 search
        scores = self.bm25.get_scores(question.lower().split())
        top_indices = np.argsort(scores)[-top_k * 2:][::-1]
        bm25_results = [
            {"text": self.corpus[i], "score": float(scores[i])}
            for i in top_indices
        ]

        # Normalize and merge scores
        combined = {}
        for r in vector_results:
            key = r["text"][:100]  # Use text prefix as key
            combined[key] = {"text": r["text"], "score": alpha * r["score"]}

        for r in bm25_results:
            key = r["text"][:100]
            if key in combined:
                combined[key]["score"] += (1 - alpha) * self._normalize(r["score"], scores)
            else:
                combined[key] = {"text": r["text"], "score": (1 - alpha) * self._normalize(r["score"], scores)}

        # Sort by combined score
        results = sorted(combined.values(), key=lambda x: x["score"], reverse=True)
        return results[:top_k]

    def _normalize(self, value, all_scores):
        max_s = max(all_scores)
        return value / max_s if max_s > 0 else 0
```

## Patterns

### Pattern: Multi-Collection RAG
```python
"""Separate collections for different knowledge domains."""
class MultiCollectionRAG:
    def __init__(self):
        self.collections = {
            "legal": RAGMemory("legal_docs"),
            "technical": RAGMemory("tech_docs"),
            "support": RAGMemory("support_tickets"),
        }

    def query(self, question: str, domain: str = None) -> str:
        if domain:
            return self.collections[domain].answer(question)
        # Query all and merge
        results = []
        for name, rag in self.collections.items():
            chunks = rag.query(question, top_k=2)
            for c in chunks:
                c["domain"] = name
            results.extend(chunks)
        results.sort(key=lambda x: x["score"], reverse=True)
        return results[:5]
```

### Pattern: Conversational RAG with History
```python
"""Rewrite queries using conversation context before retrieval."""
def contextual_query(question: str, history: list[dict], rag: RAGMemory) -> str:
    # Rewrite the question to be standalone
    rewrite = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "Rewrite the question to be standalone, incorporating context from the conversation."},
            *history[-4:],
            {"role": "user", "content": f"Rewrite: {question}"},
        ],
    )
    standalone_query = rewrite.choices[0].message.content
    return rag.answer(standalone_query)
```

### Pattern: Incremental Ingestion
```python
"""Only embed new or changed documents."""
import hashlib

def incremental_ingest(rag: RAGMemory, documents: list[dict]):
    existing = set(rag.collection.get()["ids"])
    new_docs = [d for d in documents if d["id"] not in existing]
    if new_docs:
        rag.ingest(new_docs)
        print(f"Added {len(new_docs)} new documents")
```

## Pitfalls

| Pitfall | Problem | Solution |
|---------|---------|----------|
| Bad chunking | Chunks split mid-sentence | Use sentence-aware chunking with overlap |
| Poor embeddings | Generic model doesn't fit domain | Fine-tune embeddings or use domain-specific models |
| No re-ranking | Top-K vector results aren't the best | Add a cross-encoder re-ranker |
| Stale index | Documents change but index doesn't | Implement incremental updates |
| Context overflow | Too many chunks in prompt | Limit to 3-5 most relevant chunks |
| Hallucination | LLM ignores context and makes things up | Strong system prompt + citation requirements |

## Best Practices

1. **Chunk size matters** — 300-800 characters works well for most use cases
2. **Use overlap** — 50-100 character overlap prevents losing context at boundaries
3. **Store metadata** — source, page number, timestamp for citations
4. **Re-rank results** — vector search is approximate; cross-encoders are more accurate
5. **Test retrieval quality** — measure recall@K before optimizing generation
6. **Use hybrid search** — combining vector + BM25 catches more relevant results

## Dependencies

```bash
pip install chromadb openai rank-bm25
# Alternative vector stores
pip install pinecone-client  # or: qdrant-client, weaviate-client
```

## See Also

- [Long-Term Memory](./long-term-memory.md) — Persistent agent memory
- [MCP Integration](../工具调用/mcp-integration.md) — Expose RAG as MCP tool
- [Function Calling](../工具调用/function-calling.md) — Connect RAG to agents
- [Custom Agents](../Agent框架/custom-agents.md) — Building the agent loop
