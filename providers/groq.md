---
type: reference
tags: [providers, groq, lpu, low-latency, llama, mixtral, inference, openai-compatible]
status: complete
updated: 2026-03-24
related:
  - providers/providers.md
  - providers/openrouter.md
  - ai/meta/meta.md
---

# Groq

## Overview

- **Компания:** Groq Inc. (не путать с xAI Grok)
- **Технология:** LPU (Language Processing Unit) — собственный чип для inference
- **Ключевая особенность:** скорость генерации в 5-10x быстрее GPU-провайдеров при той же цене

Groq — специализированный inference-провайдер с собственным чипом LPU. Не обучает модели, только хостит чужие (Llama, Mixtral, Gemma). Уникальная позиция: максимальная скорость токен-генерации для streaming-агентов и real-time пайплайнов.

## Specifications

| Параметр | Значение |
|----------|---------|
| Скорость | 500-2000 токенов/сек (зависит от модели) |
| API совместимость | OpenAI-совместимый формат |
| Контекст | 8k-128k (зависит от модели) |
| Rate limits | 30 req/min (free), выше на платных планах |
| Модели | чужие open-source (Meta, Mistral, Google) |

## Models (актуальные)

| Модель | Контекст | Скорость | Цена (input/output за 1M) |
|--------|----------|----------|---------------------------|
| llama-3.3-70b-versatile | 128k | ~800 tok/s | $0.59 / $0.79 |
| llama-3.1-8b-instant | 128k | ~1200 tok/s | $0.05 / $0.08 |
| mixtral-8x7b-32768 | 32k | ~600 tok/s | $0.24 / $0.24 |
| gemma2-9b-it | 8k | ~1000 tok/s | $0.2 / $0.2 |

## Agent Use Cases

- **Real-time streaming агент** — пользователь видит мгновенный отклик благодаря высокой скорости
- **High-throughput pipeline** — обработка большого числа параллельных запросов без роста latency
- **Time-sensitive задачи** — агент для торговли, мониторинга, алертов где важна скорость
- **Быстрые подзадачи** — llama-3.1-8b-instant как дешёвый и быстрый worker в multi-agent системе
- **Interactive coding** — мгновенные ответы при code generation/completion

## Considerations

- Ограниченный набор моделей — только популярные open-source
- Rate limits на бесплатном плане строгие
- Нет fine-tuning и кастомных моделей
- Контекстные окна некоторых моделей меньше чем у оригинального провайдера

## Setup

```bash
pip install groq
```

```python
from groq import Groq

client = Groq(api_key="gsk_...")

response = client.chat.completions.create(
    model="llama-3.3-70b-versatile",
    messages=[{"role": "user", "content": "Hello"}]
)
print(response.choices[0].message.content)
```

```python
# Через OpenAI SDK (совместимый API)
from openai import OpenAI

client = OpenAI(
    base_url="https://api.groq.com/openai/v1",
    api_key="gsk_..."
)

# Streaming
stream = client.chat.completions.create(
    model="llama-3.3-70b-versatile",
    messages=[{"role": "user", "content": "Tell me a story"}],
    stream=True
)
for chunk in stream:
    print(chunk.choices[0].delta.content or "", end="")
```

## See Also

- [OpenRouter (агрегатор с Groq как провайдером)](openrouter.md)
- [Meta Llama (модели, которые хостит Groq)](../ai/meta/meta.md)
- [Providers — обзор](providers.md)
