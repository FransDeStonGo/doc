---
type: guide
tags: [rag, embeddings, vector-search, retrieval, reranking]
status: complete
updated: 2026-03-25
related:
  - agents/examples/rag-agent.md
  - ai/embeddings.md
  - infrastructure/vector-stores.md
---

# RAG Implementation Guide

Complete guide to building production-ready Retrieval-Augmented Generation systems.

---

## Overview

RAG enhances LLM responses with external knowledge through:
1. **Chunking** — split documents into retrievable pieces
2. **Embedding** — convert text to vectors
3. **Indexing** — store vectors in a database
4. **Retrieval** — find relevant chunks for a query
5. **Reranking** — improve relevance ordering
6. **Generation** — LLM synthesizes answer from context

---

## 1. Chunking Strategies

### Fixed-Size Chunking
```python
def fixed_chunk(text: str, chunk_size: int = 500, overlap: int = 50) -> list[str]:
    """Simple word-based chunking with overlap."""
    words = text.split()
    chunks = []
    for i in range(0, len(words), chunk_size - overlap):
        chunk = ' '.join(words[i:i + chunk_size])
        chunks.append(chunk)
    return chunks

# Usage
text = "Long document..."
chunks = fixed_chunk(text, chunk_size=500, overlap=50)
```

**Pros:** Simple, fast  
**Cons:** Breaks semantic boundaries

---

### Semantic Chunking
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

def semantic_chunk(text: str, chunk_size: int = 1000, overlap: int = 200) -> list[str]:
    """Split on semantic boundaries (paragraphs, sentences)."""
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=chunk_size,
        chunk_overlap=overlap,
        separators=["\n\n", "\n", ". ", " ", ""]
    )
    return splitter.split_text(text)

# Usage
chunks = semantic_chunk(text, chunk_size=1000, overlap=200)
```

**Pros:** Preserves context  
**Cons:** Variable chunk sizes

---

### Markdown-Aware Chunking
```python
def markdown_chunk(text: str, max_tokens: int = 500) -> list[dict]:
    """Chunk by markdown structure (headers, code blocks)."""
    chunks = []
    current_section = {"header": "", "content": "", "level": 0}
    
    for line in text.split('\n'):
        if line.startswith('#'):
            if current_section["content"]:
                chunks.append(current_section)
            level = len(line.split()[0])
            current_section = {
                "header": line.strip('# '),
                "content": "",
                "level": level
            }
        else:
            current_section["content"] += line + '\n'
    
    if current_section["content"]:
        chunks.append(current_section)
    
    return chunks
```

**Pros:** Preserves document structure  
**Cons:** Requires markdown format

---

## 2. Embedding Models

### Comparison Table

| Model | Dimensions | Speed | Quality | Cost | Use Case |
|-------|-----------|-------|---------|------|----------|
| OpenAI text-embedding-3-small | 1536 | Fast | Good | $0.02/1M | General purpose |
| OpenAI text-embedding-3-large | 3072 | Medium | Excellent | $0.13/1M | High accuracy |
| Cohere embed-english-v3.0 | 1024 | Fast | Excellent | $0.10/1M | English only |
| nomic-embed-text (local) | 768 | Very fast | Good | Free | Privacy-first |
| bge-m3 (local) | 1024 | Fast | Good | Free | Multilingual |

---

### OpenAI Embeddings
```python
from openai import OpenAI

client = OpenAI(api_key="sk-...")

def embed_openai(texts: list[str], model: str = "text-embedding-3-small") -> list[list[float]]:
    """Generate embeddings via OpenAI API."""
    response = client.embeddings.create(
        input=texts,
        model=model
    )
    return [item.embedding for item in response.data]

# Usage
embeddings = embed_openai(["Hello world", "RAG is cool"])
```

---

### Local Embeddings (Ollama)
```python
import requests

def embed_local(texts: list[str], model: str = "nomic-embed-text") -> list[list[float]]:
    """Generate embeddings via local Ollama."""
    embeddings = []
    for text in texts:
        response = requests.post(
            "http://localhost:11434/api/embeddings",
            json={"model": model, "prompt": text}
        )
        embeddings.append(response.json()["embedding"])
    return embeddings

# Usage
embeddings = embed_local(["Hello world", "RAG is cool"])
```

---

## 3. Vector Stores

### Qdrant Setup
```python
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct

# Initialize client
client = QdrantClient(host="localhost", port=6333)

# Create collection
client.create_collection(
    collection_name="documents",
    vectors_config=VectorParams(size=768, distance=Distance.COSINE)
)

