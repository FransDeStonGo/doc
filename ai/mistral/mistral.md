---
type: reference
tags: [mistral, codestral, open-source, gdpr, european, function-calling]
status: draft
updated: 2026-03-24
related:
  - ai/ai.md
---

# Mistral AI

## Overview

Европейский AI-стартап с сильными open-source моделями. Mistral Large конкурирует с GPT-4o по цене/качеству. Модели известны эффективностью — высокое качество при меньшем числе параметров. Хороший выбор для агентов с европейскими требованиями к данным (GDPR).

## Models

| Модель | Контекст | Роль в агентной системе |
|--------|----------|-------------------------|
| Mistral Large 2 | 128k | Основной агент: сложный reasoning, tool use |
| Mistral Small 3 | 128k | Быстрые подзадачи, дешевле Large в 3-4x |
| Codestral | 32k | Специализация на коде: generation, completion, FIM |
| Mixtral 8x22B | 64k | Open-source MoE, self-hosted enterprise |

## Strengths for Agents

- **Function calling** — зрелая реализация, совместима с OpenAI-форматом
- **Codestral** — одна из лучших code-специализированных моделей для coding-агентов
- **Le Chat / La Plateforme** — европейская инфраструктура, GDPR-compliant
- **Open weights** — Mixtral и Mistral 7B доступны для локального деплоя
- **Цена** — Mistral Small одна из самых дешёвых качественных моделей на рынке

## Considerations

- Контекстное окно меньше чем у Gemini и Claude
- Экосистема и комьюнити меньше чем у OpenAI
