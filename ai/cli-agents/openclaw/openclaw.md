# OpenClaw

## Overview

OpenClaw — self-hosted шлюз, который подключает мессенджеры (WhatsApp, Telegram, Discord, iMessage) к AI-агентам. По сути, это не coding CLI-агент, а **gateway для AI-агентов через каналы коммуникации** — позволяет общаться с агентом из кармана, через привычный мессенджер.

- **Вендор:** Community
- **Модель:** любая (рекомендуется самая мощная актуальная)
- **Open Source:** ✓ MIT
- **Сайт:** docs.openclaw.ai

> **Важно:** OpenClaw — не coding CLI-агент в ряду Claude Code / Aider / Goose. Это отдельная категория: инфраструктурный слой доставки агентов через мессенджеры.

## Key Features

- **Multi-channel gateway** — один процесс обслуживает несколько мессенджеров одновременно: WhatsApp, Telegram, Discord, iMessage
- **Agent routing** — маршрутизация между агентами с изолированными сессиями для каждого
- **Media support** — обмен изображениями, аудио, документами прямо в чате
- **Web dashboard** — браузерная панель управления шлюзом
- **Mobile nodes** — iOS и Android-узлы: Camera, Canvas, голосовые команды
- **Self-hosted** — полная независимость от облачных провайдеров, MIT-лицензия
- **Daemon mode** — работает как системный сервис в фоне

## Use Cases

- Личный AI-ассистент, доступный через WhatsApp/Telegram без облаков
- Интеграция AI-агентов в корпоративные мессенджеры (Discord, iMessage)
- Мобильный доступ к агенту: голос, камера, документы прямо из смартфона
- Self-hosted альтернатива коммерческим AI-чатботам для команды

## Setup

```bash
# Требует Node 24 или Node 22 LTS (22.16+)
npm install -g openclaw@latest
openclaw onboard --install-daemon
openclaw dashboard
```

Занимает около 5 минут.

## Considerations

- Не заменяет coding CLI-агентов — другая задача (доставка, а не разработка)
- Качество агента зависит от подключённой модели
- Требует настройки API каждого мессенджера (WhatsApp Business API и т.д.)
- Молодой проект, документация в развитии
