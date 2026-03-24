---
type: concept
tags: [agents, tool-use, function-calling, mcp, api, anthropic]
status: complete
updated: 2026-03-24
related:
  - agents/what-is-agent.md
  - agents/workflows.md
  - mcp/protocol.md
---

# Tool Use

## Overview

Tool use (function calling) — механизм, через который LLM взаимодействует с внешним миром. Модель не выполняет инструменты напрямую: она генерирует структурированный запрос на вызов, а код разработчика выполняет реальное действие и возвращает результат. Это главный примитив агентных систем — добавление даже простых инструментов даёт непропорционально большой прирост возможностей.

> Источник: Anthropic Docs "Tool use with Claude" (2024)

---

## Типы инструментов

Anthropic различает два типа:

| Тип | Где выполняется | Кто реализует |
|-----|----------------|---------------|
| **Client tools** | На стороне разработчика | Разработчик |
| **Server tools** | На серверах Anthropic | Anthropic |

**Client tools** — кастомные инструменты: вызовы API, запись файлов, выполнение кода, запросы к БД. Разработчик определяет схему и реализует логику.

**Server tools** — встроенные инструменты Anthropic: `web_search`, `web_fetch`. Объявляются в запросе, но выполняются автоматически на серверах Anthropic. Используют версионированные имена: `web_search_20250305`.

---

## Структура инструмента

Каждый client tool определяется тремя полями:

```json
{
  "name": "get_weather",
  "description": "Get the current weather in a given location. Returns temperature and conditions.",
  "input_schema": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "The city and state, e.g. San Francisco, CA"
      },
      "unit": {
        "type": "string",
        "enum": ["celsius", "fahrenheit"],
        "description": "Temperature unit"
      }
    },
    "required": ["location"]
  }
}
```

| Поле | Обязательное | Описание |
|------|:-----------:|---------|
| `name` | ✓ | Уникальный идентификатор. Только `a-z`, `0-9`, `_`, `-`. Макс 64 символа |
| `description` | ✓ | Объяснение что делает инструмент и когда его использовать |
| `input_schema` | ✓ | JSON Schema параметров (тип object, properties, required) |

---

## Цикл Tool Use

```
┌─────────────────────────────────────────────────────┐
│ 1. Запрос                                           │
│    User → API (tools + messages)                    │
│                          ↓                          │
│ 2. Решение модели                                   │
│    Claude → stop_reason: "tool_use"                 │
│    Claude → tool_use block { name, input }          │
│                          ↓                          │
│ 3. Выполнение (на стороне разработчика)             │
│    Code → выполняет инструмент → получает результат │
│                          ↓                          │
│ 4. Возврат результата                               │
│    Code → API (tool_result block)                   │
│                          ↓                          │
│ 5. Финальный ответ                                  │
│    Claude → итоговый текст пользователю             │
└─────────────────────────────────────────────────────┘
```

**Ключевые сигналы:**
- `stop_reason: "tool_use"` — модель хочет вызвать инструмент
- `stop_reason: "end_turn"` — модель завершила работу
- `stop_reason: "pause_turn"` — server tool loop достиг лимита (10 итераций), нужно продолжить

**Шаг 3 опционален:** если достаточно самого факта вызова (без возврата результата), цикл можно оборвать на шаге 2.

---

## Пример вызова (Python)

```python
import anthropic

client = anthropic.Anthropic()

# Шаг 1: Запрос с инструментом
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=[{
        "name": "get_weather",
        "description": "Get the current weather in a given location",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {"type": "string", "description": "City and state"}
            },
            "required": ["location"]
        }
    }],
    messages=[{"role": "user", "content": "What's the weather in SF?"}]
)

# Шаг 2: Claude решает вызвать инструмент
# response.stop_reason == "tool_use"
tool_use = next(b for b in response.content if b.type == "tool_use")

# Шаг 3: Выполняем инструмент
result = fetch_weather(tool_use.input["location"])

# Шаг 4: Возвращаем результат
final = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=[...],
    messages=[
        {"role": "user", "content": "What's the weather in SF?"},
        {"role": "assistant", "content": response.content},
        {"role": "user", "content": [{
            "type": "tool_result",
            "tool_use_id": tool_use.id,
            "content": str(result)
        }]}
    ]
)
```

---

## Параллельные вызовы

Claude может вызывать несколько инструментов за один ответ, если задача это позволяет:

```json
// Один ответ, два вызова
[
  {"type": "tool_use", "id": "tu_1", "name": "get_weather", "input": {"location": "SF"}},
  {"type": "tool_use", "id": "tu_2", "name": "get_weather", "input": {"location": "NY"}}
]
```

Все результаты возвращаются в одном `tool_result` массиве. Если хочешь запретить параллельные вызовы — используй `"disable_parallel_tool_use": true`.

---

## Best Practices — описание инструментов

Качество tool descriptions критично. Модель решает какой инструмент вызвать и с какими параметрами, опираясь исключительно на текст описания.

**Правила хорошего описания:**

1. **Объясни что делает инструмент** — не только название, но и реальное действие
2. **Укажи когда использовать** — в каких ситуациях этот инструмент подходит
3. **Опиши ограничения** — что инструмент не умеет или в каких случаях вернёт ошибку
4. **Описывай параметры конкретно** — включай примеры значений

```json
// Плохо
"description": "Gets weather"

// Хорошо
"description": "Returns current weather conditions (temperature, humidity, wind)
for a given city. Use when user asks about current weather.
Does not support forecasts or historical data."
```

**Poka-yoke в параметрах** — защита от ошибок на уровне схемы:
- Требуй абсолютные пути вместо относительных: `"description": "Absolute path, e.g. /home/user/file.txt"`
- Используй `enum` для ограниченных значений
- Добавляй `strict: true` для гарантии соответствия схеме в production

---

## MCP и Tool Use

MCP-инструменты используют тот же механизм, что и client tools. Разница в именовании поля схемы:

| Источник | Поле схемы |
|---------|-----------|
| Anthropic API | `input_schema` |
| MCP | `inputSchema` |

При конвертации MCP-инструментов для Anthropic API — просто переименовать поле:

```python
claude_tools = [{
    "name": tool.name,
    "description": tool.description,
    "input_schema": tool.inputSchema  # inputSchema → input_schema
} for tool in mcp_tools]
```

---

## Tradeoffs

**Плюсы:**
- Единственный способ дать агенту доступ к внешним данным и действиям
- Строго типизированный интерфейс снижает галлюцинации в параметрах
- Параллельные вызовы ускоряют выполнение

**Минусы / риски:**
- Каждый вызов инструмента — дополнительные токены и latency
- Плохое описание → неверный выбор инструмента или неверные параметры
- Нужна обработка ошибок: что если инструмент вернул ошибку или упал

**Обработка ошибок:** возвращай ошибку в `tool_result` с `is_error: true` — Claude учтёт это в следующем шаге:

```json
{
  "type": "tool_result",
  "tool_use_id": "tu_1",
  "is_error": true,
  "content": "Location not found. Please provide city and country."
}
```

---

## See Also

- [Что такое агент](what-is-agent.md) — роль инструментов в агентной архитектуре
- [Workflow-паттерны](workflows.md) — как tool use встраивается в паттерны
- [MCP Protocol](../mcp/protocol.md) — стандарт для инструментов
