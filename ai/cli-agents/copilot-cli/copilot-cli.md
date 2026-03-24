---
type: reference
tags: [cli-agents, copilot, github, microsoft, shell, terminal]
status: complete
updated: 2026-03-24
related:
  - ai/cli-agents/cli-agents.md
---

# GitHub Copilot CLI

## Overview

CLI-расширение GitHub Copilot для работы в терминале. Помогает строить shell-команды, объяснять команды и автоматизировать терминальные задачи. Более узкоспециализированный инструмент по сравнению с полноценными агентами — фокус на shell, а не на коде.

- **Вендор:** GitHub / Microsoft
- **Модель:** GPT-4o
- **Open Source:** нет
- **Требования:** подписка GitHub Copilot

## Key Features

- **`gh copilot suggest`** — генерация shell-команды по описанию на естественном языке
- **`gh copilot explain`** — объяснение любой команды или её вывода
- **Shell integration** — интеграция с bash/zsh/fish: `??` вместо полной команды
- **Context-aware** — понимает текущую OS, shell, установленные инструменты
- **Safe execution** — показывает команду перед выполнением, спрашивает подтверждение

## Use Cases

- Быстрое построение сложных pipeline-команд (find, awk, sed, jq и др.)
- Объяснение незнакомых команд или скриптов
- Помощь с git-командами и флагами
- Новички в терминале, которым нужна подсказка

## Setup

```bash
gh extension install github/gh-copilot
gh copilot suggest "найди все файлы больше 100MB"
gh copilot explain "grep -rn --include='*.js' 'TODO' ."
```

## Considerations

- Не полноценный агент — не редактирует файлы, не выполняет многошаговые задачи
- Требует платной подписки GitHub Copilot
- Узкий фокус: только shell-команды, не общая разработка
- Слабее специализированных агентов для кода
