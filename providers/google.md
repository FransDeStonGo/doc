---
type: reference
tags: [providers, google, vertex-ai, gemini, ai-studio, gcp, grounding, long-context]
status: complete
updated: 2026-03-24
related:
  - ai/google/google.md
  - providers/providers.md
  - providers/openrouter.md
---

# Google — Vertex AI / Gemini API

## Overview

- **Вендор:** Google Cloud / DeepMind
- **Два endpoint:** AI Studio (dev, бесплатно) и Vertex AI (prod, GCP)
- **Ключевая особенность:** 1M контекст, нативный Google Search grounding, multimodal

Google предоставляет два способа доступа к Gemini: через AI Studio для быстрого старта и через Vertex AI для production с enterprise SLA. Оба поддерживают OpenAI-совместимый формат.

## Specifications

| Параметр | AI Studio | Vertex AI |
|----------|-----------|-----------|
| Среда | Dev / personal | Production / enterprise |
| Auth | API Key | GCP Service Account / ADC |
| SLA | нет | есть |
| Регионы | глобально | мультирегион GCP |
| Fine-tuning | нет | есть |
| Batch API | нет | есть |
| Цена | free tier + pay-per-use | pay-per-use |

## Models (через Vertex AI)

| Модель | Контекст | Цена (input/output за 1M) |
|--------|----------|---------------------------|
| gemini-2.5-pro | 1M | $1.25 / $10 |
| gemini-2.0-flash | 1M | $0.1 / $0.4 |
| gemini-2.0-flash-lite | 1M | $0.075 / $0.3 |

## Agent Use Cases

- **Whole-codebase analysis** — загрузить весь репозиторий в 1M контекст за один запрос
- **Multimodal pipeline** — агент читает PDF, скриншоты, видео без preprocessing
- **Grounded search** — встроенный Google Search в запросах для актуальных данных
- **Code Execution** — Python sandbox для вычислений прямо в API
- **Enterprise data в GCP** — данные остаются в Google Cloud, compliance-friendly

## Setup

### AI Studio (быстрый старт)

```bash
pip install google-generativeai
```

```python
import google.generativeai as genai

genai.configure(api_key="AIza...")  # из aistudio.google.com

model = genai.GenerativeModel("gemini-2.5-pro")
response = model.generate_content("Hello")
print(response.text)
```

### Vertex AI (production)

```bash
pip install google-cloud-aiplatform
gcloud auth application-default login
```

```python
import vertexai
from vertexai.generative_models import GenerativeModel

vertexai.init(project="my-project", location="us-central1")

model = GenerativeModel("gemini-2.5-pro")
response = model.generate_content("Hello")
print(response.text)
```

### OpenAI-совместимый endpoint

```python
from openai import OpenAI

# AI Studio
client = OpenAI(
    base_url="https://generativelanguage.googleapis.com/v1beta/openai/",
    api_key="AIza..."
)

# Vertex AI
client = OpenAI(
    base_url=f"https://{LOCATION}-aiplatform.googleapis.com/v1/projects/{PROJECT}/locations/{LOCATION}/endpoints/openapi",
    api_key=access_token
)

response = client.chat.completions.create(
    model="gemini-2.5-pro",
    messages=[{"role": "user", "content": "Hello"}]
)
```

### Google Search grounding

```python
from google.generativeai.types import Tool, GoogleSearchRetrieval

model = genai.GenerativeModel(
    "gemini-2.0-flash",
    tools=[Tool(google_search=GoogleSearchRetrieval())]
)
response = model.generate_content("Latest news about AI agents in 2026")
# response.candidates[0].grounding_metadata — источники
```

## Considerations

- Vertex AI требует GCP-проект, биллинг и настройку IAM — не подходит для быстрого прототипа
- AI Studio имеет rate limits (requests per minute) даже на платном плане
- 1M контекст стоит дорого — использовать осознанно, не по умолчанию
- OpenAI-совместимый endpoint не поддерживает все Gemini-специфичные параметры

## See Also

- [Gemini — модель](../ai/google/google.md)
- [OpenRouter (доступ к Gemini без GCP)](openrouter.md)
- [Providers — обзор](providers.md)
