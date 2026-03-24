---
type: guide
tags: [mcp, configuration, claude-code, settings, stdio, http, env, setup]
status: complete
updated: 2026-03-24
related:
  - mcp/mcp.md
  - mcp/protocol.md
  - mcp/servers.md
---

# Как настроить MCP в Claude Code

## Overview

Подключение MCP-серверов к Claude Code через `settings.json`. После настройки серверы появляются как дополнительные инструменты в агентной сессии.

---

## Prerequisites

- Claude Code установлен (`npm install -g @anthropic-ai/claude-code`)
- Нужный MCP-сервер установлен или доступен по URL

---

## Где хранятся настройки

| Файл | Область | Попадает в git |
|------|---------|:--------------:|
| `~/.claude/settings.json` | Глобальный (все проекты) | Нет |
| `.claude/settings.json` | Проектный | Да |
| `.claude/settings.local.json` | Проектный (локальный) | Нет |

**Приоритет:** `settings.local.json` > `settings.json` (проектный) > `settings.json` (глобальный)

Рекомендация: общие серверы (memory, fetch) — в глобальный; серверы с API-ключами — в локальный (`settings.local.json`), не коммитить.

---

## Steps

### 1. Создать или открыть файл настроек

```bash
# Глобальные настройки
nano ~/.claude/settings.json

# Локальные настройки проекта (не попадёт в git)
nano .claude/settings.local.json
```

### 2. Добавить секцию `mcpServers`

Базовая структура:

```json
{
  "mcpServers": {
    "<имя-сервера>": { /* конфигурация */ }
  }
}
```

### 3. Настроить stdio сервер (локальный)

Для серверов, работающих как дочерний процесс:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/projects"],
      "env": {}
    },
    "git": {
      "command": "uvx",
      "args": ["mcp-server-git"]
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

| Поле | Описание |
|------|---------|
| `command` | Команда для запуска сервера (`node`, `python`, `npx`, `uvx`) |
| `args` | Аргументы командной строки |
| `env` | Переменные окружения для процесса сервера |

### 4. Настроить HTTP сервер (удалённый)

Для серверов с Streamable HTTP транспортом:

```json
{
  "mcpServers": {
    "sentry": {
      "url": "https://mcp.sentry.io/mcp",
      "headers": {
        "Authorization": "Bearer ${SENTRY_AUTH_TOKEN}"
      }
    },
    "custom-api": {
      "url": "http://localhost:8080/mcp",
      "headers": {
        "X-API-Key": "your-api-key"
      }
    }
  }
}
```

| Поле | Описание |
|------|---------|
| `url` | URL MCP-сервера |
| `headers` | HTTP заголовки (авторизация и др.) |

### 5. Использование переменных окружения

Ссылки на переменные через `${VAR_NAME}` — заменяются из окружения:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

Установить переменные в `.env` файл проекта или экспортировать в shell перед запуском `claude`.

### 6. Перезапустить Claude Code

После изменения `settings.json` нужно перезапустить сессию Claude Code — серверы подключаются при старте.

---

## Полный пример настроек

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/home/user/projects",
        "/home/user/documents"
      ]
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    },
    "fetch": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-fetch"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "git": {
      "command": "uvx",
      "args": ["mcp-server-git", "--repository", "/path/to/repo"]
    }
  },
  "permissions": {
    "allow": [
      "mcp__filesystem__read_file",
      "mcp__filesystem__list_directory"
    ],
    "ask": [
      "mcp__filesystem__write_file",
      "mcp__filesystem__delete_file"
    ]
  }
}
```

---

## Troubleshooting

**Сервер не появляется в инструментах:**
- Проверь синтаксис JSON (`settings.json` должен быть валидным)
- Убедись что команда сервера установлена (`which npx`, `which uvx`)
- Попробуй запустить команду вручную в терминале

**Ошибка запуска сервера:**
- Проверь что пакет установлен: `npx -y @modelcontextprotocol/server-memory` (без Claude)
- Проверь что путь в `args` существует (для filesystem-сервера)

**Переменная окружения не подставляется:**
- Экспортируй переменную в shell перед запуском: `export GITHUB_TOKEN=...`
- Или пропиши значение напрямую в `env` (но не коммить в git)

**Проблемы с правами доступа:**
- Добавь разрешения в `permissions.allow` для нужных MCP-инструментов
- Формат: `mcp__<server-name>__<tool-name>`

---

## See Also

- [MCP Серверы](servers.md) — каталог доступных серверов
- [MCP Protocol](protocol.md) — как работает протокол
- [Claude Code навигация](../ai/anthropic/claude/claude.md) — другие настройки Claude Code
