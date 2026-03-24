---
type: reference
tags: [cli-agents, amp, sourcegraph, multi-repo, codebase, enterprise]
status: complete
updated: 2026-03-24
related:
  - ai/cli-agents/cli-agents.md
---

# Amp (Sourcegraph)

## Overview

CLI-агент от Sourcegraph, авторов Cody. Фокус — работа с большими и сложными кодовыми базами, включая multi-repo окружения. Использует Sourcegraph-контекст: понимает не только открытые файлы, но и весь граф зависимостей проекта.

- **Вендор:** Sourcegraph
- **Модель:** Claude Sonnet (по умолчанию), GPT-4o
- **Open Source:** нет (закрытый бета)
- **Сайт:** sourcegraph.com/amp

## Key Features

- **Codebase context** — доступ ко всему кодовому графу через Sourcegraph: символы, определения, ссылки
- **Multi-repo** — агент понимает зависимости между несколькими репозиториями одновременно
- **Sourcegraph search** — точный семантический и regexp поиск по всему org
- **Code navigation** — go-to-definition, find-references на уровне всего монорепо
- **Terminal + editor** — редактирование файлов и выполнение команд
- **Threads** — именованные треды задач с сохранением контекста

## Use Cases

- Задачи в крупных монорепо или multi-repo организациях
- Рефакторинги, затрагивающие интерфейсы между несколькими сервисами
- Поиск всех мест использования API перед его изменением
- Команды, уже использующие Sourcegraph для навигации по коду

## Setup

```bash
npm install -g @sourcegraph/amp
amp
```

## Considerations

- Максимальная ценность только при наличии Sourcegraph-инстанса
- Закрытая бета — ограниченный доступ
- Менее универсален без Sourcegraph-контекста
- Нет hooks, MCP и других агентных примитивов уровня Claude Code
