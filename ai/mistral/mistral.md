---
type: reference
tags: [mistral, codestral, open-source, gdpr, european, function-calling, moe]
status: complete
updated: 2026-03-24
related:
  - ai/ai.md
  - providers/openrouter.md
---

# Mistral AI

## Overview

- **Основана:** 2023, Франция (бывшие сотрудники DeepMind и Meta)
- **Флагман:** Mistral Large 2
- **Ключевая особенность:** сильные open-source модели + GDPR-compliant европейская инфраструктура

Европейский AI-стартап с сильными open-source моделями. Mistral Large конкурирует с GPT-4o по цене/качеству. Модели известны эффективностью — высокое качество при меньшем числе параметров. Хороший выбор для агентов с европейскими требованиями к данным.

## Specifications

| Параметр | Значение |
|----------|---------|
| Флагман | Mistral Large 2 |
| Контекст | 128k токенов (Large), 32k (Codestral) |
| Лицензия | MRL (часть моделей Apache 2.0) |
| API совместимость | OpenAI-совместимый format |
| Датацентры | ЕС (GDPR-compliant) |
| Платформа | La Plateforme (la.mistral.ai) |

## Models

| Модель | Контекст | Роль в агентной системе | Цена (input/output) |
|--------|----------|-------------------------|---------------------|
| Mistral Large 2 | 128k | Основной агент: сложный reasoning, tool use | $2 / $6 за 1M |
| Mistral Small 3 | 128k | Быстрые подзадачи, дешевле Large в 3-4x | $0.1 / $0.3 за 1M |
| Codestral | 32k | Специализация на коде: generation, completion, FIM | $0.2 / $0.6 за 1M |
| Mixtral 8x22B | 64k | Open-source MoE, self-hosted enterprise | open weights |

## Agent Use Cases

- **GDPR-sensitive данные** — европейская инфраструктура, данные остаются в ЕС
- **Coding-агент** — Codestral с FIM (Fill-In-the-Middle) для code completion
- **Бюджетный агент** — Mistral Small 3 одна из дешёвых качественных моделей
- **Self-hosted enterprise** — Mixtral 8x22B для корпоративного деплоя с open weights
- **MoE-архитектура** — эффективное использование ресурсов при inference

## Strengths for Agents

- **Function calling** — зрелая реализация, совместима с OpenAI-форматом
- **Codestral** — одна из лучших code-специализированных моделей для coding-агентов
- **Le Chat / La Plateforme** — европейская инфраструктура, GDPR-compliant
- **Open weights** — Mixtral и Mistral 7B доступны для локального деплоя
- **Цена** — Mistral Small одна из самых дешёвых качественных моделей на рынке

## Setup

```bash
pip install mistralai
```

```python
from mistralai import Mistral

client = Mistral(api_key="...")

response = client.chat.complete(
    model="mistral-large-latest",
    messages=[{"role": "user", "content": "Hello"}]
)
print(response.choices[0].message.content)
```

```python
# Через OpenAI SDK (совместимый API)
from openai import OpenAI

client = OpenAI(
    base_url="https://api.mistral.ai/v1",
    api_key="..."
)
```

```python
# Codestral FIM (fill-in-the-middle)
response = client.fim.complete(
    model="codestral-latest",
    prompt="def fibonacci(n):",
    suffix="    return result"
)
```

## Considerations

- Контекстное окно меньше чем у Gemini и Claude (128k vs 200k/1M)
- Экосистема и комьюнити меньше чем у OpenAI
- Codestral доступен отдельно через codestral.mistral.ai

## See Also

- [OpenRouter (агрегатор с Mistral)](../../providers/openrouter.md)
- [AI — обзор моделей](../ai.md)
