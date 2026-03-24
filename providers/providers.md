---
type: nav
tags: [providers, api, openrouter, anthropic, openai, google, groq, routing]
status: complete
updated: 2026-03-24
---

# Providers

## Overview

Провайдеры — это точки доступа к AI-моделям через API. Здесь описаны платформы, через которые агент получает доступ к моделям: от агрегаторов до оригинальных поставщиков.

Важно разделять **модель** (что думает) и **провайдера** (откуда берём доступ). Один и тот же Claude Sonnet можно получить через Anthropic напрямую, через OpenRouter или через AWS Bedrock — с разной ценой, лимитами и условиями.

## Priority

**OpenRouter** — основной провайдер. Единый API для всех моделей, маршрутизация, фоллбэки, мониторинг стоимости.

Оригинальные провайдеры подключаются напрямую там, где нужны специфичные фичи (батчинг, fine-tuning, enterprise SLA).

## Structure

```
providers/
├── providers.md        # Этот файл — навигация
├── openrouter.md       # Агрегатор: единый API для всех моделей (приоритет)
├── anthropic.md        # Прямой доступ к Claude API
├── openai.md           # Прямой доступ к GPT / o-серии
├── google.md           # Vertex AI / Gemini API — 1M контекст, multimodal, GCP
└── groq.md             # Groq — быстрый inference (LPU), низкая latency
```

## Topics

### Готово

- **[OpenRouter](openrouter.md)** — единый API, маршрутизация, fallback, мониторинг стоимости
- **[Anthropic API](anthropic.md)** — прямой доступ к Claude: Batch API (50% скидка), Prompt Caching, Extended Thinking
- **[OpenAI API](openai.md)** — GPT-4o, o-серия, Structured Outputs, Assistants API

- **[Google Vertex AI](google.md)** — Gemini через GCP или AI Studio: 1M контекст, Search grounding, multimodal
- **[Groq](groq.md)** — быстрый inference на LPU: 500-2000 tok/s, Llama, Mixtral, низкая латентность

## When to Use What

| Сценарий | Провайдер |
|----------|-----------|
| Основная работа агента, мультимодельный pipeline | OpenRouter |
| Batch API, fine-tuning, максимальный лимит Claude | Anthropic напрямую |
| Assistants API, Batch API GPT, fine-tuning | OpenAI напрямую |
| Нужна минимальная латентность для Llama | Groq |
| Enterprise, данные в GCP, Gemini-фичи | Google Vertex AI |
