---
type: reference
tags: [google, gemini, long-context, multimodal, vertex-ai, grounding, code-execution]
status: complete
updated: 2026-03-24
related:
  - ai/ai.md
  - ai/cli-agents/gemini-cli/gemini-cli.md
  - providers/google.md
---

# Google DeepMind — Gemini

## Overview

- **Разработчик:** Google DeepMind
- **Флагман:** Gemini 2.5 Pro
- **Ключевая особенность:** рекордное контекстное окно 1M токенов + нативная мультимодальность

Google DeepMind создаёт семейство моделей Gemini. Ключевое преимущество — нативная мультимодальность (текст, изображения, аудио, видео, код) и рекордное контекстное окно. Тесная интеграция с экосистемой Google Cloud и Workspace.

## Specifications

| Параметр | Значение |
|----------|---------|
| Макс. контекст | 1M токенов (Gemini 2.5 Pro) |
| Модальности | текст, изображения, аудио, видео, PDF |
| Google Search | нативный grounding через API |
| Code Execution | встроенный sandbox для Python |
| Vertex AI | enterprise-инфраструктура на GCP |
| AI Studio | бесплатный dev-endpoint с generous квотами |

## Models

| Модель | Контекст | Роль в агентной системе | Цена (input/output) |
|--------|----------|-------------------------|---------------------|
| Gemini 2.5 Pro | 1M | Анализ огромных кодовых баз, длинных документов, видео | $1.25 / $10 за 1M |
| Gemini 2.0 Flash | 1M | Быстрый агент с большим контекстом, Realtime API | $0.1 / $0.4 за 1M |
| Gemini 2.0 Flash Lite | 1M | Массовые дешёвые вызовы в pipeline | $0.075 / $0.3 за 1M |

## Agent Use Cases

- **Whole-repo анализ** — 1M токенов позволяют загрузить весь репозиторий за один запрос
- **Document intelligence** — нативная обработка PDF, скриншотов, таблиц без OCR preprocessing
- **Grounded search агент** — встроенный Google Search для доступа к актуальным данным
- **Video analysis** — агент анализирует видеозаписи, скринкасты, демо напрямую
- **Code Execution** — агент пишет Python-код и сразу выполняет его в sandbox

## Strengths for Agents

- **1M токенов** — можно загрузить весь репозиторий или базу знаний в один запрос
- **Нативная мультимодальность** — агент работает с документами, скриншотами, видео без preprocessing
- **Google Search grounding** — встроенный доступ к актуальному поиску прямо в API
- **Code Execution** — встроенный sandbox для выполнения кода
- **Vertex AI** — enterprise-инфраструктура: безопасность, SLA, интеграция с GCP

## Setup

```bash
pip install google-generativeai
```

```python
import google.generativeai as genai

genai.configure(api_key="AIza...")

model = genai.GenerativeModel("gemini-2.5-pro")
response = model.generate_content("Hello")
print(response.text)
```

```python
# С Google Search grounding
model = genai.GenerativeModel(
    "gemini-2.0-flash",
    tools=["google_search"]
)
response = model.generate_content("What happened in AI this week?")
```

```python
# Через OpenAI-совместимый endpoint (Vertex AI)
from openai import OpenAI

client = OpenAI(
    base_url="https://generativelanguage.googleapis.com/v1beta/openai/",
    api_key="AIza..."
)
```

## Considerations

- Качество tool use исторически уступало Claude и GPT-4o, улучшается с каждой версией
- Большой контекст не означает хороший recall — деградация на очень длинных промптах
- Экосистема агентных фреймворков меньше чем у OpenAI
- Vertex AI требует GCP аккаунт и настройку IAM

## See Also

- [Google Vertex AI (провайдер)](../../providers/google.md)
- [Gemini CLI](../cli-agents/gemini-cli/gemini-cli.md)
- [AI — обзор моделей](../ai.md)
