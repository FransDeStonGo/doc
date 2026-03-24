---
type: nav
tags: [mcp, protocol, servers, tools, integration, claude-code]
status: draft
updated: 2026-03-24
---

# MCP — Model Context Protocol

## Overview

Model Context Protocol — открытый стандарт для подключения AI-агентов к внешним инструментам и источникам данных. Раздел охватывает протокол, примеры серверов, паттерны интеграции и практику настройки в Claude Code.

## Structure

```
mcp/
├── mcp.md              # Этот файл — навигация по разделу
├── protocol.md         # Архитектура MCP: host/client/server, primitives, lifecycle
├── servers.md          # Каталог серверов: референс, официальные, комьюнити
└── configuration.md    # Настройка в Claude Code: settings.json, stdio, HTTP
```

## Topics

### Готово
- **[Protocol](protocol.md)** — архитектура: host/client/server, транспорты (stdio/HTTP), primitives (tools/resources/prompts), JSON-RPC lifecycle
- **[Servers](servers.md)** — каталог готовых серверов: референс от MCP-команды, официальные интеграции, комьюнити
- **[Configuration](configuration.md)** — подключение серверов к Claude Code через `settings.json`

### В планах
- **Custom Servers** — как написать собственный MCP-сервер на Python/TypeScript
