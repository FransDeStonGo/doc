---
type: reference
tags: [cli-agents, codex, openai, open-source, sandbox, composable]
status: complete
updated: 2026-03-24
related:
  - ai/openai/openai.md
  - ai/cli-agents/cli-agents.md
---

# Codex CLI (OpenAI)

## Overview

Open-source CLI-агент от OpenAI — лёгкий, composable, с упором на безопасность. Работает в sandboxed-окружении: сетевой доступ и запись файлов ограничены по умолчанию. Позиционируется как "coding agent for the terminal" с простой архитектурой.

- **Вендор:** OpenAI
- **Модель:** GPT-4o, o3, o4-mini (настраивается)
- **Open Source:** ✓ Apache 2.0
- **Сайт:** github.com/openai/codex

## Key Features

- **Sandbox by default** — выполнение кода в изолированном окружении, сеть заблокирована
- **Three approval modes** — `suggest` (только показывает), `auto-edit` (редактирует файлы), `full-auto` (всё автономно)
- **Multimodal input** — принимает скриншоты и изображения как контекст задачи
- **Composable** — легко встраивается в скрипты и CI/CD через stdin/stdout
- **Git-aware** — понимает состояние репозитория, работает с diff
- **Zero config** — работает из коробки без настройки конфигурационных файлов

## Use Cases

- Простые coding-задачи с минимальным setup
- Встраивание в CI/CD пайплайны и скрипты автоматизации
- Безопасное выполнение задач в sandbox без риска сломать окружение
- Изучение агентных подходов OpenAI

## Setup

```bash
npm install -g @openai/codex
export OPENAI_API_KEY=your_key
codex "напиши функцию сортировки на Python"
```

## Considerations

- Менее функциональный чем Claude Code (нет hooks, MCP, sub-agents, памяти)
- Молодой проект, активно развивается
- Sandbox ограничивает часть реальных задач
- Нет аналога CLAUDE.md для персистентного контекста проекта
