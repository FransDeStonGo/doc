---
type: reference
tags: [meta, llama, open-source, self-hosted, fine-tuning, privacy]
status: draft
updated: 2026-03-24
related:
  - ai/ai.md
  - ai/deepseek/deepseek.md
---

# Meta — Llama

## Overview

Meta открыла веса моделей Llama — это изменило рынок. Llama 3.x — production-ready open-source модели, которые можно деплоить локально, файнтюнить и встраивать в продукты без ограничений лицензии OpenAI/Anthropic. Стандарт для self-hosted агентов.

## Models

| Модель | Параметры | Роль в агентной системе |
|--------|-----------|-------------------------|
| Llama 3.3 70B | 70B | Основная модель: баланс качества и скорости на GPU |
| Llama 3.1 405B | 405B | Максимальное качество, нужен серьёзный hardware |
| Llama 3.2 11B Vision | 11B | Лёгкий vision-агент, запускается на одной GPU |
| Llama 3.2 3B / 1B | 1-3B | Edge-агенты, мобильные устройства |

## Strengths for Agents

- **Open weights** — полный контроль: fine-tuning, quantization, локальный деплой
- **Privacy** — данные не покидают инфраструктуру, критично для enterprise
- **Cost** — себестоимость inference при достаточных объёмах ниже API-провайдеров
- **Llama Stack** — официальный фреймворк Meta для агентных систем на базе Llama
- **Ollama / vLLM** — зрелые инструменты локального запуска

## Considerations

- Требует собственной GPU-инфраструктуры для production
- Tool use качественнее с fine-tuned версиями (Groq, Fireworks, Together AI)
- 70B модель хорошо работает на 2× A100 или аналогах
