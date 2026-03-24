---
type: nav
tags: [ai, models, comparison, anthropic, openai, google, meta, mistral, deepseek, qwen, perplexity]
status: complete
updated: 2026-03-24
---

# AI Platforms & Models

## Overview

Подборка AI-платформ и языковых моделей, актуальных для работы в паре с AI-агентом. Для каждого производителя — сравнение моделей, сильные стороны в агентных сценариях и практические выводы.

## Structure

```
ai/
├── ai.md               # Этот файл — навигация по разделу
├── anthropic/          # Claude Opus / Sonnet / Haiku + Claude Code CLI
│   ├── anthropic.md
│   └── claude/
│       └── claude.md
├── openai/             # GPT-4o, o3, o4-mini
│   └── openai.md
├── google/             # Gemini 2.5 Pro, Flash — рекордный контекст 1M
│   └── google.md
├── meta/               # Llama 3.x — open-source, self-hosted
│   └── meta.md
├── mistral/            # Mistral Large, Codestral — европейский GDPR-compliant
│   └── mistral.md
├── deepseek/           # DeepSeek V3 / R1 — лучшая цена/качество на рынке
│   └── deepseek.md
├── qwen/               # Qwen2.5 — open-source от Alibaba, сильный code и vision
│   └── qwen.md
├── perplexity/         # Sonar — модели с нативным веб-поиском и citations
│   └── perplexity.md
└── cli-agents/         # CLI-агенты: Claude Code, Aider, Goose, OpenHands и др.
    └── cli-agents.md
```

## Quick Comparison

| Производитель | Лучшая модель | Контекст | Фишка для агентов |
|---------------|---------------|----------|--------------------|
| Anthropic | Claude Sonnet 4.6 | 200k | Tool use, safety, Claude Code CLI |
| OpenAI | o3 / GPT-4o | 128-200k | Structured outputs, экосистема фреймворков |
| Google | Gemini 2.5 Pro | 1M | Огромный контекст, мультимодальность |
| Meta | Llama 3.3 70B | 128k | Open weights, локальный деплой, privacy |
| Mistral | Mistral Large 2 | 128k | Цена/качество, Codestral для кода |
| DeepSeek | DeepSeek V3 / R1 | 128k | Самая низкая цена, reasoning open weights |
| Qwen | Qwen2.5-72B / Coder-32B | 128k | Open-source, лучший код среди open моделей |
| Perplexity | Sonar Pro / Deep Research | 200k | Нативный веб-поиск, citations, актуальные данные |

## Selection Guide

- **Нужен лучший tool use и агентная архитектура** → Anthropic Claude
- **Нужна экосистема и фреймворки** → OpenAI
- **Нужен огромный контекст или мультимодальность** → Google Gemini
- **Нужен self-hosted / privacy / fine-tuning** → Meta Llama
- **Нужен код-агент по разумной цене** → Mistral Codestral
- **Нужна максимальная экономия или локальный reasoning** → DeepSeek
- **Нужен open-source code-агент** → Qwen2.5-Coder
- **Нужен агент с актуальными данными из интернета** → Perplexity Sonar
