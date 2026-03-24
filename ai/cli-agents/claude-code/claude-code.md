---
type: reference
tags: [cli-agents, claude-code, anthropic, hooks, mcp, sub-agents, permissions, memory]
status: complete
updated: 2026-03-24
related:
  - ai/anthropic/claude/claude.md
  - ai/cli-agents/cli-agents.md
  - mcp/mcp.md
  - skills/skills.md
---

# Claude Code

## Overview

CLI-агент от Anthropic, построенный поверх моделей Claude. Работает прямо в терминале, глубоко интегрируется с кодовой базой и файловой системой. Главное отличие — продуманная агентная архитектура: hooks, sub-agents, MCP, система разрешений.

- **Вендор:** Anthropic
- **Модель:** Claude Sonnet 4.6 (по умолчанию), Opus 4.6, Haiku 4.5
- **Open Source:** нет
- **Лицензия:** коммерческая, оплата по токенам

## Key Features

- **Sub-agents** — запуск специализированных агентов внутри сессии (Explore, Plan, general-purpose и др.)
- **Hooks** — shell-команды, выполняемые до/после любого инструмента (PreToolUse, PostToolUse, Stop)
- **MCP** — нативная поддержка Model Context Protocol: подключение любых внешних инструментов
- **CLAUDE.md** — персистентный системный промпт для проекта, читается при каждом старте
- **Skills** — кастомные slash-команды (`/commit`, `/review-pr`), легко писать свои
- **Permissions** — гранулярная система: авто-разрешить / спросить / запретить для каждого инструмента
- **Worktree isolation** — агент работает в изолированном git worktree, не трогая рабочий код
- **Memory** — файловая система памяти между сессиями: user, feedback, project, reference

## Use Cases

- Повседневная разработка: написание, рефакторинг, объяснение кода
- Автоматизация рутины через hooks (форматирование, линтинг, тесты)
- Подключение корпоративных инструментов через MCP-серверы
- Multi-agent пайплайны с делегированием задач sub-агентам

## Setup

```bash
npm install -g @anthropic-ai/claude-code
claude
```

## Considerations

- Требует API-ключ Anthropic, оплата по токенам
- Нет локального запуска — всё через Anthropic API
- Закрытый исходный код

> Подробно: `ai/anthropic/claude/claude.md`
