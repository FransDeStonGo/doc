---
type: reference
tags: [deepseek, reasoning, open-source, cheap, moe, chinese]
status: draft
updated: 2026-03-24
related:
  - ai/ai.md
  - ai/qwen/qwen.md
  - ai/meta/meta.md
---

# DeepSeek

## Overview

Китайская AI-лаборатория, которая в 2025 году взорвала рынок: DeepSeek R1 показал результаты уровня o1 при на порядок меньшей стоимости обучения. Открытые веса, агрессивная цена API. Стал стандартом для бюджетных reasoning-агентов.

## Models

| Модель | Контекст | Роль в агентной системе |
|--------|----------|-------------------------|
| DeepSeek V3 | 128k | Быстрый универсальный агент, дешевле GPT-4o в 10-20x |
| DeepSeek R1 | 128k | Reasoning-агент: планирование, математика, сложная логика |
| DeepSeek R1 Distill (Qwen/Llama) | 128k | Self-hosted reasoning, open weights |

## Strengths for Agents

- **Цена** — DeepSeek V3 один из самых дешёвых frontier-моделей на рынке
- **Reasoning** — R1 конкурирует с o1/o3 на математике и коде
- **Open weights** — R1 и дистилляты можно запускать локально через Ollama / vLLM
- **MoE-архитектура** — эффективнее dense-моделей по соотношению FLOPs/качество

## Considerations

- Данные обрабатываются на серверах в Китае — критично для enterprise privacy
- Tool use стабильнее у V3, чем у R1 (reasoning модели иногда "уходят в думание" лишний раз)
- Регуляторные риски для компаний в ряде юрисдикций
