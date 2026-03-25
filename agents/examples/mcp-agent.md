---
type: guide
tags: [agents, examples, mcp, model-context-protocol, filesystem, github, python, intermediate]
status: complete
updated: 2026-03-25
related:
  - ../tool-use.md
  - ../../mcp/protocol.md
  - ../../mcp/servers.md
---

# MCP Agent

Агент с подключением к MCP-серверам (Model Context Protocol). Использует стандартизированный протокол для доступа к внешним инструментам: файловой системе, GitHub, базам данных и др.

---

## Overview

Этот пример показывает:
- Как подключиться к MCP-серверам (stdio transport)
- Как вызывать MCP tools через агента
- Как работать с filesystem и GitHub через MCP

**Сложность:** ⭐⭐ Средний  
**Строк кода:** ~180  
**MCP серверы:** filesystem, github

---

## Architecture

```
Claude Agent
    ↓
MCP Client (stdio)
    ↓
┌─────────────────────────────────┐
│  MCP Servers (stdio processes)  │
├─────────────────────────────────┤
│  @modelcontextprotocol/         │
│  server-filesystem              │
│  - read_file                    │
│  - write_file                   │
│  - list_directory               │
├─────────────────────────────────┤
│  @modelcontextprotocol/         │
│  server-github                  │
│  - search_repositories          │
│  - get_file_contents            │
│  - create_issue                 │
└─────────────────────────────────┘
```

---

## Code

