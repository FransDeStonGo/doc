---
type: concept
tags: [agents, evaluation, testing, metrics, evals, braintrust, langsmith, quality]
status: complete
updated: 2026-03-24
related:
  - agents/planning.md
  - agents/multi-agent.md
  - agents/observability.md
---

# Evaluation — Оценка качества агентов

## Overview

Оценка агентных систем качественно сложнее, чем тестирование классических LLM. Агент производит не один ответ, а последовательность действий — и финальный результат зависит от всей цепочки шагов. Недетерминизм, ветвление, длинные горизонты действий делают традиционный unit-testing недостаточным.

Цель eval: уверенность, что агент делает правильное в правильных ситуациях, не ломается на edge cases и улучшается, а не деградирует при изменениях.

## How It Works

### Что оцениваем

```
Агент: prompt → [tool_call_1 → result_1] → [tool_call_2 → result_2] → final_answer
             ↑                                                            ↑
       Трасса (trajectory)                                         Финальный ответ
```

Можно оценивать:
- **Финальный ответ** — правильность результата (легко автоматизировать)
- **Трассу** — правильность последовательности действий (труднее, но важнее для отладки)
- **Отдельные шаги** — quality каждого tool call (unit-подход для агентов)

### Источники оценочных сигналов

| Источник | Автоматизация | Стоимость | Применение |
|----------|--------------|-----------|-----------|
| LLM-judge (другая модель оценивает) | высокая | средняя | качество текста, корректность |
| Детерминированные проверки | полная | низкая | формат ответа, наличие полей |
| Человеческая разметка | нет | высокая | субъективное качество, новые задачи |
| Production metrics | автоматическая | низкая | успешность задач, retry rate |

## Patterns

### 1. Unit Evals — отдельные шаги

Тестируем каждый инструмент / подзадачу изолированно:

```python
# Тест: агент правильно формирует поисковый запрос
def test_search_query_generation():
    response = agent.plan_search(
        goal="Find recent papers about multi-agent systems"
    )
    assert "multi-agent" in response.query.lower()
    assert response.date_filter is not None  # не забыл про фильтр дат
```

**Плюс:** быстро запускать, легко отлаживать
**Минус:** не проверяет взаимодействие компонентов

---

### 2. End-to-End Evals — полный сценарий

Запускаем агента на реальных задачах из golden set:

```python
# Структура eval-датасета
eval_cases = [
    {
        "input": "Find and summarize 3 papers on RAG from 2025",
        "expected": {
            "num_papers": 3,
            "has_summary": True,
            "year_filter_used": True,
        },
        "grader": check_research_task
    }
]

def check_research_task(output, expected):
    papers = extract_papers(output)
    return len(papers) >= expected["num_papers"]
```

**Плюс:** проверяет реальное поведение
**Минус:** дорого, медленно, флакуют из-за недетерминизма

---

### 3. LLM-as-Judge

Использовать другую (более сильную) модель для оценки:

```python
judge_prompt = """
Оцени ответ агента на задачу. Критерии:
1. Полнота (1-5): все ли аспекты задачи охвачены?
2. Точность (1-5): нет ли фактических ошибок?
3. Использование tools (pass/fail): агент использовал нужные инструменты?

Задача: {task}
Ответ агента: {response}
"""
```

**Важно:** judge должен быть сильнее оцениваемого агента; учитывать positional bias (judge предпочитает первый из двух вариантов).

---

### 4. Trajectory Evaluation

Оценка последовательности действий, а не только результата:

```python
# Проверить что агент не сделал лишних шагов
def eval_trajectory(trace):
    tool_calls = [step for step in trace if step.type == "tool_call"]

    # Нет избыточных поисков
    search_calls = [t for t in tool_calls if t.name == "search"]
    assert len(search_calls) <= 3, "Too many search calls (efficiency issue)"

    # Агент проверил результат перед финальным ответом
    assert any(t.name == "verify" for t in tool_calls)
```

## Tools

| Инструмент | Тип | Сильные стороны |
|-----------|-----|----------------|
| **Braintrust** | SaaS | датасеты, LLM judge, A/B, CI интеграция |
| **LangSmith** | SaaS | интеграция с LangChain, трассировка + eval |
| **PromptFoo** | open-source | CLI-first, быстрое добавление тест-кейсов |
| **RAGAS** | open-source | специализация на RAG: faithfulness, relevancy |
| **Inspect AI** | open-source | UK AISI, детальный анализ трасс |
| **Pytest** | open-source | unit evals с детерминированными проверками |

## Tradeoffs

| Подход | Скорость | Стоимость | Надёжность | Применение |
|--------|----------|-----------|-----------|-----------|
| Unit evals | быстрая | низкая | высокая | регрессии при рефакторинге |
| E2E evals | медленная | высокая | средняя | релизные проверки |
| LLM judge | средняя | средняя | средняя | субъективное качество |
| Human eval | очень медленная | очень высокая | высокая | создание golden set |

**Практическое правило:** начинать с маленького (10-20 задач) golden set из реальных production-кейсов, запускать на каждом PR. Большие eval-сьюты — только перед релизами.

## See Also

- [Planning — ReAct и reasoning стратегии](planning.md)
- [Multi-Agent — тестирование orchestration](multi-agent.md)
- [Observability — трассировка для отладки](observability.md)
