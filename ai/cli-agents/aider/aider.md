---
type: reference
tags: [cli-agents, aider, open-source, git, multi-model, pair-programming]
status: complete
updated: 2026-03-24
related:
  - ai/cli-agents/cli-agents.md
---

# Aider

## Overview

Open-source CLI pair programmer с глубокой интеграцией в git. Философия: каждое изменение — это коммит. Aider работает с любой LLM-моделью и позиционирует себя как "AI в вашем терминале, который понимает ваш репозиторий".

- **Вендор:** Community (Paul Gauthier)
- **Модель:** любая — GPT-4o, Claude Sonnet, Gemini, Llama (через litellm)
- **Open Source:** ✓ Apache 2.0
- **Сайт:** aider.chat

## Key Features

- **Git-native** — автоматически коммитит каждое изменение с осмысленным сообщением
- **Repo map** — строит карту репозитория (файлы, классы, функции) для контекста модели
- **Multi-model** — поддержка любой LLM через litellm, легко переключать
- **Architect mode** — отдельная модель для планирования, другая для написания кода
- **Voice mode** — голосовой ввод задач
- **Lint & test** — автоматический запуск линтера и тестов после изменений, авто-фикс ошибок
- **Watch mode** — следит за комментариями `# AI: ...` в коде и выполняет их

## Use Cases

- Рефакторинг больших файлов с сохранением git-истории
- Парное программирование с чётким контролем изменений через git
- Работа в проектах, где важна воспроизводимость (каждый шаг — коммит)
- Использование дешёвых/локальных моделей для кодирования

## Setup

```bash
pip install aider-install
aider-install
aider --model claude-sonnet-4-6
```

## Considerations

- Нет встроенной системы памяти между сессиями
- Нет hooks/автоматизации уровня Claude Code
- Repo map может быть большим для крупных монорепо
- Нет браузера или выполнения произвольных команд (только bash через `/run`)
