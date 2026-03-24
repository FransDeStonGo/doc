---
type: nav
tags: [archivist, agents, search, knowledge-base, meta]
status: complete
updated: 2026-03-24
---

# Archivist — Агенты-архивариусы

## Overview

Два агента для работы с базой знаний. Различаются по мощности и задачам.

## Structure

```
archivist/
├── archivist.md        # Этот файл
├── haiku.md            # Системный промпт лёгкого архивариуса (Haiku)
└── sonnet.md           # Системный промпт полного архивариуса (Sonnet)
```

Skill-файлы для запуска через `/`:
```
.claude/commands/
├── search.md           # /search — поиск по базе (Haiku)
└── archivist.md        # /archivist — полный архивариус (Sonnet)
```

## Agents

| Агент | Модель | Назначение | Файл промпта |
|-------|--------|-----------|-------------|
| Search | Haiku 4.5 | Поиск, простые запросы, сводки | `archivist/haiku.md` |
| Archivist | Sonnet 4.5+ | Поиск + запись + обслуживание базы | `archivist/sonnet.md` |

## When to Use Which

- **`/search`** — найти информацию, ответить на вопрос по базе, сравнить варианты
- **`/archivist`** — добавить документ, обновить существующий, исправить структуру, поддержать INDEX.md

## See Also

- [CONVENTIONS.md](../CONVENTIONS.md)
- [INDEX.md](../INDEX.md)
