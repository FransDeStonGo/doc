---
type: concept
tags: [agents, planning, reasoning, react, chain-of-thought, extended-thinking, thinking, anthropic]
status: complete
updated: 2026-03-24
related:
  - agents/what-is-agent.md
  - agents/tool-use.md
  - agents/workflows.md
---

# Planning & Reasoning

## Overview

Планирование — это то, как агент решает что делать дальше. В отличие от фиксированного workflow, агент опирается на рассуждение: анализирует текущее состояние, выбирает следующее действие, обрабатывает результат. Качество планирования напрямую определяет качество агента.

---

## ReAct — базовый паттерн планирования

**ReAct (Reasoning + Acting)** — самый распространённый паттерн: модель чередует рассуждение и действие, используя обратную связь от окружения.

```
Задача → [Think] → [Act] → [Observe] → [Think] → [Act] → ... → Ответ
```

**Пример цикла:**
```
Task: "Найди цену акций Apple и сравни с Microsoft"

Think: "Нужно узнать текущую цену AAPL. Вызову get_stock_price."
Act:   get_stock_price("AAPL") → $189.50
Observe: AAPL = $189.50

Think: "Теперь нужна цена MSFT."
Act:   get_stock_price("MSFT") → $415.20
Observe: MSFT = $415.20

Think: "Есть оба значения. MSFT в 2.2x дороже AAPL. Могу ответить."
Answer: "Apple торгуется по $189.50, Microsoft — $415.20. Microsoft в 2.2 раза дороже."
```

**Почему это работает:** модель видит результат каждого шага и может скорректировать план. Нет жёсткого пайплайна — агент адаптируется к реальным данным.

---

## Chain-of-Thought (CoT)

**Цепочка рассуждений** — явная запись промежуточных шагов мышления перед финальным ответом. Улучшает качество на задачах с многошаговой логикой.

**Виды:**

| Вид | Как вызывается | Когда |
|-----|---------------|-------|
| **Zero-shot CoT** | "Think step by step" в промпте | Быстро, без примеров |
| **Few-shot CoT** | Примеры с reasoning в промпте | Специфичный стиль мышления |
| **Extended Thinking** | `thinking: {type: "enabled"}` в API | Сложные задачи, нужен контроль |

---

## Extended Thinking (Anthropic)

Extended Thinking — нативная функция Claude: модель генерирует внутренние рассуждения (`thinking` блоки) перед финальным ответом. Эти блоки видны разработчику.

**Когда включать:**
- Многошаговые математические или логические задачи
- Стратегическое планирование агента
- Tool use с анализом промежуточных результатов
- Задачи где нужен "думающий" агент, а не быстрый

**Когда НЕ включать:**
- Простые фактические вопросы
- Задачи с низким требованием к latency
- Routing и классификация

**Базовый пример:**
```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000  # сколько токенов на рассуждение
    },
    messages=[{"role": "user", "content": task}]
)

for block in response.content:
    if block.type == "thinking":
        print("Reasoning:", block.thinking)  # внутренние рассуждения
    elif block.type == "text":
        print("Answer:", block.text)         # финальный ответ
```

**Бюджет токенов:**
- Больше бюджет → лучше для сложных задач, но дороже
- Модель не обязана тратить весь бюджет
- Рекомендуется начинать с 5000–10000, увеличивать при необходимости

### Extended Thinking + Tool Use

При использовании инструментов с extended thinking, `thinking` блоки нужно передавать обратно в API без изменений на каждом шаге:

```python
# Шаг 1: Claude думает и решает вызвать инструмент
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=16000,
    thinking={"type": "enabled", "budget_tokens": 8000},
    tools=[my_tool],
    messages=[{"role": "user", "content": query}]
)

thinking_block = next(b for b in response.content if b.type == "thinking")
tool_use_block = next(b for b in response.content if b.type == "tool_use")

# Шаг 2: Выполняем инструмент и передаём thinking обратно
result = execute_tool(tool_use_block)
continuation = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=16000,
    thinking={"type": "enabled", "budget_tokens": 8000},
    tools=[my_tool],
    messages=[
        {"role": "user", "content": query},
        {"role": "assistant", "content": [thinking_block, tool_use_block]},  # ← передаём thinking
        {"role": "user", "content": [{"type": "tool_result", "tool_use_id": tool_use_block.id, "content": result}]}
    ]
)
```

---

## Interleaved Thinking

Начиная с Claude Sonnet 4.6, модель может думать между вызовами инструментов — не только перед первым действием, но и анализировать результаты каждого шага:

```
Think: "Нужно вычислить 150 * 50..."
  → calculator("150 * 50") = 7500
Think: "Получил $7,500. Теперь проверю среднее по базе..."
  → database("SELECT AVG(revenue)") = 5200
Think: "$7,500 против $5,200 — это +44%. Готов ответить."
Answer: "..."
```

Включается через beta header `interleaved-thinking-2025-05-14`.

---

## Принцип Transparency (Anthropic)

Anthropic настаивает: шаги планирования агента должны быть явно видны. Это не только хорошая практика — это одно из трёх базовых принципов дизайна.

**Практически:**
- Используй extended thinking чтобы видеть рассуждения модели
- Логируй каждый шаг цикла (думаю → действую → наблюдаю)
- Показывай пользователю что агент делает (не только финальный ответ)
- Дай агенту возможность "остановиться и спросить" при неуверенности

---

## Tradeoffs

| Подход | Качество | Скорость | Стоимость | Когда |
|--------|:--------:|:--------:|:---------:|-------|
| No explicit planning | Базовое | Быстро | Дёшево | Простые задачи |
| Zero-shot CoT | Лучше | Средне | Средне | Общий случай |
| Extended Thinking | Высокое | Медленно | Дорого | Сложные рассуждения |
| ReAct loop | Высокое | Медленно | Зависит от шагов | Агентные задачи |

**Compounding errors:** каждый неверный шаг в цепочке рассуждений увеличивает вероятность неверного следующего. Поэтому:
- Чем длиннее цепочка → тем важнее качество каждого шага
- Gate-проверки между шагами снижают риск накопления ошибок
- Human-in-the-loop на критичных точках — надёжнее автономии

---

## See Also

- [Что такое агент](what-is-agent.md) — принцип Transparency как один из базовых
- [Workflow-паттерны](workflows.md) — Evaluator-Optimizer как итеративное планирование
- [Tool Use](tool-use.md) — инструменты как действия в цикле ReAct
