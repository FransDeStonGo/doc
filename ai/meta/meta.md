---
type: reference
tags: [meta, llama, open-source, self-hosted, fine-tuning, privacy, ollama, vllm]
status: complete
updated: 2026-03-24
related:
  - ai/ai.md
  - ai/deepseek/deepseek.md
---

# Meta — Llama

## Overview

- **Разработчик:** Meta AI Research
- **Флагман:** Llama 3.3 70B (баланс), Llama 3.1 405B (максимум)
- **Ключевая особенность:** open weights — полный контроль, без API dependency

Meta открыла веса моделей Llama — это изменило рынок. Llama 3.x — production-ready open-source модели, которые можно деплоить локально, файнтюнить и встраивать в продукты без ограничений лицензии OpenAI/Anthropic. Стандарт для self-hosted агентов.

## Specifications

| Параметр | Значение |
|----------|---------|
| Лицензия | Llama Community License (коммерческое использование разрешено) |
| Контекст | 128k токенов (3.x серия) |
| Tool use | поддерживается в instruct-версиях |
| Quantization | GGUF (llama.cpp), GPTQ, AWQ |
| Llama Stack | официальный фреймворк Meta для агентов |

## Models

| Модель | Параметры | Роль в агентной системе | VRAM |
|--------|-----------|-------------------------|------|
| Llama 3.3 70B | 70B | Основная модель: баланс качества и скорости | ~40 GB |
| Llama 3.1 405B | 405B | Максимальное качество | ~240 GB |
| Llama 3.2 11B Vision | 11B | Лёгкий vision-агент | ~8 GB |
| Llama 3.2 3B / 1B | 1-3B | Edge-агенты, мобильные устройства | ~2-4 GB |

## Agent Use Cases

- **Self-hosted агент** — данные не покидают инфраструктуру, GDPR и enterprise privacy
- **Fine-tuned специализация** — файнтюнинг под конкретную domain: юридические, медицинские, технические задачи
- **High-volume pipeline** — при больших объёмах inference себестоимость ниже API
- **Edge-агенты** — 1-3B модели работают на мобильных устройствах и IoT
- **Local coding-агент** — Llama 3.3 70B с Ollama как альтернатива Claude Code для privacy-sensitive проектов

## Strengths for Agents

- **Open weights** — полный контроль: fine-tuning, quantization, локальный деплой
- **Privacy** — данные не покидают инфраструктуру, критично для enterprise
- **Cost** — себестоимость inference при достаточных объёмах ниже API-провайдеров
- **Llama Stack** — официальный фреймворк Meta для агентных систем на базе Llama
- **Ollama / vLLM** — зрелые инструменты локального запуска

## Setup

```bash
# Ollama — самый простой способ запустить локально
curl -fsSL https://ollama.com/install.sh | sh
ollama run llama3.3
```

```python
# Через Ollama OpenAI-совместимый API
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama"
)

response = client.chat.completions.create(
    model="llama3.3",
    messages=[{"role": "user", "content": "Hello"}]
)
```

```bash
# vLLM — production inference с высокой пропускной способностью
pip install vllm
python -m vllm.entrypoints.openai.api_server \
    --model meta-llama/Llama-3.3-70B-Instruct \
    --tensor-parallel-size 2
```

## Considerations

- Требует собственной GPU-инфраструктуры для production
- Tool use качественнее с fine-tuned версиями (Groq, Fireworks, Together AI)
- 70B модель хорошо работает на 2× A100 или аналогах
- Llama 3.1 405B требует серьёзного кластера или offloading на CPU

## See Also

- [DeepSeek (альтернативный open-source)](../deepseek/deepseek.md)
- [AI — обзор моделей](../ai.md)
