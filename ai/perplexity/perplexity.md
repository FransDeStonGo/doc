---
type: reference
tags: [perplexity, sonar, web-search, citations, real-time, research, grounding]
status: complete
updated: 2026-03-24
related:
  - ai/ai.md
  - providers/openrouter.md
---

# Perplexity AI

## Overview

- **Основана:** 2022, CEO Aravind Srinivas (бывший Google Research)
- **Флагман:** Sonar Pro, Sonar Deep Research
- **Ключевая особенность:** модели нативно интегрированы с веб-поиском и возвращают citations

Perplexity специализируется на поиске с ответами (answer engine). Их собственные модели Sonar построены поверх открытых LLM и оптимизированы под задачи с реальным веб-поиском. Единственный провайдер, где поиск — встроенная способность модели, а не отдельный tool.

## Specifications

| Параметр | Значение |
|----------|---------|
| Флагман | Sonar Pro |
| Контекст | 200k (Sonar Pro), 128k (Sonar) |
| Поиск | нативный, без отдельного tool call |
| Citations | структурированные ссылки на источники в каждом ответе |
| API совместимость | OpenAI-совместимый формат |
| Recency filter | `month`, `week`, `day`, `hour` |

## Models

| Модель | Контекст | Назначение |
|--------|----------|-----------|
| Sonar Pro | 200k | Сложные поисковые запросы, глубокий анализ |
| Sonar | 128k | Быстрый поиск для простых фактических запросов |
| Sonar Reasoning Pro | 128k | Reasoning + поиск: аналитика по актуальным данным |
| Sonar Deep Research | 128k | Автономное многошаговое исследование с отчётом |

## Agent Use Cases

- **Research-агент** — сбор актуальных данных, мониторинг новостей, competitive intelligence
- **Fact-checking** — верификация данных перед принятием решений агентом
- **Knowledge augmentation** — обогащение контекста агента свежей информацией
- **Due diligence** — исследование компаний, технологий, персон с верифицированными источниками
- **News monitoring** — агент отслеживает события в реальном времени с источниками

## Strengths for Agents

- **Real-time web search** — модель нативно ищет в интернете, не нужен отдельный search-инструмент
- **Citations** — каждый ответ содержит ссылки на источники, верифицируемость данных
- **Актуальность** — нет проблемы knowledge cutoff, данные всегда свежие
- **Sonar Deep Research** — агент автономно проводит многошаговое исследование и синтезирует отчёт
- **Простота интеграции** — совместим с OpenAI API форматом
- **Цена** — дешевле GPT-4o при задачах, где нужен поиск

## Setup

```bash
pip install openai  # использует OpenAI-совместимый SDK
```

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.perplexity.ai",
    api_key="pplx-..."
)

response = client.chat.completions.create(
    model="sonar-pro",
    messages=[
        {
            "role": "system",
            "content": "Be precise and cite sources."
        },
        {
            "role": "user",
            "content": "What are the latest AI agent frameworks in 2026?"
        }
    ]
)

# Citations в ответе
print(response.choices[0].message.content)
# response.citations — список источников
```

```python
# С фильтром по свежести
response = client.chat.completions.create(
    model="sonar",
    messages=[{"role": "user", "content": "Latest news about Claude"}],
    extra_body={"search_recency_filter": "day"}
)
```

## Considerations

- Не подходит для задач без поиска — стандартные LLM дешевле и точнее
- Качество зависит от доступности источников в сети
- Меньше контроля над поведением модели по сравнению с "чистыми" LLM API
- Sonar Deep Research может занять несколько минут для глубоких исследований

## See Also

- [OpenRouter (альтернативный доступ к Sonar)](../../providers/openrouter.md)
- [AI — обзор моделей](../ai.md)
