---
type: guide
tags: [mcp, custom-server, python, typescript, sdk, tools, development]
status: complete
updated: 2026-03-24
related:
  - mcp/protocol.md
  - mcp/configuration.md
  - mcp/servers.md
---

# Как написать собственный MCP-сервер

## Overview

Цель: создать кастомный MCP-сервер, который расширяет агента собственными инструментами. На выходе — работающий сервер, подключённый к Claude Code.

Когда нужен кастомный сервер:
- Интеграция с внутренними системами (CRM, ERP, internal APIs)
- Инструменты, специфичные для проекта (деплой, CI/CD, базы данных)
- Обёртка над существующими REST API

## Prerequisites

- Понимание [MCP primitives](protocol.md): Tools, Resources, Prompts
- Node.js 18+ (для TypeScript) или Python 3.10+ (для Python)
- Claude Code для тестирования

## Steps

### 1. Инициализировать проект

**TypeScript (рекомендуется для production):**

```bash
mkdir my-mcp-server && cd my-mcp-server
npm init -y
npm install @modelcontextprotocol/sdk
npm install -D typescript @types/node
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./dist",
    "strict": true
  },
  "include": ["src/**/*"]
}
```

**Python:**

```bash
pip install mcp
```

---

### 2. Реализовать сервер с одним tool

**TypeScript:**

```typescript
// src/index.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  { name: "my-server", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

// Описание инструментов
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "get_status",
      description: "Получить статус системы",
      inputSchema: {
        type: "object",
        properties: {
          service: {
            type: "string",
            description: "Название сервиса",
          },
        },
        required: ["service"],
      },
    },
  ],
}));

// Выполнение инструментов
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "get_status") {
    const { service } = request.params.arguments as { service: string };
    // Ваша логика
    return {
      content: [{ type: "text", text: `Status of ${service}: OK` }],
    };
  }
  throw new Error(`Unknown tool: ${request.params.name}`);
});

// Запуск
const transport = new StdioServerTransport();
await server.connect(transport);
```

```bash
npx tsc && node dist/index.js
```

**Python:**

```python
# server.py
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp import types

app = Server("my-server")

@app.list_tools()
async def list_tools() -> list[types.Tool]:
    return [
        types.Tool(
            name="get_status",
            description="Получить статус системы",
            inputSchema={
                "type": "object",
                "properties": {
                    "service": {"type": "string", "description": "Название сервиса"}
                },
                "required": ["service"],
            },
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[types.TextContent]:
    if name == "get_status":
        service = arguments["service"]
        return [types.TextContent(type="text", text=f"Status of {service}: OK")]
    raise ValueError(f"Unknown tool: {name}")

async def main():
    async with stdio_server() as streams:
        await app.run(*streams, app.create_initialization_options())

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

---

### 3. Подключить к Claude Code

```json
// ~/.claude/settings.json или .claude/settings.json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["/absolute/path/to/dist/index.js"]
    }
  }
}
```

Для Python:

```json
{
  "mcpServers": {
    "my-server": {
      "command": "python",
      "args": ["/absolute/path/to/server.py"]
    }
  }
}
```

---

### 4. Проверить подключение

```bash
# Перезапустить Claude Code, затем в сессии:
/mcp  # показывает список подключённых серверов
```

Инструмент появится в списке доступных tools для Claude.

---

### 5. Добавить Resources (опционально)

Resources — данные, которые Claude может читать (файлы, endpoints, состояние):

```typescript
server.setRequestHandler(ListResourcesRequestSchema, async () => ({
  resources: [
    {
      uri: "system://config",
      name: "System Config",
      mimeType: "application/json",
    },
  ],
}));

server.setRequestHandler(ReadResourceRequestSchema, async (request) => {
  if (request.params.uri === "system://config") {
    return {
      contents: [
        {
          uri: request.params.uri,
          mimeType: "application/json",
          text: JSON.stringify({ version: "1.0", env: "production" }),
        },
      ],
    };
  }
  throw new Error("Resource not found");
});
```

## Troubleshooting

**Сервер не появляется в Claude Code:**
- Убедитесь в абсолютном пути в конфигурации
- Проверьте что `command` и `args` запускают сервер без ошибок в терминале
- Перезапустите Claude Code после изменения настроек

**Tool call возвращает ошибку:**
- Добавьте `console.error()` / `print()` для логирования (stdout зарезервирован под MCP JSON-RPC, использовать только stderr)
- Проверьте соответствие `inputSchema` реально передаваемым аргументам

**Сервер падает сразу:**
- Для stdio-транспорта сервер должен читать stdin и писать в stdout через JSON-RPC
- Не выводить ничего в stdout кроме MCP-сообщений

## See Also

- [MCP Protocol — архитектура и primitives](protocol.md)
- [MCP Configuration — подключение к Claude Code](configuration.md)
- [MCP Servers — каталог готовых серверов](servers.md)
