---
type: concept
tags: [mcp, protocol, architecture, tools, resources, prompts, transport, stdio, http, json-rpc]
status: complete
updated: 2026-03-24
related:
  - mcp/mcp.md
  - mcp/servers.md
  - mcp/configuration.md
  - agents/tool-use.md
---

# MCP — Model Context Protocol

## Overview

MCP (Model Context Protocol) — открытый стандарт для подключения AI-приложений к внешним системам. Как USB-C унифицирует подключение устройств, MCP унифицирует подключение агентов к данным и инструментам. Написан один раз — работает везде: Claude, ChatGPT, VS Code, Cursor и другие хосты поддерживают один и тот же протокол.

> Источник: modelcontextprotocol.io (2024), спецификация версии 2025-06-18

---

## Архитектура

MCP использует клиент-серверную модель с тремя участниками:

```
┌─────────────────────────────────────────┐
│           MCP Host (AI-приложение)       │
│   ┌──────────┐  ┌──────────┐            │
│   │ Client 1 │  │ Client 2 │  ...       │
│   └────┬─────┘  └────┬─────┘            │
└────────┼─────────────┼──────────────────┘
         │             │
    ┌────▼─────┐  ┌────▼──────────┐
    │ Server A │  │   Server B    │
    │ (local)  │  │  (remote)     │
    └──────────┘  └───────────────┘
```

| Участник | Роль |
|---------|------|
| **MCP Host** | AI-приложение (Claude Code, Claude Desktop, VS Code). Управляет клиентами и предоставляет LLM |
| **MCP Client** | Компонент внутри host'а. Один client = одно соединение с одним server'ом |
| **MCP Server** | Программа, предоставляющая инструменты, данные, промпты. Локальная или удалённая |

**Ключевое правило:** один client — один server. Если хост подключается к 3 серверам — создаётся 3 клиента.

---

## Транспорты

### Stdio (Standard I/O)

- Сервер запускается как дочерний процесс хоста
- Общение через stdin/stdout
- Только локальные серверы
- Нет накладных расходов сети — максимальная производительность
- Используется большинством локальных MCP-серверов

```json
// settings.json — stdio сервер
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/user"]
    }
  }
}
```

### Streamable HTTP

- Хост → сервер: HTTP POST запросы
- Сервер → хост: опциональный Server-Sent Events (streaming)
- Поддерживает удалённые серверы
- Аутентификация: bearer tokens, API keys, OAuth
- Один сервер обслуживает множество клиентов одновременно

```json
// settings.json — remote HTTP сервер
{
  "mcpServers": {
    "sentry": {
      "url": "https://mcp.sentry.io/mcp",
      "headers": {
        "Authorization": "Bearer ${SENTRY_TOKEN}"
      }
    }
  }
}
```

---

## Протокол данных (JSON-RPC 2.0)

MCP использует JSON-RPC 2.0 для всех сообщений.

### Lifecycle: инициализация соединения

При подключении клиент и сервер согласовывают возможности (capability negotiation):

```json
// 1. Клиент отправляет initialize
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2025-06-18",
    "capabilities": { "elicitation": {} },
    "clientInfo": { "name": "claude-code", "version": "1.0.0" }
  }
}

// 2. Сервер отвечает своими capabilities
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "2025-06-18",
    "capabilities": {
      "tools": { "listChanged": true },
      "resources": {}
    },
    "serverInfo": { "name": "my-server", "version": "1.0.0" }
  }
}

// 3. Клиент подтверждает готовность
{
  "jsonrpc": "2.0",
  "method": "notifications/initialized"
}
```

---

## Primitives — что сервер может предоставить

### Tools (Инструменты)

Исполняемые функции — агент вызывает их для действий. Аналог tool use в Anthropic API.

```json
// Обнаружение инструментов
{ "method": "tools/list" }

// Вызов инструмента
{
  "method": "tools/call",
  "params": {
    "name": "read_file",
    "arguments": { "path": "/home/user/notes.txt" }
  }
}
```

Схема инструмента в MCP (`inputSchema`) отличается от Anthropic API (`input_schema`) только именем поля — при конвертации нужно переименовать.

### Resources (Ресурсы)

Источники данных для контекста — файлы, записи БД, API-ответы. Не исполняются, только читаются.

```json
// Список ресурсов
{ "method": "resources/list" }

// Чтение ресурса
{ "method": "resources/read", "params": { "uri": "file:///home/user/notes.txt" } }
```

### Prompts (Промпты)

Переиспользуемые шаблоны для взаимодействия с LLM. Системные промпты, few-shot примеры.

```json
{ "method": "prompts/list" }
{ "method": "prompts/get", "params": { "name": "code_review" } }
```

### Primitives клиента

Серверы тоже могут запрашивать что-то у клиента:

| Primitive | Что делает |
|-----------|-----------|
| **Sampling** | Сервер просит LLM-ответ у клиента (остаётся model-agnostic) |
| **Elicitation** | Сервер запрашивает ввод у пользователя |
| **Logging** | Сервер отправляет логи клиенту |

---

## Notifications — динамические обновления

Сервер может уведомлять клиента об изменениях без запроса:

```json
// Сервер: список инструментов изменился
{ "jsonrpc": "2.0", "method": "notifications/tools/list_changed" }
```

Клиент реагирует обновлением списка инструментов. Это делает MCP-соединение живым — агент всегда знает актуальные возможности сервера.

---

## Сравнение с прямым Tool Use

| | Tool Use (Anthropic API) | MCP |
|--|:------------------------:|:---:|
| **Стандартизация** | Только Anthropic | Открытый стандарт |
| **Переиспользование** | Нет (каждый пишет свой) | Да (один сервер для всех хостов) |
| **Транспорт** | HTTP API | stdio или HTTP |
| **Discovery** | Нет (передаёшь вручную) | Да (tools/list) |
| **Resources** | Нет | Да |
| **Prompts** | Нет | Да |
| **Локальный сервер** | Нет | Да (stdio) |

**Когда MCP лучше прямого tool use:**
- Нужно подключить один инструментарий к нескольким AI-приложениям
- Инструменты уже есть как MCP-сервер (GitHub, Sentry, Slack)
- Нужны Resources или Prompts помимо инструментов

---

## See Also

- [MCP навигация](mcp.md) — обзор раздела
- [MCP серверы](servers.md) — готовые серверы для подключения
- [MCP конфигурация](configuration.md) — подключение к Claude Code
- [Tool Use](../agents/tool-use.md) — как MCP-инструменты используются агентом
