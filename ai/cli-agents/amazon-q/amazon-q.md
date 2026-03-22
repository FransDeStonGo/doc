# Amazon Q Developer

## Overview

Enterprise AI-агент от AWS, встроенный в CLI и IDE. Знает AWS-сервисы лучше любого другого агента: может трансформировать код, объяснять и генерировать IaC (CloudFormation, CDK), работать с консолью AWS. Ориентирован на команды, глубоко сидящие в AWS-экосистеме.

- **Вендор:** Amazon Web Services
- **Модель:** Amazon Titan + дообученные модели на AWS-документации
- **Open Source:** нет
- **Требования:** AWS-аккаунт, подписка Q Developer (есть free tier)

## Key Features

- **AWS-контекст** — глубокое знание всех AWS-сервисов, SDK, CDK, CloudFormation
- **`/dev` mode** — полноценный агент для написания кода с доступом к файловой системе
- **Code transformation** — автоматический апгрейд кода (Java 8 → 17, .NET Framework → .NET 8)
- **Security scanning** — встроенное сканирование уязвимостей в коде
- **`q chat`** — диалог в терминале с AWS-контекстом и документацией
- **IaC generation** — генерация CloudFormation / CDK / Terraform по описанию
- **IDE plugins** — VS Code, JetBrains, Visual Studio

## Use Cases

- Работа с AWS: написание Lambda, настройка IAM, генерация CDK стеков
- Апгрейд legacy-кода (Java, .NET) с автоматической трансформацией
- Security review кода на AWS best practices
- Команды с AWS-центричной инфраструктурой

## Setup

```bash
# Через AWS CLI
aws configure
q chat
```

## Considerations

- Слабее вне AWS-экосистемы — универсальные задачи лучше у Claude Code
- Ограниченная модель разрешений по сравнению с Claude Code
- Требует AWS-аккаунта и настройки IAM
- Меньше кастомизации агентного поведения
