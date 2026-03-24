---
type: reference
tags: [anthropic, claude, models, tool-use, safety, ai-safety, constitutional-ai]
status: complete
updated: 2026-03-24
related:
  - ai/anthropic/claude/claude.md
  - ai/ai.md
  - providers/anthropic.md
---

# Anthropic

## Overview

- **Основана:** 2021, бывшие сотрудники OpenAI
- **Фокус:** AI Safety — предсказуемое и безопасное поведение моделей
- **Ключевая особенность:** Constitutional AI — модели обучены следовать принципам безопасности, а не просто инструкциям

Anthropic — AI-компания, сфокусированная на безопасности моделей. Создатели семейства моделей Claude. Отличительная черта — Constitutional AI и упор на предсказуемость поведения в агентных сценариях.

## Specifications

| Параметр | Значение |
|----------|---------|
| Флагман | Claude Opus 4.6 |
| Контекст всех моделей | 200k токенов |
| Prompt Caching | до 90% экономии на повторных промптах |
| Batch API | 50% скидка, асинхронная обработка |
| Extended Thinking | явная цепочка рассуждений (Sonnet+) |
| Прямой API | api.anthropic.com |

## Models

| Модель | Роль в агентной системе | Цена (input/output) |
|--------|-------------------------|---------------------|
| Claude Opus 4.6 | Оркестратор, сложные рассуждения, архитектурные решения | $15 / $75 за 1M токенов |
| Claude Sonnet 4.6 | Основная рабочая лошадка: баланс качества и скорости | $3 / $15 за 1M токенов |
| Claude Haiku 4.5 | Быстрые подзадачи, классификация, фильтрация | $0.8 / $4 за 1M токенов |

## Agent Use Cases

- **Оркестратор multi-agent системы** — Opus 4.6 управляет подагентами, Sonnet 4.6 выполняет задачи
- **Coding-агент** — одно из лучших tool use в индустрии: Claude Code, Aider, OpenHands используют Claude
- **Document analysis** — 200k токенов позволяют передать весь контракт или кодовую базу
- **Safety-critical агенты** — предсказуемый отказ от деструктивных действий; важно для автономных систем
- **Агенты с кэшированием** — системные промпты и базы знаний кэшируются для экономии

## Strengths for Agents

- **Tool use** — один из лучших в индустрии: точное следование схемам, минимум галлюцинаций при вызовах
- **Long context** — 200k токенов с хорошим recall по всему окну
- **Instruction following** — высокая точность следования сложным системным промптам
- **Safety** — предсказуемый отказ от деструктивных действий, важно для автономных агентов
- **Prompt Caching** — кэширование prefix-контекста снижает стоимость и latency

## Setup

```python
pip install anthropic
```

```python
import anthropic

client = anthropic.Anthropic(api_key="sk-ant-...")

message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}]
)
print(message.content[0].text)
```

```python
# С tool use
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=[{
        "name": "get_weather",
        "description": "Get current weather",
        "input_schema": {
            "type": "object",
            "properties": {"location": {"type": "string"}},
            "required": ["location"]
        }
    }],
    messages=[{"role": "user", "content": "What's the weather in London?"}]
)
```

## Considerations

- Нет поддержки fine-tuning (в отличие от OpenAI)
- Нет streaming tool calls в batch mode
- Extended Thinking увеличивает стоимость и latency — использовать только для сложных задач

## See Also

- [Claude (детали модели и Claude Code)](claude/claude.md)
- [Anthropic API (провайдер)](../../providers/anthropic.md)
- [AI — обзор моделей](../ai.md)
