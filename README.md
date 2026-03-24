# AI Agent Knowledge Base

Личная база знаний по архитектуре, принципам и инструментам построения AI-агентов. Покрывает всё от теоретических основ до практических паттернов и настройки инфраструктуры.

---

## Содержание

### 🤖 Agents — Архитектура агентов

Концептуальная основа: что такое агент, как он думает, действует и взаимодействует с другими агентами.

| Документ | Описание |
|----------|----------|
| [what-is-agent.md](agents/what-is-agent.md) | Агент vs workflow, Augmented LLM, когда что выбирать |
| [workflows.md](agents/workflows.md) | 5 паттернов workflow по Anthropic (Chaining, Routing, Parallelization, Orchestrator, Evaluator) |
| [tool-use.md](agents/tool-use.md) | Механизм tool use: структура инструмента, цикл вызова, параллельные вызовы |
| [memory.md](agents/memory.md) | 4 типа памяти: in-context, external, in-weights, in-cache |
| [planning.md](agents/planning.md) | ReAct, Chain-of-Thought, Extended Thinking, Interleaved Thinking |
| [multi-agent.md](agents/multi-agent.md) | Оркестратор + воркеры, изоляция контекста, паттерны коммуникации |
| [permissions.md](agents/permissions.md) | Least privilege, blast radius, human-in-the-loop, stopping conditions |

---

### 🔌 MCP — Model Context Protocol

Открытый стандарт подключения AI-агентов к внешним инструментам и данным.

| Документ | Описание |
|----------|----------|
| [protocol.md](mcp/protocol.md) | Архитектура: host/client/server, транспорты (stdio/HTTP), JSON-RPC lifecycle |
| [servers.md](mcp/servers.md) | Каталог серверов: Filesystem, GitHub, Slack, PostgreSQL, Playwright и др. |
| [configuration.md](mcp/configuration.md) | Настройка в Claude Code: `settings.json`, stdio и HTTP серверы, troubleshooting |

---

### ☁️ Providers — Провайдеры API

Точки доступа к AI-моделям: от агрегаторов до оригинальных поставщиков.

| Документ | Описание |
|----------|----------|
| [openrouter.md](providers/openrouter.md) | Единый API для 300+ моделей, маршрутизация, fallback, мониторинг стоимости |
| [anthropic.md](providers/anthropic.md) | Anthropic API: Batch API (−50%), Prompt Caching, Extended Thinking |
| [openai.md](providers/openai.md) | OpenAI API: GPT-4o, o-серия, Structured Outputs, Assistants API |

---

### ⚡ Skills — Slash Commands

Кастомные команды для Claude Code: `/commit`, `/review-pr`, `/deploy` и любые другие.

| Документ | Описание |
|----------|----------|
| [anatomy.md](skills/anatomy.md) | Структура SKILL.md: frontmatter, аргументы, bash injection, subagent execution |
| [examples.md](skills/examples.md) | Готовые примеры: commit, review-pr, refactor, deploy, security-review |

---

### 🧠 AI — Модели и CLI-агенты

| Документ | Описание |
|----------|----------|
| [ai/ai.md](ai/ai.md) | Сравнительная таблица моделей и гайд по выбору |
| [cli-agents/cli-agents.md](ai/cli-agents/cli-agents.md) | Сводная таблица 14 CLI-агентов: Claude Code, Aider, Goose, Plandex и др. |

---

### 🗂️ Архивариус

Два AI-агента для работы с базой: лёгкий (поиск/чтение) и полный (чтение + запись + обслуживание).

| Документ | Описание |
|----------|----------|
| [archivist/haiku.md](archivist/haiku.md) | Системный промпт архивариуса на Haiku — только чтение |
| [archivist/sonnet.md](archivist/sonnet.md) | Системный промпт архивариуса на Sonnet — полный доступ |

---

## Структура проекта

```
doc/
├── INDEX.md              # Плоский индекс всех документов (для AI-навигации)
├── CONVENTIONS.md        # Стандарты базы: типология, frontmatter, шаблоны
├── agents/               # Архитектура агентов
├── ai/                   # Модели и CLI-агенты по производителям
├── archivist/            # Системные промпты агентов-архивариусов
├── mcp/                  # Model Context Protocol
├── providers/            # Провайдеры API
└── skills/               # Кастомные slash-команды
```

Каждый документ содержит YAML frontmatter с типом, тегами, статусом и связанными файлами — для навигации как людьми, так и AI-агентами.

---

## Стандарты

База написана по правилам из [CONVENTIONS.md](CONVENTIONS.md):

- **5 типов документов:** `nav`, `concept`, `reference`, `comparison`, `guide`
- **3 статуса:** `stub` → `draft` → `complete`
- **YAML frontmatter** в каждом файле: `type`, `tags`, `status`, `updated`, `related`
- **INDEX.md** — машиночитаемый индекс для быстрой навигации архивариуса
