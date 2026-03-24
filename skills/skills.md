---
type: nav
tags: [skills, slash-commands, claude-code, automation, prompts]
status: complete
updated: 2026-03-24
---

# Skills — Slash Commands

## Overview

Skills — кастомные slash-команды (`/commit`, `/review-pr` и т.д.), расширяющие возможности Claude Code. Файлы с промптами, которые превращают типовые задачи в переиспользуемые мини-агенты. Поддерживают аргументы, bash injection, subagent execution.

## Structure

```
skills/
├── skills.md           # Этот файл — навигация по разделу
├── anatomy.md          # Структура skill: SKILL.md, frontmatter, аргументы, subagent
├── examples.md         # Готовые примеры: commit, review, refactor, deploy, security
└── builtin.md          # Встроенные команды: /help, /compact, /init, /batch, /loop и др.
```

## Topics

### Готово

- **[Anatomy](anatomy.md)** — структура SKILL.md, все frontmatter поля, аргументы ($ARGUMENTS), bash injection, subagent execution (context: fork)
- **[Examples](examples.md)** — готовые skills: commit, review-pr, fix-issue, refactor, docs, deploy, security-review, и другие

### Встроенные skills Claude Code

| Команда | Назначение |
|---------|-----------|
| `/batch <instruction>` | Масштабные изменения кодовой базы параллельными агентами |
| `/debug [description]` | Диагностика текущей сессии Claude Code |
| `/loop [interval] <prompt>` | Периодическое выполнение промпта |
| `/simplify [focus]` | Ревью изменённого кода на качество и эффективность |

- **[Built-in Commands](builtin.md)** — полный справочник встроенных команд: CLI (`/help`, `/compact`, `/init`, `/cost`) и агентных (`/batch`, `/loop`, `/simplify`, `/debug`)
