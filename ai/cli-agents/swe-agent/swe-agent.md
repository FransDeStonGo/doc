# SWE-agent

## Overview

Исследовательский агент от Princeton NLP Group. Создан специально для автономного решения реальных GitHub issues — находит баг, пишет фикс, создаёт патч. Один из первых агентов, показавших высокие результаты на бенчмарке SWE-bench.

- **Вендор:** Princeton NLP Group
- **Модель:** GPT-4, Claude (настраивается)
- **Open Source:** ✓ MIT
- **Сайт:** swe-agent.com

## Key Features

- **ACI (Agent-Computer Interface)** — специальный интерфейс взаимодействия агента с компьютером: bash + file editor
- **SWE-bench** — создан для и оптимизирован под задачи из реальных GitHub issues
- **Trajectory logging** — детальное логирование всех действий агента для анализа
- **Config-driven** — поведение агента настраивается через YAML-конфиги
- **Агентные паттерны** — встроенные паттерны: поиск → локализация бага → фикс → верификация

## Use Cases

- Автоматическое закрытие GitHub issues с патчем
- Исследование агентных систем и бенчмаркинг
- Автоматизация bug fixing в CI/CD пайплайне
- Изучение того, как агент рассуждает при отладке кода

## Setup

```bash
pip install sweagent
sweagent run \
  --agent.model.model_name=claude-sonnet-4-6 \
  --env.repo.github_url=https://github.com/owner/repo \
  --problem_statement.github_url=https://github.com/owner/repo/issues/123
```

## Considerations

- Заточен под конкретный сценарий (GitHub issues), менее универсален
- Высокое потребление токенов на каждую задачу
- Требует хорошо сформулированного issue для качественного результата
- Исследовательский проект — production-support ограничен
