---
type: reference
tags: [cli-agents, gemini-cli, google, long-context, multimodal, open-source, mcp]
status: complete
updated: 2026-03-24
related:
  - ai/google/google.md
  - ai/cli-agents/cli-agents.md
---

# Gemini CLI

## Overview

Open-source CLI-агент от Google на базе Gemini 2.5 Pro. Выделяется рекордным контекстным окном в 1M токенов — можно загрузить весь репозиторий целиком. Поддерживает MCP и Google Search grounding из коробки.

- **Вендор:** Google
- **Модель:** Gemini 2.5 Pro (по умолчанию), Flash
- **Open Source:** ✓ Apache 2.0
- **Сайт:** github.com/google-gemini/gemini-cli

## Key Features

- **1M токенов контекста** — загружает огромные кодовые базы, документы, логи целиком
- **Google Search grounding** — нативный доступ к актуальному поиску Google прямо в агенте
- **MCP-поддержка** — подключение MCP-серверов как источников инструментов
- **Multimodal** — работает с изображениями, PDF, видео прямо в терминале
- **Free tier** — щедрый бесплатный лимит через Google AI Studio API key
- **`@`-синтаксис** — добавление файлов/папок в контекст командой `@path/to/file`
- **Shell tool** — выполнение bash-команд, редактирование файлов

## Use Cases

- Анализ огромных кодовых баз, которые не влезают в контекст других агентов
- Работа с мультимодальными данными (скриншоты, схемы, PDF-документация)
- Задачи, требующие актуальных данных из интернета (Google Search)
- Бесплатная альтернатива Claude Code для знакомства с AI-агентами

## Setup

```bash
npm install -g @google/gemini-cli
gemini
```

## Considerations

- Молодой проект (2025), менее зрелый чем Claude Code или Aider
- Качество tool use уступает Claude на сложных задачах
- Большой контекст не означает хорошее качество на краях окна
- Зависимость от Google-инфраструктуры
