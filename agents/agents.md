---
type: nav
tags: [agents, architecture, tool-use, memory, planning, multi-agent, permissions, workflows]
status: complete
updated: 2026-03-24
---

# Agents

## Overview

Архитектура, паттерны и типы AI-агентов. Раздел охватывает фундаментальные концепции построения агентных систем: от простых tool-use агентов до сложных multi-agent оркестраций. За основу взяты стандарты и терминология Anthropic.

## Structure

```
agents/
├── agents.md           # Этот файл — навигация по разделу
├── what-is-agent.md    # Что такое агент: agent vs workflow, augmented LLM, принципы
├── workflows.md        # 5 паттернов workflow по Anthropic
├── tool-use.md         # Механизм tool use: структура, цикл вызова, best practices
├── memory.md           # 4 типа памяти: in-context, external, in-weights, in-cache
├── planning.md         # ReAct, CoT, Extended Thinking, Interleaved Thinking
├── multi-agent.md      # Multi-agent: роли, изоляция контекста, оркестрация
├── permissions.md      # Безопасность: least privilege, blast radius, human-in-the-loop
├── evaluation.md       # Оценка качества агентных систем, eval frameworks
└── observability.md    # Наблюдаемость: трассировка, метрики, Langfuse, Phoenix
```

## Topics

- **[Что такое агент](what-is-agent.md)** — agentic systems, agent vs workflow, augmented LLM, когда что выбирать, принципы дизайна Anthropic
- **[Workflow-паттерны](workflows.md)** — 5 паттернов: Prompt Chaining, Routing, Parallelization, Orchestrator-Workers, Evaluator-Optimizer
- **[Tool Use](tool-use.md)** — структура инструмента, цикл вызова, best practices, MCP-совместимость
- **[Memory](memory.md)** — 4 типа памяти: in-context, external, in-weights, in-cache
- **[Planning](planning.md)** — ReAct, CoT, Extended Thinking, Interleaved Thinking
- **[Multi-Agent](multi-agent.md)** — роли, изоляция контекста, паттерны оркестрации
- **[Permissions](permissions.md)** — least privilege, blast radius, human-in-the-loop, stopping conditions
- **[Evaluation](evaluation.md)** — оценка качества: unit evals, e2e, LLM-judge, инструменты (Braintrust, LangSmith, PromptFoo)
- **[Observability](observability.md)** — наблюдаемость: traces, spans, метрики стоимости, Langfuse, Phoenix, OpenTelemetry
