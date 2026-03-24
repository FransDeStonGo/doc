---
type: reference
tags: [cli-agents, goose, block, open-source, mcp, toolkits, extensible]
status: complete
updated: 2026-03-24
related:
  - ai/cli-agents/cli-agents.md
  - mcp/mcp.md
---

# Goose

## Overview

Open-source CLI-агент от Block (бывший Square). Расширяемая архитектура на основе toolkits — наборов инструментов, которые можно подключать и отключать. Поддерживает MCP из коробки. Позиционируется как "extensible AI agent" без привязки к конкретной модели.

- **Вендор:** Block (Square)
- **Модель:** любая — OpenAI, Anthropic, Google, Ollama (локальные)
- **Open Source:** ✓ Apache 2.0
- **Сайт:** block.github.io/goose

## Key Features

- **Toolkits** — модульные наборы инструментов: developer, github, jira, google drive, screen и др.
- **MCP-поддержка** — подключение MCP-серверов как источников инструментов
- **Provider-agnostic** — работает с любым LLM-провайдером через единый интерфейс
- **Sessions** — именованные сессии с сохранением контекста
- **Extensions** — пишешь свои расширения на Python или через MCP
- **Screen toolkit** — может делать скриншоты и взаимодействовать с UI
- **Goose Desktop** — опциональный GUI поверх CLI

## Use Cases

- Автоматизация рабочих процессов с интеграцией корпоративных инструментов (Jira, GitHub, Drive)
- Построение кастомных агентных пайплайнов через toolkits
- Команды, которые хотят self-hosted агента без зависимости от одного вендора
- Исследование и прототипирование агентных систем

## Setup

```bash
pip install goose-ai
goose session start
```

## Considerations

- Менее зрелый чем Claude Code по агентным возможностям
- Документация и комьюнити меньше
- Screen toolkit работает нестабильно на headless серверах
