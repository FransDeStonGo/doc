---
type: reference
tags: [deepseek, reasoning, open-source, cheap, moe, chinese, r1, v3]
status: complete
updated: 2026-03-24
related:
  - ai/ai.md
  - ai/qwen/qwen.md
  - ai/meta/meta.md
---

# DeepSeek

## Overview

- **Разработчик:** DeepSeek AI (Китай)
- **Флагман:** DeepSeek V3 (универсальный), DeepSeek R1 (reasoning)
- **Ключевая особенность:** frontier-качество при на порядок меньшей стоимости

Китайская AI-лаборатория, которая в 2025 году взорвала рынок: DeepSeek R1 показал результаты уровня o1 при на порядок меньшей стоимости обучения. Открытые веса, агрессивная цена API. Стал стандартом для бюджетных reasoning-агентов.

## Specifications

| Параметр | Значение |
|----------|---------|
| Флагман | DeepSeek V3 (671B MoE, 37B active) |
| Контекст | 128k токенов |
| Лицензия | MIT (R1 и V3 open weights) |
| Архитектура V3 | MoE — 671B total, 37B active параметров |
| Архитектура R1 | dense reasoning с цепочкой мышлений |
| API совместимость | OpenAI-совместимый формат |

## Models

| Модель | Контекст | Роль в агентной системе | Цена (input/output) |
|--------|----------|-------------------------|---------------------|
| DeepSeek V3 | 128k | Быстрый универсальный агент, дешевле GPT-4o в 10-20x | $0.27 / $1.1 за 1M |
| DeepSeek R1 | 128k | Reasoning-агент: планирование, математика, сложная логика | $0.55 / $2.19 за 1M |
| DeepSeek R1 Distill Qwen 32B | 128k | Self-hosted reasoning, open weights, A100 x1 | open weights |
| DeepSeek R1 Distill Llama 70B | 128k | Self-hosted reasoning на Llama backbone | open weights |

## Agent Use Cases

- **Бюджетный pipeline** — V3 заменяет GPT-4o в большинстве задач при стоимости в 10-20x меньше
- **Reasoning-агент** — R1 для задач с многошаговым планированием, математикой, программированием
- **Self-hosted reasoning** — R1 Distill модели запускаются локально, reasoning без API dependency
- **Code generation** — DeepSeek Coder V2 для coding-агентов

## Strengths for Agents

- **Цена** — DeepSeek V3 один из самых дешёвых frontier-моделей на рынке
- **Reasoning** — R1 конкурирует с o1/o3 на математике и коде
- **Open weights** — R1 и дистилляты можно запускать локально через Ollama / vLLM
- **MoE-архитектура** — эффективнее dense-моделей по соотношению FLOPs/качество

## Setup

```bash
# Через официальный API (OpenAI-совместимый)
pip install openai
```

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.deepseek.com",
    api_key="sk-..."
)

response = client.chat.completions.create(
    model="deepseek-chat",     # V3
    # model="deepseek-reasoner",  # R1
    messages=[{"role": "user", "content": "Hello"}]
)
```

```bash
# Локально через Ollama
ollama run deepseek-r1:32b
ollama run deepseek-v3
```

```python
# Через OpenRouter
client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key="sk-or-..."
)
response = client.chat.completions.create(
    model="deepseek/deepseek-chat-v3-0324",
    messages=[{"role": "user", "content": "Hello"}]
)
```

## Considerations

- Данные обрабатываются на серверах в Китае — критично для enterprise privacy
- Tool use стабильнее у V3, чем у R1 (reasoning модели иногда "уходят в думание" лишний раз)
- Регуляторные риски для компаний в ряде юрисдикций
- R1 генерирует длинные `<think>` блоки — увеличивает выходные токены и стоимость

## See Also

- [Qwen (альтернативный Chinese open-source)](../qwen/qwen.md)
- [Meta Llama (open weights)](../meta/meta.md)
- [AI — обзор моделей](../ai.md)