```python
#!/usr/bin/env python3
"""
MCP Agent — агент с подключением к MCP-серверам.
Демонстрирует работу с filesystem и GitHub через Model Context Protocol.
"""

import os
import json
import subprocess
from typing import List, Dict, Any
from anthropic import Anthropic
from dotenv import load_dotenv

load_dotenv()

client = Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))

# ============================================================================
# MCP CLIENT (Simplified stdio transport)
# ============================================================================

class MCPClient:
    """Упрощённый MCP клиент для stdio transport"""
    
    def __init__(self, server_command: List[str], server_args: List[str] = None):
        """
        Args:
            server_command: Команда запуска сервера, e.g. ["npx", "-y", "@modelcontextprotocol/server-filesystem"]
            server_args: Аргументы сервера, e.g. ["/path/to/allowed/dir"]
        """
        self.server_command = server_command
        self.server_args = server_args or []
        self.process = None
        self.tools = []
    
    def start(self):
        """Запустить MCP сервер"""
        cmd = self.server_command + self.server_args
        
        self.process = subprocess.Popen(
            cmd,
            stdin=subprocess.PIPE,
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE,
            text=True,
            bufsize=1
        )
        
        # Инициализация: отправляем initialize request
        init_request = {
            "jsonrpc": "2.0",
            "id": 1,
            "method": "initialize",
            "params": {
                "protocolVersion": "2024-11-05",
                "capabilities": {},
                "clientInfo": {
                    "name": "mcp-agent",
                    "version": "1.0.0"
                }
            }
        }
        
        self._send(init_request)
        response = self._receive()
        
        # Запрашиваем список tools
        tools_request = {
            "jsonrpc": "2.0",
            "id": 2,
            "method": "tools/list",
            "params": {}
        }
        
        self._send(tools_request)
        tools_response = self._receive()
        
        if "result" in tools_response and "tools" in tools_response["result"]:
            self.tools = tools_response["result"]["tools"]
        
        print(f"✅ MCP Server started: {len(self.tools)} tools available")
    
    def _send(self, message: dict):
        """Отправить JSON-RPC сообщение"""
        self.process.stdin.write(json.dumps(message) + "\n")
        self.process.stdin.flush()
    
    def _receive(self) -> dict:
        """Получить JSON-RPC ответ"""
        line = self.process.stdout.readline()
        return json.loads(line)
    
    def call_tool(self, tool_name: str, arguments: dict) -> Any:
        """Вызвать MCP tool"""
        request = {
            "jsonrpc": "2.0",
            "id": 3,
            "method": "tools/call",
            "params": {
                "name": tool_name,
                "arguments": arguments
            }
        }
        
        self._send(request)
        response = self._receive()
        
        if "result" in response:
            return response["result"]["content"][0]["text"]
        elif "error" in response:
            return f"Error: {response['error']['message']}"
        else:
            return "Unknown error"
    
    def get_tools_for_claude(self) -> List[Dict]:
        """Конвертировать MCP tools в формат Claude"""
        claude_tools = []
        
        for tool in self.tools:
            claude_tools.append({
                "name": tool["name"],
                "description": tool["description"],
                "input_schema": tool["inputSchema"]
            })
        
        return claude_tools
    
    def stop(self):
        """Остановить MCP сервер"""
        if self.process:
            self.process.terminate()
            self.process.wait()


# ============================================================================
# MCP AGENT
# ============================================================================

class MCPAgent:
    """Агент с подключением к MCP-серверам"""
    
    def __init__(self):
        self.mcp_clients = {}
    
    def add_mcp_server(self, name: str, command: List[str], args: List[str] = None):
        """Добавить MCP сервер"""
        print(f"\n🔌 Connecting to MCP server: {name}")
        
        mcp_client = MCPClient(command, args)
        mcp_client.start()
        
        self.mcp_clients[name] = mcp_client
    
    def get_all_tools(self) -> List[Dict]:
        """Получить все tools от всех MCP серверов"""
        all_tools = []
        
        for name, client in self.mcp_clients.items():
            tools = client.get_tools_for_claude()
            all_tools.extend(tools)
        
        return all_tools
    
    def call_tool(self, tool_name: str, tool_input: dict) -> str:
        """Вызвать tool (найти нужный MCP сервер)"""
        for name, client in self.mcp_clients.items():
            for tool in client.tools:
                if tool["name"] == tool_name:
                    return client.call_tool(tool_name, tool_input)
        
        return f"Error: Tool '{tool_name}' not found"
    
    def run(self, user_message: str):
        """Запустить агента с MCP tools"""
        print(f"\n🧑 User: {user_message}\n")
        
        # Получаем все tools
        tools = self.get_all_tools()
        
        print(f"🔧 Available tools: {len(tools)}")
        for tool in tools:
            print(f"   - {tool['name']}")
        print()
        
        messages = [{"role": "user", "content": user_message}]
        
        # Первый запрос к Claude
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=2048,
            tools=tools,
            messages=messages
        )
        
        print(f"🤖 Agent thinking... (stop_reason: {response.stop_reason})\n")
        
        # Tool use loop
        while response.stop_reason == "tool_use":
            messages.append({"role": "assistant", "content": response.content})
            
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    tool_name = block.name
                    tool_input = block.input
                    
                    print(f"🔧 Calling MCP tool: {tool_name}")
                    print(f"   Input: {json.dumps(tool_input, indent=2)}")
                    
                    # Вызываем MCP tool
                    result = self.call_tool(tool_name, tool_input)
                    
                    print(f"   Result: {result[:200]}...\n")
                    
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": result
                    })
            
            messages.append({"role": "user", "content": tool_results})
            
            response = client.messages.create(
                model="claude-sonnet-4-20250514",
                max_tokens=2048,
                tools=tools,
                messages=messages
            )
            
            print(f"🤖 Agent thinking... (stop_reason: {response.stop_reason})\n")
        
        # Финальный ответ
        final_response = next(
            (block.text for block in response.content if hasattr(block, "text")),
            None
        )
        
        print(f"💬 Agent: {final_response}\n")
        
        return final_response
    
    def cleanup(self):
        """Остановить все MCP серверы"""
        for name, client in self.mcp_clients.items():
            client.stop()
            print(f"🔌 Disconnected from: {name}")


# ============================================================================
# EXAMPLES
# ============================================================================

if __name__ == "__main__":
    agent = MCPAgent()
    
    try:
        # 1. Подключаем filesystem MCP server
        agent.add_mcp_server(
            name="filesystem",
            command=["npx", "-y", "@modelcontextprotocol/server-filesystem"],
            args=[os.path.expanduser("~/Documents")]  # Разрешённая директория
        )
        
        # 2. Подключаем GitHub MCP server (требует GITHUB_TOKEN)
        if os.environ.get("GITHUB_TOKEN"):
            agent.add_mcp_server(
                name="github",
                command=["npx", "-y", "@modelcontextprotocol/server-github"],
                args=[]
            )
        
        print("\n" + "=" * 70)
        print("MCP AGENT EXAMPLES")
        print("=" * 70)
        
        # Пример 1: Работа с файловой системой
        agent.run("List all files in the current directory and read the first .txt file you find")
        
        print("\n" + "=" * 70 + "\n")
        
        # Пример 2: GitHub (если токен доступен)
        if os.environ.get("GITHUB_TOKEN"):
            agent.run("Search for popular Python MCP server repositories on GitHub")
        
    finally:
        # Очистка
        agent.cleanup()
```

