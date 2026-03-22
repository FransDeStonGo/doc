# OpenAI

## Overview

OpenAI — пионер рынка LLM, создатели GPT-серии и o-серии (reasoning models). Широкая экосистема: API, Assistants API, функции, structured outputs. Де-факто стандарт совместимости для большинства агентных фреймворков.

## Models

| Модель | Контекст | Роль в агентной системе |
|--------|----------|-------------------------|
| GPT-4o | 128k | Универсальный агент: текст + vision, быстрый tool use |
| GPT-4o mini | 128k | Лёгкие подзадачи, дешёвые вызовы в pipeline |
| o3 | 200k | Глубокое планирование, математика, сложная логика |
| o4-mini | 128k | Reasoning по разумной цене, быстрее o3 |

## Strengths for Agents

- **Structured outputs** — гарантированный JSON по схеме, критично для agent pipelines
- **Assistants API** — встроенные threads, file search, code interpreter
- **Function calling** — зрелый, стабильный, широко поддерживается во фреймворках
- **o-серия** — модели с "думанием": лучше справляются с многошаговым планированием
- **Ecosystem** — LangChain, AutoGPT, CrewAI, OpenAI Swarm — всё заточено под OpenAI API

## Considerations

- Контекстное окно меньше чем у Claude (128k vs 200k)
- o-модели медленнее из-за chain-of-thought на стороне сервера
- Цена на o3 высокая для высоконагруженных агентных систем
