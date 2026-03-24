---
type: reference
tags: [skills, slash-commands, claude-code, builtin, help, compact, init, batch, debug, loop]
status: complete
updated: 2026-03-24
related:
  - skills/skills.md
  - skills/anatomy.md
  - skills/examples.md
---

# Встроенные команды Claude Code

## Overview

- **Источник:** встроены в Claude Code, не требуют установки
- **Вызов:** `/команда [аргументы]` в prompt
- **Тип:** два вида — управляющие (изменяют состояние сессии) и агентные (запускают Claude на задачу)

Встроенные команды делятся на:
1. **CLI-команды** — управление сессией, памятью, настройками (не вызывают LLM)
2. **Агентные skills** — запускают Claude на конкретную задачу (`/batch`, `/debug`, `/loop`, `/simplify`)

---

## CLI-команды (управление сессией)

### `/help`

Показывает список доступных команд и краткую справку.

```
/help
/help batch
```

---

### `/clear`

Очищает историю разговора. Контекст сбрасывается, токены освобождаются. Полезно при смене задачи.

```
/clear
```

---

### `/compact [instructions]`

Сжимает историю разговора в краткое резюме, сохраняя контекст задачи. Снижает потребление токенов при длинных сессиях.

```
/compact
/compact focus on the authentication refactoring task
```

Без аргументов — Claude сам выбирает, что важно сохранить.

---

### `/init`

Создаёт `CLAUDE.md` для текущего проекта — анализирует кодовую базу и генерирует персистентный системный промпт с описанием проекта, стека, конвенций.

```
/init
```

Запускается один раз при онбординге на новый проект. Результат — файл `.claude/CLAUDE.md` (или `CLAUDE.md` в корне).

---

### `/cost`

Показывает стоимость текущей сессии: токены, деньги, разбивка по запросам.

```
/cost
```

---

### `/status`

Показывает состояние текущей сессии: модель, контекстное окно, подключённые MCP-серверы.

```
/status
```

---

### `/mcp`

Список подключённых MCP-серверов и доступных через них инструментов.

```
/mcp
```

---

### `/memory`

Управление файловой системой памяти Claude Code.

```
/memory          # показать текущую память
/memory clear    # очистить память (с подтверждением)
```

---

### `/config`

Открывает интерактивную настройку Claude Code: модель, тема, режимы.

```
/config
```

---

### `/doctor`

Диагностика установки Claude Code: API-ключ, версия, конфигурация, подключение.

```
/doctor
```

---

### `/vim`

Переключение в vim-режим ввода для редактирования многострочных промптов.

```
/vim
```

---

### `/fast`

Переключает Fast Mode — более быстрые ответы (тот же Opus, оптимизированный вывод).

```
/fast
```

---

## Агентные skills

Эти команды запускают Claude на конкретную задачу. Реализованы как встроенные skills.

---

### `/batch <instruction>`

Масштабные изменения в кодовой базе с помощью параллельных агентов. Claude разбивает задачу на части и запускает несколько sub-агентов одновременно.

```
/batch Add JSDoc comments to all exported functions in src/
/batch Rename all instances of 'userId' to 'user_id' following snake_case convention
/batch Add error handling to all async functions that don't have try/catch
```

**Когда использовать:** задача затрагивает много файлов, изменения однотипные.

---

### `/debug [description]`

Диагностика текущей сессии Claude Code: что пошло не так, почему агент застрял, анализ контекста.

```
/debug
/debug the agent is repeatedly trying to edit the same file
/debug why did the last commit fail
```

---

### `/loop [interval] <prompt>`

Периодически выполняет prompt на заданном интервале. Полезно для мониторинга, polling, автоматизации.

```
/loop 5m check if the build is passing and report status
/loop 10m summarize new GitHub issues and PRs
/loop 1h update the project status report
```

Интервал по умолчанию: 10 минут. Форматы: `30s`, `5m`, `1h`.

---

### `/simplify [focus]`

Ревью кода, который был изменён в текущей сессии: анализирует на предмет излишней сложности, дублирования, возможностей упростить.

```
/simplify
/simplify focus on the authentication module
/simplify check for DRY violations
```

**Важно:** анализирует только изменённый код (diff), не всю кодовую базу.

---

### `/review [target]`

Code review: анализ кода на баги, стиль, best practices, потенциальные проблемы.

```
/review
/review src/auth/
/review the last commit
```

---

### `/schedule <cron> <prompt>`

Создаёт запланированный запуск агента по cron-расписанию (remote trigger).

```
/schedule "0 9 * * 1" weekly standup summary from git log
/schedule "0 */6 * * *" check for security advisories in dependencies
```

---

## Кастомные slash-команды

Помимо встроенных, доступны:
- **Пользовательские skills** из `~/.claude/skills/` — личные, для всех проектов
- **Проектные skills** из `.claude/skills/` — для текущего репозитория
- **Команды** из `.claude/commands/` — старый формат, один `.md` файл

```
/my-custom-skill argument1 argument2
```

Как создавать: [anatomy.md](anatomy.md), готовые примеры: [examples.md](examples.md).

---

## Quick Reference

| Команда | Тип | Назначение |
|---------|-----|-----------|
| `/help` | CLI | Справка по командам |
| `/clear` | CLI | Сбросить историю разговора |
| `/compact` | CLI | Сжать историю, сохранив контекст |
| `/init` | CLI | Создать CLAUDE.md для проекта |
| `/cost` | CLI | Стоимость текущей сессии |
| `/status` | CLI | Состояние сессии и MCP |
| `/mcp` | CLI | Список MCP-серверов |
| `/memory` | CLI | Управление памятью |
| `/config` | CLI | Настройки Claude Code |
| `/doctor` | CLI | Диагностика установки |
| `/vim` | CLI | Vim-режим ввода |
| `/fast` | CLI | Быстрый режим |
| `/batch` | Skill | Параллельные изменения в кодовой базе |
| `/debug` | Skill | Диагностика сессии |
| `/loop` | Skill | Периодическое выполнение |
| `/simplify` | Skill | Ревью изменённого кода |
| `/review` | Skill | Code review |
| `/schedule` | Skill | Запланированный запуск |

## See Also

- [Anatomy — как создать свой skill](anatomy.md)
- [Examples — готовые примеры skills](examples.md)
- [Claude Code (обзор CLI-агента)](../ai/cli-agents/claude-code/claude-code.md)
