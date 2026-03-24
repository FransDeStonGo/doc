---
type: reference
tags: [providers, openrouter, routing, fallback, api, openai-compatible, models, cost]
status: complete
updated: 2026-03-24
related:
  - providers/providers.md
  - providers/anthropic.md
  - providers/openai.md
  - ai/ai.md
---

# OpenRouter

## Overview

- **Тип:** Агрегатор / роутер моделей
- **Сайт:** openrouter.ai
- **API:** Совместим с OpenAI SDK (drop-in замена)
- **Приоритет в базе:** Основной провайдер — единая точка доступа ко всем моделям

OpenRouter — единый API для сотен AI-моделей от разных провайдеров. Вместо поддержки 5+ разных SDK, интегрируешь один OpenRouter и получаешь доступ к Claude, GPT, Gemini, DeepSeek, Llama и сотням других через одинаковый интерфейс.

---

## Specifications

| Параметр | Значение |
|---------|---------|
| Base URL | `https://openrouter.ai/api/v1` |
| API совместимость | OpenAI SDK (полная) |
| Число моделей | 300+ |
| Ценообразование | Pay-per-token, нет подписки |
| Минимальный депозит | $5 |
| Бесплатные модели | Да (`:free` суффикс) |

---

## Подключение

### Python (OpenAI SDK)

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key="sk-or-...",  # OpenRouter API ключ
)

response = client.chat.completions.create(
    model="anthropic/claude-sonnet-4-6",
    messages=[{"role": "user", "content": "Hello"}]
)
```

### Environment переменные

```bash
export OPENROUTER_API_KEY="sk-or-..."
# Или если используешь как замену OpenAI:
export OPENAI_API_KEY="sk-or-..."
export OPENAI_BASE_URL="https://openrouter.ai/api/v1"
```

---

## Именование моделей

Формат: `<провайдер>/<модель-название>`

| Модель | OpenRouter ID |
|--------|-------------|
| Claude Sonnet 4.6 | `anthropic/claude-sonnet-4-6` |
| Claude Opus 4.6 | `anthropic/claude-opus-4-6` |
| GPT-4o | `openai/gpt-4o` |
| o3 | `openai/o3` |
| Gemini 2.5 Pro | `google/gemini-2.5-pro` |
| DeepSeek V3 | `deepseek/deepseek-chat` |
| DeepSeek R1 | `deepseek/deepseek-r1` |
| Llama 3.3 70B | `meta-llama/llama-3.3-70b-instruct` |

**Суффиксы моделей:**
- `:free` — бесплатный тир (лимиты)
- `:extended` — расширенный контекст
- `:thinking` — reasoning модели с явным мышлением
- `:online` — с доступом к веб-поиску

---

## Маршрутизация и Fallback

### Provider Routing

OpenRouter автоматически выбирает провайдера для каждого запроса. Можно настроить приоритеты:

```python
response = client.chat.completions.create(
    model="anthropic/claude-sonnet-4-6",
    messages=[...],
    extra_body={
        "provider": {
            "order": ["Anthropic", "AWS Bedrock"],  # приоритет провайдеров
            "allow_fallbacks": True
        }
    }
)
```

### Model Fallbacks

Если основная модель недоступна — автоматический переход на альтернативу:

```python
extra_body={
    "models": [
        "anthropic/claude-sonnet-4-6",
        "anthropic/claude-haiku-4-5",  # fallback
        "openai/gpt-4o"                # второй fallback
    ]
}
```

### Auto-Router

OpenRouter оптимизирует выбор провайдера для tool-calling на основе throughput и success rate — автоматически.

---

## Agent Use Cases

### Мульти-модельный pipeline

```python
# Дешёвый классификатор
classifier = client.chat.completions.create(
    model="meta-llama/llama-3.3-70b-instruct",
    messages=[{"role": "user", "content": classify_prompt}]
)

# Мощный исполнитель
executor = client.chat.completions.create(
    model="anthropic/claude-sonnet-4-6",
    messages=[{"role": "user", "content": execute_prompt}]
)
```

### Оркестратор + воркеры с разными моделями

Через один OpenRouter client можно запускать оркестратора на Sonnet и воркеров на Haiku — просто меняя параметр `model`.

### Monitoring + Observability

OpenRouter поддерживает интеграцию с Langfuse, Datadog, Braintrust через `Broadcast` — логировать все запросы агента в один инструмент.

---

## Особенности и ограничения

**Плюсы:**
- Единый API и ключ для всех моделей
- Автоматический fallback при недоступности провайдера
- Мониторинг стоимости в реальном времени
- Бесплатные модели для тестирования
- Structured outputs через JSON Schema

**Минусы / ограничения:**
- Небольшая дополнительная latency (~50-100ms) из-за проксирования
- Некоторые возможности моделей могут быть недоступны через роутер
- Batching API (Anthropic) недоступен через OpenRouter
- Fine-tuning недоступен

**Когда использовать напрямую (Anthropic/OpenAI):**
- Нужен Batch API (50% скидка, асинхронные задачи)
- Fine-tuning
- Максимальные rate limits (enterprise план)
- Специфические фичи провайдера

---

## Стоимость

Цена = цена провайдера + небольшая наценка OpenRouter (~5-10%). Для большинства задач несущественно.

Полный прайс: [openrouter.ai/models](https://openrouter.ai/models)

---

## See Also

- [Providers навигация](providers.md) — обзор всех провайдеров
- [Anthropic API](anthropic.md) — прямой доступ для batching и fine-tuning
- [AI Models](../ai/ai.md) — сравнение моделей
