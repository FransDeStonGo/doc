---
type: guide
tags: [agents, examples, rag, vector-search, qdrant, embeddings, python, intermediate]
status: complete
updated: 2026-03-25
related:
  - ../memory.md
  - ../tool-use.md
  - ../../ai/openai/openai.md
---

# RAG Agent

Агент с external memory — использует Qdrant для векторного поиска и OpenAI embeddings для семантического понимания документов.

---

## Overview

Этот пример показывает:
- Как индексировать документы в векторную БД
- Как выполнять семантический поиск
- Как использовать найденный контекст для ответа

**Сложность:** ⭐⭐ Средний  
**Строк кода:** ~200  
**Компоненты:** Qdrant, OpenRouter (OpenAI embeddings), Anthropic Claude

---

## Architecture

```
User Query
    ↓
1. Generate embedding (OpenRouter: text-embedding-3-small)
    ↓
2. Search Qdrant (semantic similarity)
    ↓
3. Retrieve top-k documents
    ↓
4. Build context prompt
    ↓
5. Claude generates answer with context
    ↓
Response
```

---

## Code

```python
#!/usr/bin/env python3
"""
RAG Agent — агент с external memory через Qdrant + OpenAI embeddings.
Индексирует документы, выполняет семантический поиск, отвечает на основе контекста.
"""

import os
import json
from typing import List, Dict
from anthropic import Anthropic
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct
import requests
from dotenv import load_dotenv

load_dotenv()

# Клиенты
anthropic_client = Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))
qdrant_client = QdrantClient(host="localhost", port=6333)
openrouter_api_key = os.environ.get("OPENROUTER_API_KEY")

COLLECTION_NAME = "knowledge_base"
EMBEDDING_MODEL = "openai/text-embedding-3-small"
EMBEDDING_DIM = 1536  # text-embedding-3-small dimension

# Инициализация коллекции
def init_collection():
    """Создать коллекцию в Qdrant если не существует"""
    collections = qdrant_client.get_collections().collections
    if not any(c.name == COLLECTION_NAME for c in collections):
        qdrant_client.create_collection(
            collection_name=COLLECTION_NAME,
            vectors_config=VectorParams(size=EMBEDDING_DIM, distance=Distance.COSINE)
        )
        print(f"✅ Collection '{COLLECTION_NAME}' created")
    else:
        print(f"✅ Collection '{COLLECTION_NAME}' already exists")

# Генерация embeddings через OpenRouter
def get_embedding(text: str) -> List[float]:
    """Получить embedding через OpenRouter (OpenAI text-embedding-3-small)"""
    response = requests.post(
        "https://openrouter.ai/api/v1/embeddings",
        headers={
            "Authorization": f"Bearer {openrouter_api_key}",
            "Content-Type": "application/json"
        },
        json={
            "model": EMBEDDING_MODEL,
            "input": text
        }
    )
    
    if response.status_code != 200:
        raise Exception(f"OpenRouter API error: {response.text}")
    
    return response.json()["data"][0]["embedding"]

# Индексация документов
def index_documents(documents: List[Dict[str, str]]):
    """
    Индексировать документы в Qdrant.
    
    Args:
        documents: List of {"id": str, "text": str, "metadata": dict}
    """
    print(f"\n📚 Indexing {len(documents)} documents...")
    
    points = []
    for doc in documents:
        print(f"   Processing: {doc['id']}")
        
        # Генерируем embedding
        embedding = get_embedding(doc["text"])
        
        # Создаём точку для Qdrant
        point = PointStruct(
            id=hash(doc["id"]) % (10 ** 8),  # Простой ID из хеша
            vector=embedding,
            payload={
                "doc_id": doc["id"],
                "text": doc["text"],
                **doc.get("metadata", {})
            }
        )
        points.append(point)
    
    # Загружаем в Qdrant
    qdrant_client.upsert(
        collection_name=COLLECTION_NAME,
        points=points
    )
    
    print(f"✅ Indexed {len(documents)} documents\n")

# Семантический поиск
def search_documents(query: str, top_k: int = 3) -> List[Dict]:
    """
    Семантический поиск по документам.
    
    Args:
        query: Поисковый запрос
        top_k: Количество результатов
    
    Returns:
        List of {"doc_id": str, "text": str, "score": float, "metadata": dict}
    """
    # Генерируем embedding для запроса
    query_embedding = get_embedding(query)
    
    # Поиск в Qdrant
    results = qdrant_client.search(
        collection_name=COLLECTION_NAME,
        query_vector=query_embedding,
        limit=top_k
    )
    
    # Форматируем результаты
    documents = []
    for result in results:
        documents.append({
            "doc_id": result.payload["doc_id"],
            "text": result.payload["text"],
            "score": result.score,
            "metadata": {k: v for k, v in result.payload.items() 
                        if k not in ["doc_id", "text"]}
        })
    
    return documents

# RAG Agent
def rag_agent(user_query: str, top_k: int = 3) -> str:
    """
    RAG Agent — поиск контекста + генерация ответа.
    
    Args:
        user_query: Вопрос пользователя
        top_k: Количество документов для контекста
    
    Returns:
        Ответ агента
    """
    print(f"\n🧑 User: {user_query}\n")
    
    # 1. Семантический поиск
    print(f"🔍 Searching knowledge base (top {top_k})...")
    documents = search_documents(user_query, top_k=top_k)
    
    if not documents:
        return "Sorry, I couldn't find relevant information in the knowledge base."
    
    # Выводим найденные документы
    print(f"📄 Found {len(documents)} relevant documents:\n")
    for i, doc in enumerate(documents, 1):
        print(f"   {i}. {doc['doc_id']} (score: {doc['score']:.3f})")
        print(f"      {doc['text'][:100]}...\n")
    
    # 2. Формируем контекст
    context = "\n\n".join([
        f"Document {i}: {doc['text']}"
        for i, doc in enumerate(documents, 1)
    ])
    
    # 3. Генерируем ответ с контекстом
    system_prompt = """You are a helpful assistant with access to a knowledge base.
Answer the user's question based on the provided context documents.
If the context doesn't contain relevant information, say so clearly.
Always cite which document(s) you used (e.g., "According to Document 1...")."""

    user_prompt = f"""Context from knowledge base:

{context}

---

User question: {user_query}

Please answer based on the context above."""

    print("🤖 Generating answer with context...\n")
    
    response = anthropic_client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        system=system_prompt,
        messages=[{"role": "user", "content": user_prompt}]
    )
    
    answer = response.content[0].text
    
    print(f"💬 Agent: {answer}\n")
    
    return answer

# Пример использования
if __name__ == "__main__":
    # 1. Инициализация
    init_collection()
    
    # 2. Подготовка документов для индексации
    sample_documents = [
        {
            "id": "doc_1",
            "text": "Anthropic Claude is a family of AI models including Opus, Sonnet, and Haiku. Claude Sonnet 4 is the latest version released in 2025, offering improved reasoning and tool use capabilities.",
            "metadata": {"category": "AI Models", "source": "anthropic_docs"}
        },
        {
            "id": "doc_2",
            "text": "RAG (Retrieval-Augmented Generation) is a technique that combines vector search with LLM generation. It allows agents to access external knowledge bases and provide more accurate, grounded responses.",
            "metadata": {"category": "AI Techniques", "source": "rag_guide"}
        },
        {
            "id": "doc_3",
            "text": "Qdrant is an open-source vector database optimized for similarity search. It supports HNSW indexing, filtering, and can handle millions of vectors efficiently.",
            "metadata": {"category": "Databases", "source": "qdrant_docs"}
        },
        {
            "id": "doc_4",
            "text": "OpenAI's text-embedding-3-small is a cost-effective embedding model at $0.02 per 1M tokens. It produces 1536-dimensional vectors suitable for semantic search applications.",
            "metadata": {"category": "Embeddings", "source": "openai_pricing"}
        },
        {
            "id": "doc_5",
            "text": "Tool use (function calling) allows AI agents to interact with external systems. Anthropic Claude supports parallel tool calls and structured outputs for reliable integrations.",
            "metadata": {"category": "AI Capabilities", "source": "tool_use_guide"}
        }
    ]
    
    # 3. Индексация (запускать один раз)
    index_documents(sample_documents)
    
    # 4. Примеры запросов
    print("=" * 60)
    print("RAG Agent Examples")
    print("=" * 60)
    
    # Пример 1: Вопрос про Claude
    rag_agent("What is Claude Sonnet 4?")
    
    print("\n" + "=" * 60 + "\n")
    
    # Пример 2: Вопрос про RAG
    rag_agent("How does RAG work?")
    
    print("\n" + "=" * 60 + "\n")
    
    # Пример 3: Вопрос про embeddings
    rag_agent("What's the cheapest embedding model?")
```

