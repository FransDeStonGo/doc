---
type: concept
tags: [skills, slash-commands, claude-code, anatomy, frontmatter, arguments, subagent, invocation]
status: complete
updated: 2026-03-24
related:
  - skills/skills.md
  - skills/examples.md
  - mcp/configuration.md
  - agents/multi-agent.md
---

# Анатомия Skill

## Overview

Skill — файл с инструкциями для Claude, который превращается в slash-команду `/skill-name`. Новый формат: директория с `SKILL.md` внутри. Старый формат `.claude/commands/command.md` тоже работает.

---

## Структура файла SKILL.md

```
my-skill/
├── SKILL.md           # Основной файл (обязательный)
├── template.md        # Шаблон для заполнения
├── examples/
│   └── sample.md      # Пример результата
└── scripts/
    └── validate.sh    # Скрипт для выполнения
```

**SKILL.md** состоит из двух частей:

1. **Frontmatter** — YAML между `---` маркерами (метаданные и конфигурация)
2. **Markdown content** — инструкции для Claude

```markdown
---
name: my-skill
description: What this skill does and when to use it
---

# Instructions for Claude

Step-by-step instructions here...
```

---

## Frontmatter: все поля

```yaml
---
name: my-skill                    # Имя → /my-skill команда (lowercase, hyphens)
description: |                    # Описание — Claude использует для авто-активации
  What this skill does.
  Use when the user asks to...
argument-hint: "[filename] [format]"  # Подсказка в autocomplete
disable-model-invocation: true    # true = только ручной вызов /my-skill
user-invocable: false             # false = скрыть из меню /
allowed-tools: Read, Grep, Glob   # Инструменты без запроса разрешения
model: claude-opus-4-6            # Конкретная модель для этого skill
effort: high                      # Уровень: low, medium, high, max
context: fork                     # fork = запуск в изолированном subagent
agent: Explore                    # Тип subagent (если context: fork)
---
```

**Минимальный рабочий skill:**

```yaml
---
name: summarize
description: Summarize the current file or provided text
---

Summarize $ARGUMENTS concisely. Focus on key points.
```

---

## Где хранить skills

| Расположение | Путь | Область |
|------------|------|--------|
| Личные (все проекты) | `~/.claude/skills/<name>/SKILL.md` | Только у тебя |
| Проектные | `.claude/skills/<name>/SKILL.md` | Только этот проект |
| Старый формат | `.claude/commands/<name>.md` | Только этот проект |

**Приоритет:** личные > проектные. Команды vs skills — skills побеждают при совпадении имён.

---

## Аргументы

### $ARGUMENTS — всё что передано

```yaml
---
name: fix-issue
description: Fix a GitHub issue by number
disable-model-invocation: true
---

Fix GitHub issue $ARGUMENTS following our coding standards.

1. Read the issue description
2. Implement the fix
3. Write tests
4. Create a commit
```

Вызов: `/fix-issue 123` → `$ARGUMENTS` = `"123"`

### $ARGUMENTS[N] / $N — позиционный аргумент

```yaml
---
name: migrate
description: Migrate a component between frameworks
---

Migrate the $0 component from $1 to $2.
Preserve all existing behavior and tests.
```

Вызов: `/migrate SearchBar React Vue`
→ `$0` = `SearchBar`, `$1` = `React`, `$2` = `Vue`

### Если $ARGUMENTS не указан

Если skill не содержит `$ARGUMENTS`, но аргументы переданы — Claude Code автоматически добавляет `ARGUMENTS: <value>` в конец prompt.

---

## Динамический контекст (bash injection)

Синтаксис `` !`команда` `` — выполняет shell-команду до отправки Claude, результат вставляется в prompt:

```yaml
---
name: pr-summary
description: Summarize changes in current pull request
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## PR Context
- Diff: !`gh pr diff`
- Comments: !`gh pr view --comments`
- Changed files: !`gh pr diff --name-only`

## Task
Summarize this pull request for the team...
```

Порядок выполнения:
1. Все `` !`команда` `` выполняются первыми (Claude не видит команды)
2. Результаты вставляются в prompt
3. Claude получает готовый prompt с реальными данными

---

## Управление вызовом

### Только ручной вызов (disable-model-invocation)

Для деструктивных операций — деплой, коммит, отправка:

```yaml
---
name: deploy
description: Deploy the application to production
disable-model-invocation: true  # Claude не будет запускать сам
---

Deploy $ARGUMENTS to production...
```

### Только автоматический (user-invocable: false)

Для фонового контекста (конвенции, нотации):

```yaml
---
name: legacy-context
description: Context about the legacy authentication system
user-invocable: false  # Не показывать в меню /
---

The legacy auth system uses JWT tokens stored in...
```

### Таблица поведения

| Frontmatter | Ты вызываешь | Claude вызывает | Загрузка в контекст |
|------------|------------|----------------|-------------------|
| (по умолчанию) | Да | Да | Описание всегда, полный при вызове |
| `disable-model-invocation: true` | Да | Нет | Не в контексте |
| `user-invocable: false` | Нет | Да | Описание всегда, полный при вызове |

---

## Subagent execution (context: fork)

Skill в изолированном контексте — без доступа к истории разговора:

```yaml
---
name: deep-research
description: Research a topic thoroughly in the codebase
context: fork
agent: Explore
---

Research $ARGUMENTS:

1. Find relevant files with Glob and Grep
2. Read and analyze the code
3. Return findings with specific file references
```

Типы agent:
- `Explore` — read-only, оптимизирован для исследования кодовой базы
- `Plan` — архитектурное проектирование
- `general-purpose` — полный набор инструментов
- Любой кастомный из `.claude/agents/`

---

## Специальные переменные

| Переменная | Значение |
|----------|--------|
| `$ARGUMENTS` | Всё что передано после имени команды |
| `$ARGUMENTS[N]` / `$N` | N-й аргумент (0-indexed) |
| `${CLAUDE_SESSION_ID}` | ID текущей сессии |
| `${CLAUDE_SKILL_DIR}` | Путь к директории skill |

Пример с логированием:

```yaml
---
name: session-logger
description: Log activity for this session
---

Log the following to logs/${CLAUDE_SESSION_ID}.log:

$ARGUMENTS
```

---

## Ограничение инструментов

```yaml
---
name: safe-reader
description: Read files without making changes
allowed-tools: Read, Grep, Glob
---

Read and analyze the following without modifying anything:
$ARGUMENTS
```

Без `allowed-tools` Claude запрашивает разрешение на каждый инструмент (по настройкам сессии).

---

## Размер и организация

- Держи `SKILL.md` до **500 строк**
- Объёмные материалы (API docs, примеры) выноси в отдельные файлы
- Ссылайся на поддерживающие файлы из `SKILL.md`:

```markdown
## Additional resources

- Full API reference: see [reference.md](reference.md)
- Usage examples: see [examples/sample.md](examples/sample.md)
```

---

## Совет: Extended Thinking

Добавь слово `ultrathink` в контент skill — активирует Extended Thinking для данного skill.

---

## See Also

- [Skills examples](examples.md) — готовые примеры для типовых задач
- [Multi-agent](../agents/multi-agent.md) — subagents и оркестрация
- [MCP Configuration](../mcp/configuration.md) — другие способы расширить Claude Code
