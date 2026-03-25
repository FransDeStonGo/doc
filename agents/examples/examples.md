---
type: nav
tags: [agents, examples, code, practical]
status: complete
updated: 2026-03-25
related:
  - ../tool-use.md
  - ../memory.md
  - ../multi-agent.md
---

# Agent Examples

Практические примеры реализации AI-агентов. Каждый пример — рабочий код, готовый к запуску и модификации.

---

## Overview

Этот раздел содержит 4 типа агентов от простого к сложному:

1. **Simple Tool Agent** — базовый агент с несколькими инструментами
2. **RAG Agent** — агент с external memory (vector search)
3. **Multi-Agent Orchestrator** — система из оркестратора и специалистов
4. **MCP Agent** — агент с подключением к MCP-серверам

Все примеры используют Anthropic Claude API и написаны на Python.

---

## Structure

```
agents/examples/
├── examples.md                    # Этот файл — навигация
├── simple-tool-agent.md           # Базовый агент с tool use
├── rag-agent.md                   # Агент с Qdrant + embeddings
├── multi-agent-orchestrator.md    # Orchestrator + specialists
└── mcp-agent.md                   # Агент с MCP-серверами
```

---

## Examples

### 1. Simple Tool Agent
**Файл:** [simple-tool-agent.md](simple-tool-agent.md)  
**Сложность:** ⭐ Начальный  
**Инструменты:** weather, calculator, web_search  
**Строк кода:** ~100

Базовый агент, демонстрирующий механизм tool use. Умеет вызывать функции, обрабатывать результаты и формировать ответ.

**Use case:** Простой ассистент для информационных запросов.

---

### 2. RAG Agent
**Файл:** [rag-agent.md](rag-agent.md)  
**Сложность:** ⭐⭐ Средний  
**Инструменты:** Qdrant, OpenAI embeddings, semantic_search  
**Строк кода:** ~150

Агент с external memory. Индексирует документы в векторную БД, выполняет семантический поиск и отвечает на основе найденного контекста.

**Use case:** Агент для работы с базой знаний, документацией, FAQ.

---

### 3. Multi-Agent Orchestrator
**Файл:** [multi-agent-orchestrator.md](multi-agent-orchestrator.md)  
**Сложность:** ⭐⭐⭐ Продвинутый  
**Компоненты:** Orchestrator, Backend Specialist, Frontend Specialist  
**Строк кода:** ~200

Система из трёх агентов: оркестратор декомпозирует задачу, делегирует специалистам, собирает результаты.

**Use case:** Сложные задачи, требующие специализации (код-ревью, архитектурные решения).

---

### 4. MCP Agent
**Файл:** [mcp-agent.md](mcp-agent.md)  
**Сложность:** ⭐⭐ Средний  
**MCP серверы:** filesystem, github  
**Строк кода:** ~120

Агент, подключённый к MCP-серверам. Использует стандартизированный протокол для доступа к внешним инструментам.

**Use case:** Агент для работы с файловой системой, GitHub, базами данных через MCP.

---

## Prerequisites

Все примеры требуют:

```bash
pip install anthropic python-dotenv
```

Для RAG Agent дополнительно:
```bash
pip install qdrant-client openai
```

Для MCP Agent дополнительно:
```bash
npm install -g @modelcontextprotocol/server-filesystem
npm install -g @modelcontextprotocol/server-github
```

---

## Environment Variables

Создайте `.env` файл:

```bash
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...  # для embeddings в RAG Agent
GITHUB_TOKEN=ghp_...   # для MCP Agent с GitHub
```

---

## Quick Start

```bash
# 1. Клонируйте примеры
git clone https://github.com/FransDeStonGo/doc.git
cd doc/agents/examples

# 2. Установите зависимости
pip install -r requirements.txt

# 3. Настройте .env
cp .env.example .env
# Отредактируйте .env — добавьте API ключи

# 4. Запустите пример
python simple-tool-agent.py
```

---

## Learning Path

Рекомендуемый порядок изучения:

1. **Simple Tool Agent** — понять базовый цикл tool use
2. **MCP Agent** — научиться работать с MCP протоколом
3. **RAG Agent** — добавить external memory
4. **Multi-Agent Orchestrator** — освоить декомпозицию задач

---

## See Also

- [Tool Use](../tool-use.md) — теория механизма tool use
- [Memory](../memory.md) — типы памяти агентов
- [Multi-Agent](../multi-agent.md) — паттерны multi-agent систем
- [MCP Protocol](../../mcp/protocol.md) — Model Context Protocol
