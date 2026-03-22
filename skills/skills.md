# Skills — Slash Commands

## Overview

Skills — это кастомные slash-команды (`/commit`, `/review-pr` и т.д.), расширяющие возможности Claude Code. По сути, файлы с промптами, которые превращают типовые задачи в переиспользуемые мини-агенты. Раздел охватывает создание, структуру и примеры.

## Structure

```
skills/
├── skills.md           # Этот файл — навигация по разделу
└── ...                 # Будущие документы: шаблоны, примеры конкретных skills
```

## Topics

- **Anatomy of a Skill** — структура файла: frontmatter, промпт, аргументы
- **Built-in Skills** — обзор встроенных команд Claude Code
- **Custom Skills** — как написать свой skill под задачи команды
- **Patterns** — паттерны: review, generate, refactor, deploy, report
- **Examples** — готовые примеры skills для типовых задач
