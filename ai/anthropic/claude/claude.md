---
type: nav
tags: [anthropic, claude, claude-code, hooks, memory, mcp, permissions, skills]
status: stub
updated: 2026-03-24
related:
  - ai/anthropic/anthropic.md
  - ai/cli-agents/claude-code/claude-code.md
  - mcp/mcp.md
  - skills/skills.md
---

# Claude & Claude Code

## Overview

Всё о Claude Code — CLI-агенте от Anthropic. Фичи, конфигурация, паттерны использования, отличия от конкурентов. Цель раздела — понять, как максимально эффективно использовать Claude Code и какие его возможности стоит закладывать в архитектуру собственных агентов.

## Structure

```
claude/
├── claude.md           # Этот файл — навигация по разделу
└── ...                 # Будущие документы по конкретным темам
```

## Topics

- **Key Features** — агентная архитектура, hooks, CLAUDE.md, MCP, skills, worktree, память
- **Hooks** — автоматизация через события: PreToolUse, PostToolUse, Stop и др.
- **CLAUDE.md** — персистентный системный промпт проекта, иерархия файлов
- **Permissions** — модель разрешений, уровни автоматизации, безопасность
- **Memory** — файловая система памяти: типы (user, feedback, project, reference)
- **Sub-agents** — запуск специализированных агентов через Agent tool
- **MCP Integration** — подключение MCP-серверов к Claude Code
