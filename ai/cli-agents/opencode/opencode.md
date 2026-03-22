# opencode

## Overview

Open-source CLI-агент с терминальным UI (TUI), написанный на Go. Позиционируется как альтернатива Claude Code с полной открытостью исходников. Поддерживает множество провайдеров и моделей через единый интерфейс, работает прямо в терминале без браузера.

- **Вендор:** Community (sst.dev)
- **Модель:** любая — Claude, GPT-4o, Gemini, Llama (через OpenRouter и др.)
- **Open Source:** ✓ MIT
- **Сайт:** github.com/sst/opencode

## Key Features

- **TUI-интерфейс** — полноценный терминальный UI с панелями, историей, подсветкой синтаксиса
- **Multi-provider** — переключение между Claude, OpenAI, Gemini, Groq, Ollama без смены инструмента
- **LSP-интеграция** — использует Language Server Protocol для точного понимания кода: go-to-definition, диагностика
- **Session management** — именованные сессии, история диалогов
- **Tool use** — чтение/запись файлов, выполнение bash-команд, поиск по коду
- **Конфиг-файл** — `~/.config/opencode/config.json` для настройки провайдеров и поведения

## Use Cases

- Разработчики, которым важен open-source без vendor lock-in
- Работа с несколькими моделями в одном инструменте
- Среды, где нельзя использовать коммерческий Claude Code (лицензионные ограничения)
- Кастомизация агентного поведения на уровне исходного кода

## Setup

```bash
curl -fsSL https://opencode.ai/install | bash
# или
go install github.com/sst/opencode@latest
opencode
```

## Considerations

- Молодой проект, активно развивается — API нестабильный
- Нет аналогов hooks, MCP, sub-agents как в Claude Code
- Нет системы памяти между сессиями
- LSP-интеграция требует настройки language server для каждого языка
