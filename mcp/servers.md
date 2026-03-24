---
type: comparison
tags: [mcp, servers, tools, filesystem, github, database, browser, memory, official, community]
status: complete
updated: 2026-03-24
related:
  - mcp/mcp.md
  - mcp/protocol.md
  - mcp/configuration.md
---

# MCP Серверы

## Overview

Каталог готовых MCP-серверов для подключения к AI-агентам. Серверы делятся на официальные референс-реализации от MCP-команды, официальные интеграции от компаний-партнёров и комьюнити-серверы.

Полный список: [github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)

---

## Официальные референс-серверы (от MCP-команды)

| Сервер | Пакет | Что делает |
|--------|-------|-----------|
| **Filesystem** | `@modelcontextprotocol/server-filesystem` | Безопасные операции с файлами в указанных директориях |
| **Fetch** | `@modelcontextprotocol/server-fetch` | Загрузка веб-страниц и конвертация в текст для LLM |
| **Git** | `mcp-server-git` (Python) | Чтение, поиск и работа с git-репозиториями |
| **Memory** | `@modelcontextprotocol/server-memory` | Персистентная память на основе knowledge graph |
| **Time** | `@modelcontextprotocol/server-time` | Работа с временем и таймзонами |
| **Sequential Thinking** | `@modelcontextprotocol/server-sequentialthinking` | Структурированное пошаговое рассуждение |

### Установка и запуск

```bash
# TypeScript серверы — через npx (без установки)
npx -y @modelcontextprotocol/server-filesystem /path/to/dir
npx -y @modelcontextprotocol/server-memory

# Python серверы — через uvx (рекомендуется) или pip
uvx mcp-server-git
# или
pip install mcp-server-git && python -m mcp_server_git
```

---

## Официальные интеграции (от компаний)

Поддерживаются компаниями-владельцами платформ.

### Разработка и код

| Сервер | Компания | Что делает |
|--------|---------|-----------|
| **GitHub** | GitHub | Issues, PRs, репозитории, код |
| **Atlassian** | Atlassian | Jira задачи + Confluence страницы |
| **Sentry** | Sentry | Ошибки, трассировки, проекты |
| **AgentOps** | AgentOps | Observability и трассировка для AI-агентов |
| **Apollo GraphQL** | Apollo | Подключение GraphQL API к агентам |

### Данные и базы данных

| Сервер | Компания | Что делает |
|--------|---------|-----------|
| **Aiven** | Aiven | PostgreSQL, Kafka, ClickHouse, OpenSearch |
| **Algolia** | Algolia | Поиск по индексам Algolia |
| **Alkemi** | Alkemi | Snowflake, BigQuery, DataBricks через Alkemi |
| **Apache Doris** | Doris | MPP real-time data warehouse |

### Коммуникации и продуктивность

| Сервер | Компания | Что делает |
|--------|---------|-----------|
| **Slack** | Slack | Сообщения, каналы, поиск |
| **2slides** | 2slides | Генерация презентаций и слайдов |
| **ActionKit** | Paragon | 130+ SaaS-интеграций (Salesforce, Gmail и др.) |

### Веб и данные

| Сервер | Компания | Что делает |
|--------|---------|-----------|
| **Apify** | Apify | 6000+ инструментов для web scraping |
| **AgentQL** | AgentQL | Структурированные данные из веб-контента |
| **AlphaVantage** | AlphaVantage | 100+ API для финансовых рынков |

### Платежи и финансы

| Сервер | Компания | Что делает |
|--------|---------|-----------|
| **Adfin** | Adfin | Инвойсы, оплата, сверка |
| **Alpaca** | Alpaca | Торговля акциями/опционами, рыночные данные |

---

## Популярные комьюнити-серверы

Поддерживаются сообществом, качество варьируется.

| Сервер | Что делает |
|--------|-----------|
| **Playwright / Puppeteer** | Браузерная автоматизация, скриншоты, web testing |
| **PostgreSQL / SQLite** | Прямые SQL-запросы к БД |
| **Redis** | Работа с Redis (ключи, кэш, очереди) |
| **Docker** | Управление контейнерами и образами |
| **Kubernetes** | Управление кластерами |
| **Linear** | Задачи и проекты в Linear |
| **Notion** | Страницы и базы данных Notion |
| **Google Drive** | Файлы и документы Google |
| **Google Calendar** | Календарь и события |
| **Gmail / Email** | Чтение и отправка email |
| **Discord** | Сообщения и серверы Discord |
| **Telegram** | Бот API Telegram |
| **YouTube** | Транскрипты и метаданные видео |
| **AWS / GCP / Azure** | Облачные ресурсы |
| **Brave Search** | Веб-поиск через Brave |
| **DuckDuckGo** | Приватный веб-поиск |

---

## Конфигурация в Claude Code

```json
// ~/.claude/settings.json или .claude/settings.json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/projects"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_..."
      }
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    },
    "sentry": {
      "url": "https://mcp.sentry.io/mcp",
      "headers": {
        "Authorization": "Bearer ${SENTRY_AUTH_TOKEN}"
      }
    }
  }
}
```

---

## Выбор сервера

| Нужно | Сервер |
|-------|--------|
| Работать с файлами системы | Filesystem |
| Читать веб-страницы | Fetch |
| Git операции | Git |
| Память между сессиями | Memory |
| GitHub issues/PRs | GitHub (официальный) |
| Jira / Confluence | Atlassian |
| SQL-запросы | PostgreSQL / SQLite community |
| Браузерная автоматизация | Playwright community |
| Поиск в интернете | Brave Search / DuckDuckGo community |
| Slack | Slack (официальный) |

---

## See Also

- [MCP Protocol](protocol.md) — архитектура и протокол
- [MCP Configuration](configuration.md) — полное руководство по настройке
- [Tool Use](../agents/tool-use.md) — как MCP-инструменты работают в агенте
