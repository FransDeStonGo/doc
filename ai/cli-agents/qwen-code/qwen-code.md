---
type: reference
tags: [cli-agents, qwen-code, alibaba, open-source, claude-code-fork, self-hosted]
status: complete
updated: 2026-03-24
related:
  - ai/qwen/qwen.md
  - ai/cli-agents/claude-code/claude-code.md
  - ai/cli-agents/cli-agents.md
---

# Qwen Code

## Overview

CLI-агент от Alibaba на базе моделей Qwen2.5-Coder. Форк Claude Code с заменой модели на Qwen — сохраняет знакомый UX и архитектуру (инструменты, CLAUDE.md, сессии), но работает через Alibaba Cloud API или локально. Один из немногих Claude Code-совместимых агентов от крупного вендора.

- **Вендор:** Alibaba Cloud
- **Модель:** Qwen2.5-Coder-32B-Instruct, Qwen2.5-72B-Instruct
- **Open Source:** ✓ (форк Claude Code)
- **Сайт:** github.com/QwenLM/qwen-code

## Key Features

- **Claude Code-совместимость** — те же команды, инструменты, паттерны работы что и в Claude Code
- **Qwen2.5-Coder** — специализированная code-модель, одна из лучших open-source для кода
- **Локальный деплой** — запуск через Ollama / vLLM с открытыми весами
- **Alibaba Cloud API** — облачный вариант через DashScope
- **Tool use** — полный набор инструментов Claude Code: файлы, bash, поиск
- **CLAUDE.md-совместимость** — тот же механизм персистентного контекста проекта

## Use Cases

- Знакомый Claude Code UX с open-source моделью
- Self-hosted деплой в enterprise без зависимости от Anthropic API
- Регионы с ограниченным доступом к Anthropic/OpenAI API
- Команды, уже использующие Qwen-модели в своём стеке

## Setup

```bash
npm install -g qwen-code
# Через Alibaba Cloud DashScope
export DASHSCOPE_API_KEY=your_key
qwen

# Или локально через Ollama
ollama pull qwen2.5-coder:32b
OPENAI_BASE_URL=http://localhost:11434/v1 qwen
```

## Considerations

- Качество уступает Claude Sonnet на сложных агентных задачах
- DashScope API менее зрелый чем Anthropic/OpenAI по инфраструктуре
- Часть фич Claude Code может отсутствовать или работать иначе (MCP, sub-agents)
- Китайская инфраструктура — те же соображения по privacy что и у DeepSeek/Qwen