# Index documents
points = [
    PointStruct(
        id=i,
        vector=embedding,
        payload={"text": chunk, "source": "doc.md", "chunk_id": i}
    )
    for i, (chunk, embedding) in enumerate(zip(chunks, embeddings))
]

client.upsert(collection_name="documents", points=points)
```

---

### Pinecone Setup
```python
from pinecone import Pinecone, ServerlessSpec

# Initialize
pc = Pinecone(api_key="...")
index_name = "documents"

# Create index
pc.create_index(
    name=index_name,
    dimension=1536,
    metric="cosine",
    spec=ServerlessSpec(cloud="aws", region="us-east-1")
)

index = pc.Index(index_name)

# Upsert vectors
vectors = [
    (f"doc_{i}", embedding, {"text": chunk, "source": "doc.md"})
    for i, (chunk, embedding) in enumerate(zip(chunks, embeddings))
]

index.upsert(vectors=vectors)
```

---

## 4. Retrieval Strategies

### Similarity Search
```python
def similarity_search(query: str, top_k: int = 5) -> list[dict]:
    """Basic vector similarity search."""
    query_embedding = embed_openai([query])[0]
    
    results = client.search(
        collection_name="documents",
        query_vector=query_embedding,
        limit=top_k
    )
    
    return [
        {
            "text": hit.payload["text"],
            "score": hit.score,
            "source": hit.payload["source"]
        }
        for hit in results
    ]
```

---

### MMR (Maximal Marginal Relevance)
```python
def mmr_search(query: str, top_k: int = 5, lambda_param: float = 0.5) -> list[dict]:
    """Retrieve diverse results using MMR."""
    query_embedding = embed_openai([query])[0]
    
    # Get initial candidates (2x top_k)
    candidates = client.search(
        collection_name="documents",
        query_vector=query_embedding,
        limit=top_k * 2
    )
    
    selected = []
    candidate_embeddings = [c.vector for c in candidates]
    
    for _ in range(top_k):
        if not candidates:
            break
        
        # Calculate MMR scores
        mmr_scores = []
        for i, cand in enumerate(candidates):
            relevance = cand.score
            
            # Max similarity to already selected
            if selected:
                max_sim = max(
                    cosine_similarity(candidate_embeddings[i], s.vector)
                    for s in selected
                )
            else:
                max_sim = 0
            
            mmr = lambda_param * relevance - (1 - lambda_param) * max_sim
            mmr_scores.append((mmr, cand))
        
        # Select best MMR score
        best = max(mmr_scores, key=lambda x: x[0])[1]
        selected.append(best)
        candidates.remove(best)
    
    return [{"text": s.payload["text"], "score": s.score} for s in selected]
```

---

### Hybrid Search (Vector + Keyword)
```python
def hybrid_search(query: str, top_k: int = 5, alpha: float = 0.5) -> list[dict]:
    """Combine vector similarity with BM25 keyword search."""
    # Vector search
    query_embedding = embed_openai([query])[0]
    vector_results = client.search(
        collection_name="documents",
        query_vector=query_embedding,
        limit=top_k * 2
    )
    
    # Keyword search (BM25)
    keyword_results = client.scroll(
        collection_name="documents",
        scroll_filter={"must": [{"key": "text", "match": {"text": query}}]},
        limit=top_k * 2
    )[0]
    
    # Combine scores
    combined = {}
    for result in vector_results:
        doc_id = result.id
        combined[doc_id] = alpha * result.score
    
    for result in keyword_results:
        doc_id = result.id
        if doc_id in combined:
            combined[doc_id] += (1 - alpha) * 1.0  # BM25 score normalized
        else:
            combined[doc_id] = (1 - alpha) * 1.0
    
    # Sort and return top_k
    sorted_results = sorted(combined.items(), key=lambda x: x[1], reverse=True)[:top_k]
    
    return [
        {"text": client.retrieve(collection_name="documents", ids=[doc_id])[0].payload["text"], "score": score}
        for doc_id, score in sorted_results
    ]
```

---

## 5. Reranking

### Cross-Encoder Reranking
```python
from sentence_transformers import CrossEncoder

model = CrossEncoder('cross-encoder/ms-marco-MiniLM-L-6-v2')

def rerank_cross_encoder(query: str, documents: list[str], top_k: int = 5) -> list[dict]:
    """Rerank using cross-encoder model."""
    # Score all query-document pairs
    pairs = [[query, doc] for doc in documents]
    scores = model.predict(pairs)
    
    # Sort by score
    ranked = sorted(zip(documents, scores), key=lambda x: x[1], reverse=True)
    
    return [{"text": doc, "score": float(score)} for doc, score in ranked[:top_k]]
```

---

### LLM-Based Reranking
```python
from anthropic import Anthropic

