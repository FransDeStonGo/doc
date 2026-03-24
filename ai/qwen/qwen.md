---
type: reference
tags: [qwen, alibaba, open-source, code, vision, reasoning, chinese, dashscope]
status: complete
updated: 2026-03-24
related:
  - ai/ai.md
  - ai/deepseek/deepseek.md
  - ai/cli-agents/qwen-code/qwen-code.md
---

# Qwen (Alibaba Cloud)

## Overview

- **Разработчик:** Alibaba Cloud / Tongyi Lab
- **Флагман:** Qwen2.5-72B-Instruct, Qwen2.5-Coder-32B
- **Ключевая особенность:** широкое семейство специализированных open-source моделей

Qwen — семейство моделей от Alibaba Cloud. После DeepSeek стал вторым громким open-source успехом из Китая. Qwen2.5 серия покрывает широкий спектр: от мультимодальных до специализированных code и math моделей. Активно используется как база для дистилляций других моделей.

## Specifications

| Параметр | Значение |
|----------|---------|
| Флагман | Qwen2.5-72B-Instruct |
| Контекст | 128k токенов |
| Лицензия | Apache 2.0 (большинство моделей) |
| API платформа | DashScope (Alibaba Cloud) |
| API совместимость | OpenAI-совместимый формат |
| Open weights | HuggingFace, ModelScope |

## Models

| Модель | Параметры | Роль в агентной системе |
|--------|-----------|-------------------------|
| Qwen2.5-72B-Instruct | 72B | Основной агент: конкурирует с GPT-4o по бенчмаркам |
| Qwen2.5-Coder-32B | 32B | Code-агент: один из лучших open-source для кода |
| Qwen2.5-VL-72B | 72B | Vision-агент: анализ документов, скриншотов, UI |
| QwQ-32B | 32B | Reasoning-модель: конкурент R1 при меньшем размере |
| Qwen2.5-7B / 3B | 3-7B | Edge и мобильные агенты, локальный деплой |

## Agent Use Cases

- **Self-hosted coding-агент** — Qwen2.5-Coder-32B запускается на одном A100, код-качество
- **Vision-агент** — Qwen2.5-VL для анализа скриншотов UI, документов, графиков
- **Reasoning без API** — QwQ-32B как локальная альтернатива o3 для reasoning задач
- **Бюджетный coding pipeline** — Qwen Code CLI-агент на базе Qwen2.5-Coder
- **Дистилляция** — Qwen как backbone для собственных fine-tuned моделей

## Strengths for Agents

- **Open weights** — большинство моделей полностью открыты, доступны на HuggingFace
- **Qwen2.5-Coder** — один из топовых open-source code-агентов
- **QwQ-32B** — reasoning при скромном размере, запускается на одной A100
- **Мультимодальность** — Qwen-VL нативно понимает изображения, документы, UI-скриншоты
- **Alibaba Cloud** — API через DashScope, интеграция с экосистемой Alibaba
- **База для дистилляций** — DeepSeek R1 Distill-Qwen модели используют Qwen как backbone

## Setup

```bash
# Локально через Ollama
ollama run qwen2.5-coder:32b
ollama run qwq:32b
```

```python
# Через DashScope API (OpenAI-совместимый)
from openai import OpenAI

client = OpenAI(
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1",
    api_key="sk-..."
)

response = client.chat.completions.create(
    model="qwen2.5-72b-instruct",
    messages=[{"role": "user", "content": "Hello"}]
)
```

```python
# Через OpenRouter
client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key="sk-or-..."
)
response = client.chat.completions.create(
    model="qwen/qwen-2.5-72b-instruct",
    messages=[{"role": "user", "content": "Hello"}]
)
```

## Considerations

- Как и DeepSeek — китайская инфраструктура, важно для enterprise privacy
- Документация и комьюнити преимущественно на китайском для новых фич
- API через DashScope менее зрелый чем OpenAI/Anthropic по экосистеме интеграций
- QwQ-32B может генерировать очень длинные reasoning-цепочки

## See Also

- [DeepSeek (альтернативный Chinese open-source)](../deepseek/deepseek.md)
- [Qwen Code CLI](../cli-agents/qwen-code/qwen-code.md)
- [AI — обзор моделей](../ai.md)
