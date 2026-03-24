---
type: nav
tags: [anthropic, claude, claude-code, hooks, memory, mcp, permissions, skills, cli-agent]
status: complete
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
└── claude.md           # Этот файл — навигация по разделу
```

*Детальные документы по отдельным фичам — в планах.*

## Topics

### Уже задокументировано

- **[Claude Code (CLI-агент)](../../cli-agents/claude-code/claude-code.md)** — сравнительный обзор CLI-агента: возможности, установка, конкурентное позиционирование
- **[Skills (slash-команды)](../../../skills/skills.md)** — создание кастомных `/команд` для автоматизации задач
- **[MCP Integration](../../../mcp/configuration.md)** — подключение MCP-серверов к Claude Code

### Ключевые возможности Claude Code

| Возможность | Описание |
|-------------|---------|
| **CLAUDE.md** | Персистентный системный промпт проекта — иерархия файлов (global → project → local) |
| **Hooks** | Автоматизация через события: `PreToolUse`, `PostToolUse`, `Stop`, `Notification` |
| **Memory** | Файловая система памяти: типы `user`, `feedback`, `project`, `reference` |
| **Sub-agents** | Запуск специализированных агентов через Agent tool (Explore, Plan, general-purpose) |
| **Worktree** | Изолированный git worktree для агента — безопасные эксперименты |
| **MCP** | Подключение внешних инструментов через Model Context Protocol |
| **Skills** | Кастомные slash-команды: `/commit`, `/review-pr` и свои |
| **Plan Mode** | Режим планирования без выполнения — согласование подхода перед изменениями |

### В планах

- **Hooks** — полный справочник событий, синтаксис настройки, примеры автоматизации
- **CLAUDE.md** — иерархия, формат, best practices для проектов
- **Memory** — подробно о типах памяти и паттернах использования
- **Permissions** — модель разрешений, уровни автоматизации (`--dangerously-skip-permissions`)
- **Built-in Commands** — справочник встроенных команд `/help`, `/compact`, `/init`, `/batch`, `/debug`, `/loop`

## See Also

- [Anthropic (платформа)](../anthropic.md)
- [Claude Code CLI-агент (сравнение)](../../cli-agents/claude-code/claude-code.md)
- [Skills](../../../skills/skills.md)
- [MCP Configuration](../../../mcp/configuration.md)
