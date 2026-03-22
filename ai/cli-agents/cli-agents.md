# CLI Agents

## Overview

Подборка CLI AI-агентов — инструментов, которые работают в терминале и автономно решают задачи: пишут и редактируют код, выполняют команды, работают с файловой системой и внешними сервисами.

Агенты различаются по модели, степени автономности, глубине интеграции с кодовой базой и философии работы.

## Structure

```
cli-agents/
├── cli-agents.md          # Этот файл
├── claude-code/           # Anthropic — агентная архитектура, MCP, hooks
├── aider/                 # Open-source — git-native pair programmer
├── goose/                 # Block — extensible, open-source, MCP
├── plandex/               # Open-source — долгосрочное планирование
├── openhands/             # Open-source — браузер + терминал + код
├── swe-agent/             # Princeton — автономный агент для GitHub issues
├── copilot-cli/           # GitHub/Microsoft — терминальные команды
├── gemini-cli/            # Google — большой контекст, мультимодальность
├── amazon-q/              # AWS — enterprise, AWS-экосистема
├── codex-cli/             # OpenAI — open-source, простой и composable
├── amp/                   # Sourcegraph — codebase-aware, multi-repo
├── opencode/              # Community — open-source TUI-агент с LSP, multi-provider
├── qwen-code/             # Alibaba — форк Claude Code на Qwen2.5-Coder
└── openclaw/              # ⚠️ Другая категория: gateway для агентов через мессенджеры
```

## Comparison

| Агент | Вендор | Модель | Open Source | Фокус | Ключевая фишка |
|-------|--------|--------|:-----------:|-------|----------------|
| Claude Code | Anthropic | Claude Sonnet/Opus | — | Универсальный | Hooks, MCP, sub-agents, permissions |
| Aider | Community | Any (GPT/Claude/Gemini) | ✓ | Код + git | Git-native: коммитит каждое изменение |
| Goose | Block | Any (OpenAI/Anthropic/...) | ✓ | Универсальный | Extensible toolkits, MCP-поддержка |
| Plandex | Community | GPT-4o / Claude | ✓ | Большие задачи | Версионирование изменений, откат |
| OpenHands | Community | Any | ✓ | Full-stack | Браузер + терминал + код в sandbox |
| SWE-agent | Princeton | GPT-4 / Claude | ✓ | GitHub issues | ACI-интерфейс, bash + editor |
| Copilot CLI | GitHub/MS | GPT-4o | — | Терминал | Объяснение и генерация shell-команд |
| Gemini CLI | Google | Gemini 2.5 Pro | ✓ | Универсальный | 1M контекст, Google Search grounding |
| Amazon Q | AWS | Amazon Titan/Claude | — | Enterprise/AWS | Интеграция с AWS сервисами |
| Codex CLI | OpenAI | GPT-4o / o3 | ✓ | Код | Простой, composable, sandboxed |
| Amp | Sourcegraph | Claude / GPT-4o | — | Большие кодовые базы | Multi-repo, Cody контекст |
| opencode | Community (sst) | Any | ✓ | Универсальный | TUI + LSP, multi-provider без vendor lock-in |
| OpenClaw | Community | Any | ✓ | Messaging gateway | Self-hosted шлюз: WhatsApp/Telegram/Discord → AI-агент |
| Qwen Code | Alibaba | Qwen2.5-Coder | ✓ | Код | Claude Code UX + open-source модель |

## Selection Guide

- **Нужен лучший агент для повседневной разработки** → Claude Code
- **Нужен git-native pair programmer с любой моделью** → Aider
- **Нужен open-source с расширяемыми инструментами** → Goose
- **Нужно решить большую задачу за много шагов** → Plandex
- **Нужна полная автономия: браузер + код + терминал** → OpenHands
- **Нужно автоматически закрывать GitHub issues** → SWE-agent
- **Нужна помощь с shell-командами в терминале** → Copilot CLI
- **Нужен огромный контекст или мультимодальность** → Gemini CLI
- **Работаете в AWS-экосистеме** → Amazon Q Developer
- **Нужен простой open-source агент от OpenAI** → Codex CLI
- **Нужна работа с большой multi-repo кодовой базой** → Amp
- **Нужен open-source агент с TUI и поддержкой любого провайдера** → opencode
- **Нужен AI-агент доступный через WhatsApp/Telegram/Discord** → OpenClaw
- **Нужен Claude Code UX с open-source моделью** → Qwen Code