---

## Prerequisites

```bash
# 1. Установите Node.js (для MCP серверов)
# https://nodejs.org/

# 2. Установите Python зависимости
pip install anthropic python-dotenv

# 3. MCP серверы устанавливаются автоматически через npx
# (при первом запуске может занять время)
```

---

## Environment Variables

```bash
# .env
ANTHROPIC_API_KEY=sk-ant-...
GITHUB_TOKEN=ghp_...  # Опционально, для GitHub MCP server
```

---

## Running

```bash
# 1. Создайте .env
echo "ANTHROPIC_API_KEY=sk-ant-..." > .env

# 2. Запустите агента
python mcp-agent.py
```

**Ожидаемый вывод:**
```
🔌 Connecting to MCP server: filesystem
✅ MCP Server started: 3 tools available

======================================================================
MCP AGENT EXAMPLES
======================================================================

🧑 User: List all files in the current directory...

🔧 Available tools: 3
   - read_file
   - write_file
   - list_directory

🤖 Agent thinking... (stop_reason: tool_use)

🔧 Calling MCP tool: list_directory
   Input: {
     "path": "."
   }
   Result: ["file1.txt", "file2.md", "folder/"]...

🔧 Calling MCP tool: read_file
   Input: {
     "path": "./file1.txt"
   }
   Result: "Content of file1.txt..."...

🤖 Agent thinking... (stop_reason: end_turn)

💬 Agent: I found 3 files in the directory. The first .txt file 
contains: [content summary]

🔌 Disconnected from: filesystem
```

---

## Available MCP Servers

### Official Servers

| Server | Tools | Use Case |
|--------|-------|----------|
| **filesystem** | read_file, write_file, list_directory | Работа с локальными файлами |
| **github** | search_repositories, get_file_contents, create_issue | GitHub API |
| **postgres** | query, list_tables | PostgreSQL queries |
| **sqlite** | query, list_tables | SQLite queries |
| **puppeteer** | navigate, screenshot, click | Browser automation |
| **slack** | send_message, list_channels | Slack integration |

### Adding More Servers

```python
# PostgreSQL
agent.add_mcp_server(
    name="postgres",
    command=["npx", "-y", "@modelcontextprotocol/server-postgres"],
    args=["postgresql://user:pass@localhost/dbname"]
)

# Puppeteer (browser automation)
agent.add_mcp_server(
    name="puppeteer",
    command=["npx", "-y", "@modelcontextprotocol/server-puppeteer"],
    args=[]
)
```

---

## Best Practices

1. **Security** — ограничивайте доступ MCP серверов (filesystem: только нужные директории)
2. **Error handling** — MCP серверы могут падать, добавьте retry логику
3. **Timeouts** — добавьте таймауты для tool calls (некоторые могут зависать)
4. **Cleanup** — всегда останавливайте MCP серверы в `finally` блоке
5. **Logging** — логируйте все MCP tool calls для отладки

---

## Troubleshooting

**MCP server не запускается:**
```bash
# Проверьте что Node.js установлен
node --version

# Попробуйте запустить сервер вручную
npx -y @modelcontextprotocol/server-filesystem ~/Documents
```

**Tool not found:**
- Проверьте что сервер успешно стартовал (`tools` не пустой)
- Проверьте spelling tool name (case-sensitive)

**Timeout при tool call:**
- Увеличьте timeout в subprocess.Popen
- Проверьте что сервер не завис (stderr output)

---

## See Also

- [MCP Protocol](../../mcp/protocol.md) — детали протокола
- [MCP Servers](../../mcp/servers.md) — каталог доступных серверов
- [Tool Use](../tool-use.md) — базовый механизм tool use
