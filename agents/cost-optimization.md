---
type: guide
tags: [agents, cost-optimization, pricing, caching, batch-api, tokens]
status: complete
updated: 2026-03-25
related:
  - ../infrastructure/monitoring.md
  - error-handling.md
---

# Cost Optimization

Стратегии снижения стоимости AI-агентов: prompt caching, batch API, выбор моделей, token management.

---

## Model Pricing Comparison

### Anthropic Claude (March 2026)

| Model | Input ($/1M) | Output ($/1M) | Context | Use Case |
|-------|-------------|--------------|---------|----------|
| **Opus 4.6** | $15 | $75 | 200K | Сложные задачи, архитектура |
| **Sonnet 4.5** | $3 | $15 | 200K | Основная рабочая лошадка |
| **Haiku 4.5** | $0.25 | $1.25 | 200K | Простые задачи, классификация |

### OpenAI (March 2026)

| Model | Input ($/1M) | Output ($/1M) | Context | Use Case |
|-------|-------------|--------------|---------|----------|
| **o3** | $15 | $60 | 128K | Reasoning, сложные задачи |
| **GPT-4o** | $2.5 | $10 | 128K | Универсальная модель |
| **GPT-4o mini** | $0.15 | $0.60 | 128K | Быстрые задачи |

### Google Gemini (March 2026)

| Model | Input ($/1M) | Output ($/1M) | Context | Use Case |
|-------|-------------|--------------|---------|----------|
| **Gemini 2.5 Pro** | $1.25 | $5 | 1M | Огромный контекст |
| **Gemini 2.5 Flash** | $0.075 | $0.30 | 1M | Самая дешёвая |

### DeepSeek (March 2026)

| Model | Input ($/1M) | Output ($/1M) | Context | Use Case |
|-------|-------------|--------------|---------|----------|
| **DeepSeek V3** | $0.27 | $1.10 | 128K | Лучшая цена/качество |
| **DeepSeek R1** | $0.55 | $2.19 | 128K | Reasoning, дешевле o3 |

### Embeddings

| Model | Price ($/1M tokens) | Dimensions | Provider |
|-------|-------------------|-----------|----------|
| **text-embedding-3-small** | $0.02 | 1536 | OpenAI |
| **text-embedding-3-large** | $0.13 | 3072 | OpenAI |
| **nomic-embed-text** | Free | 768 | Ollama (local) |

---

## Cost Calculation

### Example: Simple Agent

```python
# 1000 requests per day
# Average: 500 input tokens, 200 output tokens per request

# Claude Sonnet 4.5
input_cost = (500 * 1000 / 1_000_000) * 3 = $1.50/day
output_cost = (200 * 1000 / 1_000_000) * 15 = $3.00/day
total = $4.50/day = $135/month

# Claude Haiku 4.5 (10x cheaper)
input_cost = (500 * 1000 / 1_000_000) * 0.25 = $0.125/day
output_cost = (200 * 1000 / 1_000_000) * 1.25 = $0.25/day
total = $0.375/day = $11.25/month

# Savings: $123.75/month (92% reduction)
```

### Example: RAG Agent

```python
# 1000 queries per day
# 5 documents indexed per query
# Average doc: 500 tokens

# Embeddings (text-embedding-3-small)
embedding_cost = (500 * 5 * 1000 / 1_000_000) * 0.02 = $0.05/day

# LLM (Sonnet 4.5)
# Context: 2500 tokens (5 docs), Query: 100, Output: 300
input_cost = ((2500 + 100) * 1000 / 1_000_000) * 3 = $7.80/day
output_cost = (300 * 1000 / 1_000_000) * 15 = $4.50/day

total = $12.35/day = $370.50/month
```

---

## Optimization Strategies

### 1. Prompt Caching (Anthropic)

**Savings: 90% на cached tokens**

```python
# Без кэширования
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    system="Long system prompt...",  # 5000 tokens
    messages=[{"role": "user", "content": "Question"}]
)
# Cost: 5100 input tokens * $3/1M = $0.0153

# С кэшированием
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": "Long system prompt...",  # 5000 tokens
            "cache_control": {"type": "ephemeral"}
        }
    ],
    messages=[{"role": "user", "content": "Question"}]
)
# First call: 5100 tokens * $3.75/1M = $0.019 (cache write)
# Subsequent calls: 100 tokens * $3/1M + 5000 * $0.30/1M = $0.0018
# Savings: 90% per cached request
```

**Best practices:**
- Кэшируйте system prompts (обычно не меняются)
- Кэшируйте большие контексты (документы, примеры)
- Cache TTL: 5 минут (обновляется при каждом использовании)

### 2. Batch API (Anthropic)

**Savings: 50% на всех токенах**

```python
# Создание batch запроса
batch = client.batches.create(
    requests=[
        {
            "custom_id": "req-1",
            "params": {
                "model": "claude-sonnet-4-20250514",
                "max_tokens": 1024,
                "messages": [{"role": "user", "content": "Task 1"}]
            }
        },
        {
            "custom_id": "req-2",
            "params": {
                "model": "claude-sonnet-4-20250514",
                "max_tokens": 1024,
                "messages": [{"role": "user", "content": "Task 2"}]
            }
        }
        # ... до 10,000 запросов
    ]
)

# Проверка статуса
status = client.batches.retrieve(batch.id)

# Получение результатов (когда completed)
results = client.batches.results(batch.id)
```

**Когда использовать:**
- Обработка больших датасетов
- Ночные задачи (не требуют real-time)
- Evaluation runs
- Bulk content generation

**Ограничения:**
- Результаты через 24 часа
- Не подходит для интерактивных агентов

### 3. Model Selection Strategy

