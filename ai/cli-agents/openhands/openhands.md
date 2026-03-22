# OpenHands (ex-OpenDevin)

## Overview

Open-source платформа для автономных AI-агентов-разработчиков. Агент работает в изолированном Docker-sandbox с доступом к браузеру, терминалу и редактору — как настоящий разработчик за компьютером. Один из самых автономных open-source агентов.

- **Вендор:** OpenHands Community (ex-OpenDevin, All Hands AI)
- **Модель:** любая — Claude, GPT-4o, Gemini, Llama
- **Open Source:** ✓ MIT
- **Сайт:** all-hands.dev

## Key Features

- **Sandbox environment** — Docker-контейнер с полным доступом: bash, браузер, редактор файлов
- **Browser agent** — может открывать сайты, заполнять формы, кликать — полноценный веб-браузер
- **CodeAct** — агент пишет и исполняет Python-код как основной инструмент действия
- **Multi-agent** — оркестрация нескольких специализированных агентов
- **Microagents** — встроенная база знаний по конкретным технологиям (GitHub, npm и др.)
- **GitHub integration** — может работать с issues и PR напрямую
- **Evaluation suite** — встроенный набор бенчмарков (SWE-bench, WebArena и др.)

## Use Cases

- Полностью автономное решение задач: от чтения issue до открытия PR
- Задачи с web-компонентом: парсинг, заполнение форм, исследование сайтов
- Исследование и оценка агентных систем
- Команды, которым нужна максимальная автономия в изолированной среде

## Setup

```bash
docker pull docker.all-hands.dev/all-hands-ai/openhands:main
docker run -it --rm \
  -e SANDBOX_RUNTIME_CONTAINER_IMAGE=docker.all-hands.dev/all-hands-ai/runtime:main \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -p 3000:3000 \
  docker.all-hands.dev/all-hands-ai/openhands:main
```

## Considerations

- Требует Docker — более сложный деплой чем pip/npm-агенты
- Высокое потребление токенов из-за браузерных действий
- Веб-интерфейс, нет чистого CLI-режима
- Автономия высокая — важно доверять агенту или внимательно следить