client = Anthropic(api_key="sk-...")

def rerank_llm(query: str, documents: list[str], top_k: int = 5) -> list[dict]:
    """Rerank using LLM relevance scoring."""
    prompt = f"""Rate the relevance of each document to the query on a scale of 0-10.

Query: {query}

Documents:
{chr(10).join(f"{i+1}. {doc[:200]}..." for i, doc in enumerate(documents))}

Return only a JSON array of scores: [score1, score2, ...]"""

    response = client.messages.create(
        model="claude-3-haiku-20240307",
        max_tokens=100,
        messages=[{"role": "user", "content": prompt}]
    )
    
    scores = eval(response.content[0].text)  # Parse JSON array
    ranked = sorted(zip(documents, scores), key=lambda x: x[1], reverse=True)
    
    return [{"text": doc, "score": score} for doc, score in ranked[:top_k]]
```

---

## 6. Complete RAG Pipeline

```python
class RAGPipeline:
    def __init__(self, collection_name: str = "documents"):
        self.collection_name = collection_name
        self.qdrant = QdrantClient(host="localhost", port=6333)
        self.anthropic = Anthropic(api_key="sk-...")
    
    def index_documents(self, documents: list[str], metadata: list[dict] = None):
        """Chunk, embed, and index documents."""
        all_chunks = []
        all_metadata = []
        
        for i, doc in enumerate(documents):
            chunks = semantic_chunk(doc, chunk_size=1000, overlap=200)
            embeddings = embed_openai(chunks)
            
            for j, (chunk, embedding) in enumerate(zip(chunks, embeddings)):
                all_chunks.append(chunk)
                all_metadata.append({
                    "text": chunk,
                    "doc_id": i,
                    "chunk_id": j,
                    **(metadata[i] if metadata else {})
                })
        
        # Batch upsert
        points = [
            PointStruct(id=i, vector=emb, payload=meta)
            for i, (emb, meta) in enumerate(zip(embeddings, all_metadata))
        ]
        
        self.qdrant.upsert(collection_name=self.collection_name, points=points)
    
    def retrieve(self, query: str, top_k: int = 5, rerank: bool = True) -> list[dict]:
        """Retrieve and optionally rerank relevant chunks."""
        # Initial retrieval
        results = similarity_search(query, top_k=top_k * 2 if rerank else top_k)
        
        # Rerank if enabled
        if rerank:
            documents = [r["text"] for r in results]
            results = rerank_cross_encoder(query, documents, top_k=top_k)
        
        return results
    
    def generate(self, query: str, top_k: int = 5) -> str:
        """RAG: retrieve context and generate answer."""
        # Retrieve relevant chunks
        context_chunks = self.retrieve(query, top_k=top_k)
        context = "\n\n".join(f"[{i+1}] {c['text']}" for i, c in enumerate(context_chunks))
        
        # Generate answer
        prompt = f"""Answer the question using only the provided context.

Context:
{context}

Question: {query}

Answer:"""

        response = self.anthropic.messages.create(
            model="claude-3-5-sonnet-20241022",
            max_tokens=1024,
            messages=[{"role": "user", "content": prompt}]
        )
        
        return response.content[0].text

# Usage
rag = RAGPipeline()
rag.index_documents(["Document 1...", "Document 2..."])
answer = rag.generate("What is RAG?")
print(answer)
```

---

## Best Practices

1. **Chunk size:** 500-1000 tokens for general text, smaller for code
2. **Overlap:** 10-20% of chunk size to preserve context
3. **Top-k retrieval:** Start with 5-10, tune based on precision/recall
4. **Reranking:** Always rerank for production (cross-encoder or LLM)
5. **Metadata:** Store source, timestamps, authors for filtering
6. **Hybrid search:** Combine vector + keyword for best results
7. **Caching:** Cache embeddings to reduce API costs
8. **Monitoring:** Track retrieval latency, relevance scores, cache hit rate

---

## Performance Optimization

```python
# Batch embedding (10x faster)
embeddings = embed_openai(chunks, batch_size=100)

# Async retrieval
import asyncio

async def async_retrieve(queries: list[str]) -> list[list[dict]]:
    tasks = [similarity_search(q) for q in queries]
    return await asyncio.gather(*tasks)

# Embedding cache
from functools import lru_cache

@lru_cache(maxsize=10000)
def cached_embed(text: str) -> list[float]:
    return embed_openai([text])[0]
```

---

## Related

- [RAG Agent Example](examples/rag-agent.md) — working implementation
- [Embeddings Guide](../ai/embeddings.md) — model comparison
- [Vector Stores](../infrastructure/vector-stores.md) — Qdrant, Pinecone, Chroma setup
