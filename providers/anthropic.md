---
type: reference
tags: [providers, anthropic, api, claude, batch, rate-limits, direct-access, pricing]
status: complete
updated: 2026-03-24
related:
  - providers/providers.md
  - providers/openrouter.md
  - ai/anthropic/anthropic.md
  - agents/tool-use.md
---

# Anthropic API

## Overview

- **Тип:** Прямой провайдер
- **Сайт:** anthropic.com
- **API Base URL:** `https://api.anthropic.com`
- **Приоритет в базе:** Прямой доступ для Batch API, fine-tuning, максимальных rate limits

Anthropic API — официальный прямой доступ к моделям Claude. Используется когда нужен Batch API (50% скидка), специфические фичи (Extended Thinking, prompt caching) или enterprise rate limits.

---

## Текущие модели

| Модель | API ID | Контекст | Max output | Input / Output |
|--------|--------|---------|-----------|---------------|
| Claude Opus 4.6 | `claude-opus-4-6` | 1M | 128k | $5 / $25 MTok |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 1M | 64k | $3 / $15 MTok |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | 200k | 64k | $1 / $5 MTok |

Все текущие модели поддерживают: vision, tool use, Extended Thinking, Priority Tier.

Полный список моделей: [platform.claude.com/docs/en/docs/about-claude/models/overview](https://platform.claude.com/docs/en/docs/about-claude/models/overview)

---

## Подключение

### Python SDK

```python
import anthropic

client = anthropic.Anthropic(api_key="sk-ant-...")

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}]
)
print(response.content[0].text)
```

### Environment переменная

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

SDK автоматически читает `ANTHROPIC_API_KEY`.

---

## Message Batches API

### Что это

Асинхронная обработка больших объёмов запросов. **50% скидка** на все модели. Большинство батчей завершается < 1 часа.

### Лимиты

| Параметр | Значение |
|---------|---------|
| Максимум запросов в батче | 100,000 |
| Максимальный размер батча | 256 MB |
| Время обработки (обычно) | < 1 часа |
| Максимальное время обработки | 24 часа |
| Хранение результатов | 29 дней |

### Пример — создание батча

```python
import anthropic

client = anthropic.Anthropic()

batch = client.messages.batches.create(
    requests=[
        {
            "custom_id": "request-1",
            "params": {
                "model": "claude-sonnet-4-6",
                "max_tokens": 1024,
                "messages": [{"role": "user", "content": "Summarize: ..."}]
            }
        },
        {
            "custom_id": "request-2",
            "params": {
                "model": "claude-sonnet-4-6",
                "max_tokens": 1024,
                "messages": [{"role": "user", "content": "Translate: ..."}]
            }
        }
    ]
)
print(batch.id)  # msgbatch_...
```

### Пример — получение результатов

```python
# Проверить статус
batch = client.messages.batches.retrieve(batch_id)
print(batch.processing_status)  # "in_progress" / "ended"

# Скачать результаты (когда ended)
for result in client.messages.batches.results(batch_id):
    if result.result.type == "succeeded":
        print(result.custom_id, result.result.message.content[0].text)
```

### Цены с батч-скидкой

| Модель | Input | Output |
|--------|-------|--------|
| Opus 4.6 | $2.50 / MTok | $12.50 / MTok |
| Sonnet 4.6 | $1.50 / MTok | $7.50 / MTok |
| Haiku 4.5 | $0.50 / MTok | $2.50 / MTok |

---

## Prompt Caching

Кэширование префиксов промптов для снижения стоимости при повторных вызовах с общим контекстом.

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": "<long system prompt>",
            "cache_control": {"type": "ephemeral"}  # кэшировать этот блок
        }
    ],
    messages=[{"role": "user", "content": "Question about the system prompt"}]
)
```

- Кэш живёт 5 минут (стандарт) или 1 час (бета-заголовок)
- Кэшированные токены стоят ~10% от обычной цены
- Подходит для: длинных system prompts, документов, историй разговоров

---

## Extended Thinking

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000  # сколько токенов на рассуждение
    },
    messages=[{"role": "user", "content": "Solve this complex problem..."}]
)

# В ответе — блоки thinking + text
for block in response.content:
    if block.type == "thinking":
        print("Reasoning:", block.thinking)
    elif block.type == "text":
        print("Answer:", block.text)
```

---

## Когда использовать напрямую (vs OpenRouter)

| Сценарий | Рекомендация |
|---------|-------------|
| Batch API (50% скидка) | Anthropic напрямую — недоступен через OpenRouter |
| Fine-tuning | Anthropic напрямую |
| Prompt caching | Anthropic напрямую (OpenRouter не поддерживает) |
| Extended Thinking | Anthropic напрямую (полная поддержка) |
| Enterprise rate limits | Anthropic напрямую |
| Обычные запросы к Claude | OpenRouter (проще, единый ключ) |
| Мульти-модельный pipeline | OpenRouter |

---

## Rate Limits

Лимиты зависят от уровня (Tier 1–5) и модели. Растут с потреблением.

- **Tier 1** (новые аккаунты): ~50 RPM, ~200K TPM
- **Tier 5** (enterprise): сотни RPM, миллионы TPM

Проверить текущие лимиты: [console.anthropic.com → Rate limits](https://console.anthropic.com)

---

## Доступность через облачные платформы

Claude также доступен через:

| Платформа | ID префикс |
|---------|-----------|
| AWS Bedrock | `anthropic.claude-sonnet-4-6` |
| Google Vertex AI | `claude-sonnet-4-6` |

Через облачные платформы — другая система биллинга, но те же модели.

---

## See Also

- [OpenRouter](openrouter.md) — единый API для всех моделей, проще для большинства задач
- [AI Models](../ai/ai.md) — сравнение Claude с другими моделями
- [Planning](../agents/planning.md) — Extended Thinking в контексте агентного планирования
