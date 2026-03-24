---
type: reference
tags: [openai, gpt, o-series, models, structured-outputs, function-calling, assistants]
status: complete
updated: 2026-03-24
related:
  - ai/ai.md
  - ai/cli-agents/codex-cli/codex-cli.md
  - providers/openai.md
---

# OpenAI

## Overview

- **Основана:** 2015, пионер рынка LLM
- **Флагман:** GPT-4o (универсальный) + o3/o4-mini (reasoning)
- **Ключевая особенность:** де-факто стандарт совместимости — большинство фреймворков заточены под OpenAI API

OpenAI создали GPT-серию и o-серию (reasoning models). Широкая экосистема: API, Assistants API, функции, Structured Outputs. Первый выбор по умолчанию для большинства агентных фреймворков.

## Specifications

| Параметр | Значение |
|----------|---------|
| Флагман | GPT-4o / o3 |
| Контекст GPT-4o | 128k токенов |
| Контекст o3 | 200k токенов |
| Structured Outputs | гарантированный JSON по JSON Schema |
| Batch API | 50% скидка, 24-часовое окно |
| Assistants API | встроенные threads, file search, code interpreter |

## Models

| Модель | Контекст | Роль в агентной системе | Цена (input/output) |
|--------|----------|-------------------------|---------------------|
| GPT-4o | 128k | Универсальный агент: текст + vision, быстрый tool use | $2.5 / $10 за 1M |
| GPT-4o mini | 128k | Лёгкие подзадачи, дешёвые вызовы в pipeline | $0.15 / $0.6 за 1M |
| o3 | 200k | Глубокое планирование, математика, сложная логика | $10 / $40 за 1M |
| o4-mini | 128k | Reasoning по разумной цене, быстрее o3 | $1.1 / $4.4 за 1M |

## Agent Use Cases

- **Универсальный рабочий агент** — GPT-4o как основная модель в pipeline, хорошо следует инструкциям
- **Reasoning-задачи** — o3/o4-mini для планирования многошаговых задач, анализа кода, математики
- **Assistants API** — встроенные threads и file search избавляют от реализации памяти вручную
- **Structured extraction** — Structured Outputs для надёжного парсинга ответов в JSON
- **Coding-агент** — Codex CLI, GitHub Copilot основаны на OpenAI моделях
- **Multi-modal агент** — GPT-4o понимает изображения, скриншоты UI, диаграммы

## Strengths for Agents

- **Structured Outputs** — гарантированный JSON по схеме, критично для agent pipelines
- **Assistants API** — встроенные threads, file search, code interpreter без самостоятельной реализации
- **Function calling** — зрелый, стабильный, широко поддерживается во фреймворках
- **o-серия** — модели с "думанием": лучше справляются с многошаговым планированием
- **Ecosystem** — LangChain, AutoGPT, CrewAI, OpenAI Swarm — всё заточено под OpenAI API

## Setup

```bash
pip install openai
```

```python
from openai import OpenAI

client = OpenAI(api_key="sk-...")

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}]
)
print(response.choices[0].message.content)
```

```python
# Structured Outputs
from pydantic import BaseModel

class Answer(BaseModel):
    reasoning: str
    result: str

response = client.beta.chat.completions.parse(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Solve 2+2"}],
    response_format=Answer,
)
answer = response.choices[0].message.parsed
```

## Considerations

- Контекстное окно меньше чем у Claude и Gemini (128k vs 200k/1M)
- o-модели медленнее из-за chain-of-thought на стороне сервера
- Цена на o3 высокая для высоконагруженных агентных систем
- Нет нативного веб-поиска в базовом API (нужен отдельный tool)

## See Also

- [OpenAI API (провайдер)](../../providers/openai.md)
- [Codex CLI](../cli-agents/codex-cli/codex-cli.md)
- [AI — обзор моделей](../ai.md)