```python
class SmartModelSelector:
    """Выбор модели на основе сложности задачи"""
    
    MODELS = {
        "simple": ("claude-haiku-4-20250514", 0.25, 1.25),
        "medium": ("claude-sonnet-4-20250514", 3.0, 15.0),
        "complex": ("claude-opus-4-20250514", 15.0, 75.0)
    }
    
    def select_model(self, task: str) -> tuple:
        """Выбрать модель на основе задачи"""
        # Простые задачи
        if self._is_simple(task):
            return self.MODELS["simple"]
        
        # Сложные задачи
        elif self._is_complex(task):
            return self.MODELS["complex"]
        
        # По умолчанию средняя
        else:
            return self.MODELS["medium"]
    
    def _is_simple(self, task: str) -> bool:
        """Проверка на простую задачу"""
        simple_keywords = [
            "classify", "summarize", "extract",
            "yes or no", "true or false"
        ]
        return any(kw in task.lower() for kw in simple_keywords)
    
    def _is_complex(self, task: str) -> bool:
        """Проверка на сложную задачу"""
        complex_keywords = [
            "architecture", "design system",
            "refactor", "optimize", "debug complex"
        ]
        return any(kw in task.lower() for kw in complex_keywords)

# Использование
selector = SmartModelSelector()
model, input_price, output_price = selector.select_model(
    "Classify this email as spam or not spam"
)
# Returns: Haiku (10x cheaper than Sonnet)
```

### 4. Token Management

```python
import tiktoken

def count_tokens(text: str, model: str = "claude-3") -> int:
    """Подсчёт токенов"""
    # Для Claude используем cl100k_base (приблизительно)
    encoding = tiktoken.get_encoding("cl100k_base")
    return len(encoding.encode(text))

def truncate_to_budget(text: str, max_tokens: int) -> str:
    """Обрезать текст до бюджета токенов"""
    encoding = tiktoken.get_encoding("cl100k_base")
    tokens = encoding.encode(text)
    
    if len(tokens) <= max_tokens:
        return text
    
    # Обрезаем и декодируем
    truncated = tokens[:max_tokens]
    return encoding.decode(truncated)

# Использование
context = "Very long document..."
if count_tokens(context) > 10000:
    context = truncate_to_budget(context, 10000)
```

### 5. Streaming vs Non-Streaming

```python
# Non-streaming (дешевле для коротких ответов)
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=100,  # Короткий ответ
    messages=[{"role": "user", "content": "Yes or no?"}]
)

# Streaming (лучше UX, но та же цена)
with client.messages.stream(
    model="claude-sonnet-4-20250514",
    max_tokens=2000,  # Длинный ответ
    messages=[{"role": "user", "content": "Explain..."}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

**Вывод:** Streaming не влияет на стоимость, но улучшает UX для длинных ответов.

---

## Cost Monitoring

```python
# cost_tracker.py
from dataclasses import dataclass
from datetime import datetime

@dataclass
class CostRecord:
    timestamp: datetime
    model: str
    input_tokens: int
    output_tokens: int
    cost_usd: float
    agent_name: str

class CostTracker:
    """Трекинг стоимости агентов"""
    
    def __init__(self):
        self.records = []
        self.daily_budget = 100.0  # $100/day
        self.daily_spent = 0.0
    
    def record_request(
        self,
        model: str,
        input_tokens: int,
        output_tokens: int,
        agent_name: str
    ):
        """Записать запрос"""
        cost = calculate_cost(model, input_tokens, output_tokens)
        
        record = CostRecord(
            timestamp=datetime.now(),
            model=model,
            input_tokens=input_tokens,
            output_tokens=output_tokens,
            cost_usd=cost,
            agent_name=agent_name
        )
        
        self.records.append(record)
        self.daily_spent += cost
        
        # Проверка бюджета
        if self.daily_spent > self.daily_budget:
            self._alert_budget_exceeded()
    
    def get_daily_report(self) -> dict:
        """Дневной отчёт"""
        today = datetime.now().date()
        today_records = [
            r for r in self.records
            if r.timestamp.date() == today
        ]
        
        return {
            "total_cost": sum(r.cost_usd for r in today_records),
            "total_requests": len(today_records),
            "by_model": self._group_by_model(today_records),
            "by_agent": self._group_by_agent(today_records)
        }
```

---

## Best Practices Summary

| Strategy | Savings | Effort | When to Use |
|----------|---------|--------|-------------|
| **Use Haiku instead of Sonnet** | 90% | Low | Simple tasks |
| **Prompt Caching** | 90% on cached | Medium | Repeated contexts |
| **Batch API** | 50% | Medium | Non-real-time |
| **DeepSeek V3** | 90% vs Sonnet | Low | Cost-sensitive |
| **Local embeddings (Ollama)** | 100% | High | Privacy + cost |
| **Token truncation** | 20-50% | Low | Long contexts |
| **Smart model selection** | 50-80% | Medium | Mixed workload |

---

## Cost Optimization Checklist

- [ ] Используйте Haiku для простых задач (классификация, извлечение)
- [ ] Включите prompt caching для system prompts
- [ ] Используйте Batch API для bulk операций
- [ ] Рассмотрите DeepSeek V3 для cost-sensitive задач
- [ ] Truncate длинные контексты до необходимого минимума
- [ ] Мониторьте стоимость через Prometheus metrics
- [ ] Установите daily budget alerts
- [ ] Используйте local embeddings (Ollama) где возможно

---

## See Also

- [Monitoring](../infrastructure/monitoring.md) — cost tracking metrics
- [Error Handling](error-handling.md) — fallback между моделями
- [Anthropic Pricing](https://www.anthropic.com/pricing) — актуальные цены