---

## Prerequisites

```bash
# Установка зависимостей
pip install anthropic qdrant-client requests python-dotenv

# Запуск Qdrant (Docker)
docker run -p 6333:6333 -p 6334:6334 \
    -v $(pwd)/qdrant_storage:/qdrant/storage:z \
    qdrant/qdrant
```

---

## Environment Variables

```bash
# .env
ANTHROPIC_API_KEY=sk-ant-...
OPENROUTER_API_KEY=sk-or-v1-...
```

---

## Running

```bash
# 1. Запустите Qdrant
docker run -d -p 6333:6333 qdrant/qdrant

# 2. Создайте .env с ключами
echo "ANTHROPIC_API_KEY=sk-ant-..." > .env
echo "OPENROUTER_API_KEY=sk-or-v1-..." >> .env

# 3. Запустите агента
python rag-agent.py
```

**Ожидаемый вывод:**
```
✅ Collection 'knowledge_base' created

📚 Indexing 5 documents...
   Processing: doc_1
   Processing: doc_2
   Processing: doc_3
   Processing: doc_4
   Processing: doc_5
✅ Indexed 5 documents

============================================================
RAG Agent Examples
============================================================

🧑 User: What is Claude Sonnet 4?

🔍 Searching knowledge base (top 3)...
📄 Found 3 relevant documents:

   1. doc_1 (score: 0.892)
      Anthropic Claude is a family of AI models including Opus, Sonnet, and Haiku...

🤖 Generating answer with context...

💬 Agent: According to Document 1, Claude Sonnet 4 is the latest version 
of Anthropic's Claude AI model family, released in 2025. It offers improved 
reasoning and tool use capabilities compared to previous versions.
```

