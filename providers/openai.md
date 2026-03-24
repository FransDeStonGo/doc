---
type: reference
tags: [providers, openai, api, gpt, o-series, structured-outputs, assistants, pricing]
status: complete
updated: 2026-03-24
related:
  - providers/providers.md
  - providers/openrouter.md
  - ai/openai/openai.md
  - agents/tool-use.md
---

# OpenAI API

## Overview

- **Тип:** Прямой провайдер
- **Сайт:** platform.openai.com
- **API Base URL:** `https://api.openai.com/v1`
- **SDK:** `openai` (Python / Node.js)

OpenAI — провайдер GPT-4o, o3, o4-mini и других моделей. API де-факто стандарт индустрии: большинство альтернативных провайдеров (OpenRouter, Groq, Together AI) совместимы с ним.

---

## Текущие модели

### Флагманские (GPT серия)

| Модель | API ID | Контекст | Input / Output |
|--------|--------|---------|---------------|
| GPT-4o | `gpt-4o` | 128k | $2.50 / $10 MTok |
| GPT-4o mini | `gpt-4o-mini` | 128k | $0.15 / $0.60 MTok |
| GPT-4.1 | `gpt-4.1` | 1M | $2 / $8 MTok |
| GPT-4.1 mini | `gpt-4.1-mini` | 1M | $0.40 / $1.60 MTok |
| GPT-4.1 nano | `gpt-4.1-nano` | 1M | $0.10 / $0.40 MTok |

### Reasoning (o-серия)

| Модель | API ID | Контекст | Особенность |
|--------|--------|---------|------------|
| o3 | `o3` | 200k | Лучший reasoning, медленный |
| o4-mini | `o4-mini` | 200k | Быстрый reasoning, дешевле |
| o3-mini | `o3-mini` | 200k | Лёгкий вариант o3 |

Модели o-серии: думают перед ответом (chain-of-thought внутри), лучше в математике, коде, логике.

---

## Подключение

### Python SDK

```python
from openai import OpenAI

client = OpenAI(api_key="sk-...")

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}]
)
print(response.choices[0].message.content)
```

### Environment переменная

```bash
export OPENAI_API_KEY="sk-..."
```

---

## Structured Outputs

Гарантированный JSON-ответ строго по заданной схеме:

```python
from pydantic import BaseModel
from openai import OpenAI

class CalendarEvent(BaseModel):
    name: str
    date: str
    participants: list[str]

client = OpenAI()
completion = client.beta.chat.completions.parse(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "Extract event info"},
        {"role": "user", "content": "Alice and Bob meet on Friday"}
    ],
    response_format=CalendarEvent,
)
event = completion.choices[0].message.parsed
print(event.name, event.date)
```

Используется для: извлечения данных, классификации, pipeline'ов где нужен надёжный парсинг.

---

## Tool Use (Function Calling)

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get current weather for a city",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {"type": "string", "description": "City name"}
                },
                "required": ["city"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "What's the weather in Paris?"}],
    tools=tools,
    tool_choice="auto"
)

# Проверить tool call
if response.choices[0].message.tool_calls:
    tool_call = response.choices[0].message.tool_calls[0]
    print(tool_call.function.name)   # "get_weather"
    print(tool_call.function.arguments)  # '{"city": "Paris"}'
```

---

## Batch API

Аналог Anthropic Batch API — 50% скидка, асинхронная обработка:

```python
# Создать .jsonl файл с запросами
import json

requests = [
    {
        "custom_id": f"req-{i}",
        "method": "POST",
        "url": "/v1/chat/completions",
        "body": {
            "model": "gpt-4o-mini",
            "messages": [{"role": "user", "content": f"Summarize item {i}"}]
        }
    }
    for i in range(100)
]

with open("batch_requests.jsonl", "w") as f:
    for req in requests:
        f.write(json.dumps(req) + "\n")

# Загрузить файл
batch_file = client.files.create(
    file=open("batch_requests.jsonl", "rb"),
    purpose="batch"
)

# Создать батч
batch = client.batches.create(
    input_file_id=batch_file.id,
    endpoint="/v1/chat/completions",
    completion_window="24h"
)
```

---

## Assistants API

Высокоуровневый API для агентов с состоянием: threads, runs, встроенные инструменты.

```python
# Создать ассистента
assistant = client.beta.assistants.create(
    name="Code Helper",
    instructions="You are an expert programmer",
    model="gpt-4o",
    tools=[{"type": "code_interpreter"}]
)

# Создать thread (контекст разговора)
thread = client.beta.threads.create()

# Добавить сообщение
client.beta.threads.messages.create(
    thread_id=thread.id,
    role="user",
    content="Write a Python function to sort a list"
)

# Запустить
run = client.beta.threads.runs.create_and_poll(
    thread_id=thread.id,
    assistant_id=assistant.id
)

# Получить результат
if run.status == "completed":
    messages = client.beta.threads.messages.list(thread_id=thread.id)
    print(messages.data[0].content[0].text.value)
```

**Встроенные инструменты Assistants API:**
- `code_interpreter` — выполнение Python в sandbox
- `file_search` — поиск по загруженным документам (RAG)
- `function` — кастомные функции (как обычный tool use)

---

## Когда использовать OpenAI напрямую (vs OpenRouter)

| Сценарий | Рекомендация |
|---------|-------------|
| Только GPT/o-серия модели | OpenAI или OpenRouter — одинаково |
| Assistants API | OpenAI напрямую (OpenRouter не поддерживает) |
| Fine-tuning | OpenAI напрямую |
| Batch API (50% скидка) | OpenAI напрямую |
| Structured Outputs `.parse()` | OpenAI напрямую (SDK-фича) |
| Мульти-модельный pipeline | OpenRouter |
| Нужна fallback на Claude | OpenRouter |

---

## Совместимые провайдеры

OpenAI-совместимый API поддерживают:

| Провайдер | Специализация |
|---------|--------------|
| OpenRouter | Агрегатор 300+ моделей |
| Groq | Очень быстрый inference (Llama, Gemma) |
| Together AI | Open-source модели |
| Fireworks AI | Быстрый inference, fine-tuning |
| Anyscale | Scalable inference |

Замена: просто меняй `base_url` и `api_key`, остальной код не меняется.

---

## See Also

- [OpenRouter](openrouter.md) — единый API для всех моделей включая GPT
- [AI Models](../ai/openai/openai.md) — сравнение GPT-4o и o-серии
- [Tool Use](../agents/tool-use.md) — механизм function calling в агентах