---

## Cost Estimation

**Embeddings (OpenRouter text-embedding-3-small):**
- $0.02 per 1M tokens
- ~100 tokens per document
- 1000 documents = $0.002

**Claude Sonnet 4:**
- $3 per 1M input tokens
- $15 per 1M output tokens
- Typical query: ~500 input + 200 output = $0.0045

**Total for 100 queries with 1000 docs:** ~$0.45

---

## Extending

### 1. Chunking для длинных документов

```python
def chunk_text(text: str, chunk_size: int = 500, overlap: int = 50) -> List[str]:
    """Разбить текст на чанки с перекрытием"""
    words = text.split()
    chunks = []
    
    for i in range(0, len(words), chunk_size - overlap):
        chunk = " ".join(words[i:i + chunk_size])
        chunks.append(chunk)
    
    return chunks

# Использование
for doc in documents:
    chunks = chunk_text(doc["text"])
    for i, chunk in enumerate(chunks):
        index_documents([{
            "id": f"{doc['id']}_chunk_{i}",
            "text": chunk,
            "metadata": {**doc["metadata"], "chunk_index": i}
        }])
```

### 2. Reranking для улучшения точности

```python
def rerank_documents(query: str, documents: List[Dict], top_k: int = 3) -> List[Dict]:
    """Rerank документов через Claude (LLM-as-judge)"""
    rerank_prompt = f"""Query: {query}

Rank these documents by relevance (1 = most relevant):

{chr(10).join([f"{i+1}. {doc['text'][:200]}" for i, doc in enumerate(documents)])}

Return only numbers in order, e.g.: 2,1,3"""
    
    response = anthropic_client.messages.create(
        model="claude-haiku-4-20250514",  # Дешёвая модель для reranking
        max_tokens=50,
        messages=[{"role": "user", "content": rerank_prompt}]
    )
    
    # Парсим порядок
    order = [int(x.strip()) - 1 for x in response.content[0].text.split(",")]
    return [documents[i] for i in order[:top_k]]
```

### 3. Hybrid search (vector + keyword)

```python
from qdrant_client.models import Filter, FieldCondition, MatchValue

def hybrid_search(query: str, category: str = None, top_k: int = 3):
    """Комбинация векторного поиска и фильтрации"""
    query_embedding = get_embedding(query)
    
    # Фильтр по категории
    query_filter = None
    if category:
        query_filter = Filter(
            must=[FieldCondition(key="category", match=MatchValue(value=category))]
        )
    
    results = qdrant_client.search(
        collection_name=COLLECTION_NAME,
        query_vector=query_embedding,
        query_filter=query_filter,
        limit=top_k
    )
    
    return results
```

---

## Best Practices

1. **Chunking strategy** — 300-500 слов с 10-20% overlap
2. **Metadata** — добавляйте source, timestamp, category для фильтрации
3. **Cache embeddings** — не генерируйте повторно для одинаковых текстов
4. **Batch indexing** — индексируйте пачками по 100-500 документов
5. **Monitor costs** — логируйте количество embedding calls

---

## Troubleshooting

**Qdrant connection error:**
```bash
# Проверьте что Qdrant запущен
curl http://localhost:6333/collections
```

**OpenRouter 429 (rate limit):**
- Добавьте retry с exponential backoff
- Используйте batch embeddings API

**Low relevance scores:**
- Проверьте качество документов (слишком короткие/длинные?)
- Попробуйте другую embedding модель (text-embedding-3-large)
- Добавьте reranking

---

## See Also

- [Memory](../memory.md) — типы памяти агентов
- [Tool Use](../tool-use.md) — как добавить search_documents как tool
- [OpenAI Embeddings](../../ai/openai/openai.md) — документация по embedding моделям
