# INDEX — Индекс базы знаний

Плоский список всех документов базы. Используется для быстрой навигации и как точка входа для AI-архивариуса.

Формат: `путь | тип | статус | теги | описание`

Обновлять при добавлении каждого нового файла.

---

## Мета

| Файл | Тип | Статус | Теги | Описание |
|------|-----|--------|------|---------|
| `project.md` | — | complete | project, structure | Описание проекта, структура базы, принципы |
| `CONVENTIONS.md` | — | complete | meta, standards, conventions | Стандарты базы: типология, frontmatter, шаблоны, правила |
| `INDEX.md` | — | complete | meta, index | Этот файл — плоский индекс всех документов |

## Archivist — Агенты-архивариусы

| Файл | Тип | Статус | Теги | Описание |
|------|-----|--------|------|---------|
| `archivist/archivist.md` | nav | complete | archivist, agents, meta | Навигация по агентам-архивариусам, таблица сравнения |
| `archivist/haiku.md` | reference | complete | archivist, haiku, search, system-prompt | Системный промпт лёгкого архивариуса для Haiku (только чтение) |
| `archivist/sonnet.md` | reference | complete | archivist, sonnet, full, system-prompt, write | Системный промпт полного архивариуса для Sonnet (чтение + запись) |

---

## Agents — Архитектура агентов

| Файл | Тип | Статус | Теги | Описание |
|------|-----|--------|------|---------|
| `agents/agents.md` | nav | complete | agents, architecture, overview | Навигация по разделу: 9 документов (what-is-agent, workflows, tool-use, memory, planning, multi-agent, permissions, evaluation, observability) |
| `agents/what-is-agent.md` | concept | complete | agents, agentic-systems, workflow, definition, anthropic, augmented-llm | Что такое агент: agent vs workflow, augmented LLM, когда что выбирать, принципы дизайна |
| `agents/workflows.md` | concept | complete | agents, workflows, patterns, prompt-chaining, routing, parallelization, orchestrator, evaluator | 5 паттернов workflow по Anthropic с примерами и decision guide |
| `agents/tool-use.md` | concept | complete | agents, tool-use, function-calling, mcp, api, anthropic | Механизм tool use: структура инструмента, цикл вызова, параллельные вызовы, best practices |
| `agents/memory.md` | concept | complete | agents, memory, storage, in-context, external, rag, vector, in-cache | 4 типа памяти агента: in-context, external, in-weights, in-cache — сравнение и паттерны |
| `agents/planning.md` | concept | complete | agents, planning, reasoning, react, chain-of-thought, extended-thinking, anthropic | Планирование агентов: ReAct, CoT, Extended Thinking, Interleaved Thinking |
| `agents/multi-agent.md` | concept | complete | agents, multi-agent, orchestration, subagents, specialization, context-isolation | Multi-agent системы: роли, изоляция контекста, паттерны коммуникации, tradeoffs |
| `agents/permissions.md` | concept | complete | agents, permissions, security, blast-radius, human-in-the-loop, least-privilege, safety | Безопасность агентов: least privilege, blast radius, human-in-the-loop, stopping conditions |
| `agents/evaluation.md` | concept | complete | agents, evaluation, testing, evals, metrics, braintrust, langsmith, llm-judge | Оценка качества агентов: unit evals, e2e, LLM-judge, трассы, инструменты |
| `agents/observability.md` | concept | complete | agents, observability, tracing, spans, opentelemetry, langfuse, phoenix, cost | Наблюдаемость агентов: traces/spans, cost tracking, структурированные логи, Langfuse/Phoenix |

---

## AI — Платформы и модели

| Файл | Тип | Статус | Теги | Описание |
|------|-----|--------|------|---------|
| `ai/ai.md` | nav | complete | ai, models, comparison, overview | Навигация, сравнительная таблица моделей, гайд по выбору |
| `ai/anthropic/anthropic.md` | reference | complete | anthropic, claude, models, constitutional-ai, tool-use | Anthropic: Constitutional AI, модели Claude (Opus/Sonnet/Haiku), агентные сильные стороны, setup |
| `ai/anthropic/claude/claude.md` | nav | complete | anthropic, claude, claude-code, hooks, memory, mcp, skills | Claude Code: ключевые возможности (hooks, CLAUDE.md, memory, sub-agents, MCP, skills) |
| `ai/openai/openai.md` | reference | complete | openai, gpt, o-series, structured-outputs, assistants | OpenAI: GPT-4o, o3/o4-mini, Structured Outputs, Assistants API, setup |
| `ai/google/google.md` | reference | complete | google, gemini, long-context, multimodal, vertex-ai, grounding | Gemini: 1M контекст, Google Search grounding, Code Execution, Vertex AI vs AI Studio, setup |
| `ai/meta/meta.md` | reference | complete | meta, llama, open-source, self-hosted, ollama, vllm | Llama 3.x: open weights, локальный деплой, Ollama/vLLM, privacy, fine-tuning |
| `ai/mistral/mistral.md` | reference | complete | mistral, codestral, gdpr, european, open-source, fim | Mistral: Large/Small/Codestral, GDPR-compliant, FIM для code completion, setup |
| `ai/deepseek/deepseek.md` | reference | complete | deepseek, reasoning, open-source, cheap, moe, r1 | DeepSeek V3/R1: MoE-архитектура, reasoning open weights, цена в 10-20x ниже GPT-4o, setup |
| `ai/qwen/qwen.md` | reference | complete | qwen, alibaba, open-source, code, vision, qwq | Qwen2.5: open-source, Coder-32B, VL-72B vision, QwQ-32B reasoning, DashScope API |
| `ai/perplexity/perplexity.md` | reference | complete | perplexity, sonar, web-search, citations, real-time | Perplexity Sonar: нативный веб-поиск, citations, Sonar Deep Research, setup |

---

## AI — CLI-агенты

| Файл | Тип | Статус | Теги | Описание |
|------|-----|--------|------|---------|
| `ai/cli-agents/cli-agents.md` | comparison | complete | cli-agents, comparison, overview | Сводная таблица 14 CLI-агентов: возможности, модели, лицензии |
| `ai/cli-agents/claude-code/claude-code.md` | reference | complete | cli-agents, claude-code, anthropic | Claude Code — CLI-агент от Anthropic, глубокая интеграция с Claude |
| `ai/cli-agents/aider/aider.md` | reference | complete | cli-agents, aider, open-source, git | Aider — open-source агент с git-интеграцией и multi-model |
| `ai/cli-agents/goose/goose.md` | reference | complete | cli-agents, goose, block, mcp | Goose от Block — MCP-native агент, расширяемый плагинами |
| `ai/cli-agents/plandex/plandex.md` | reference | complete | cli-agents, plandex, planning, long-tasks | Plandex — агент для длинных задач с планированием и rollback |
| `ai/cli-agents/openhands/openhands.md` | reference | complete | cli-agents, openhands, open-source, sandbox | OpenHands (ex-OpenDevin) — open-source агент в изолированной среде |
| `ai/cli-agents/swe-agent/swe-agent.md` | reference | complete | cli-agents, swe-agent, research, princeton | SWE-agent — исследовательский агент от Princeton для SWE-bench |
| `ai/cli-agents/copilot-cli/copilot-cli.md` | reference | complete | cli-agents, copilot, github, microsoft | GitHub Copilot CLI — агент в экосистеме GitHub/VS Code |
| `ai/cli-agents/gemini-cli/gemini-cli.md` | reference | complete | cli-agents, gemini-cli, google, long-context | Gemini CLI от Google — доступ к 1M контексту в терминале |
| `ai/cli-agents/amazon-q/amazon-q.md` | reference | complete | cli-agents, amazon-q, aws, enterprise | Amazon Q Developer — агент глубоко интегрированный с AWS |
| `ai/cli-agents/codex-cli/codex-cli.md` | reference | complete | cli-agents, codex, openai, sandbox | Codex CLI от OpenAI — агент с изолированной sandbox-средой |
| `ai/cli-agents/amp/amp.md` | reference | complete | cli-agents, amp, ampcode | Amp (AmpCode) — CLI-агент от Sourcegraph |
| `ai/cli-agents/opencode/opencode.md` | reference | complete | cli-agents, opencode, open-source, tui | opencode — open-source агент с TUI, multi-model |
| `ai/cli-agents/qwen-code/qwen-code.md` | reference | complete | cli-agents, qwen-code, open-source, alibaba | Qwen Code — CLI-агент на базе Qwen2.5-Coder |
| `ai/cli-agents/openclaw/openclaw.md` | reference | complete | cli-agents, openclaw, open-source, claude-code-fork | OpenClaw — open-source форк Claude Code |

---

## MCP — Model Context Protocol

| Файл | Тип | Статус | Теги | Описание |
|------|-----|--------|------|---------|
| `mcp/mcp.md` | nav | complete | mcp, protocol, tools, servers | Навигация: протокол, серверы, кастомные серверы, конфигурация |
| `mcp/protocol.md` | concept | complete | mcp, protocol, architecture, tools, resources, prompts, transport, stdio, http | Архитектура MCP: host/client/server, транспорты, primitives, lifecycle, JSON-RPC |
| `mcp/servers.md` | comparison | complete | mcp, servers, filesystem, github, database, browser, memory, official, community | Каталог MCP-серверов: референс, официальные интеграции, комьюнити |
| `mcp/configuration.md` | guide | complete | mcp, configuration, claude-code, settings, stdio, http, env | Настройка MCP в Claude Code: settings.json, stdio и HTTP серверы, env vars, troubleshooting |
| `mcp/custom-server.md` | guide | complete | mcp, custom-server, python, typescript, sdk, tools, development | Написание кастомного MCP-сервера: TypeScript/Python SDK, tool + resource, подключение к Claude Code |

---

## Providers — Провайдеры API

| Файл | Тип | Статус | Теги | Описание |
|------|-----|--------|------|---------|
| `providers/providers.md` | nav | complete | providers, api, openrouter, routing | Навигация: OpenRouter (приоритет), Anthropic, OpenAI, Google, Groq |
| `providers/openrouter.md` | reference | complete | providers, openrouter, routing, fallback, api, openai-compatible, models, cost | OpenRouter: единый API для 300+ моделей, маршрутизация, fallback, мониторинг стоимости |
| `providers/anthropic.md` | reference | complete | providers, anthropic, api, claude, batch, rate-limits, direct-access, pricing | Anthropic API: прямой доступ, Batch API (50% скидка), Prompt Caching, Extended Thinking |
| `providers/openai.md` | reference | complete | providers, openai, api, gpt, o-series, structured-outputs, assistants, pricing | OpenAI API: GPT-4o, o-серия, Structured Outputs, Assistants API, Batch API |
| `providers/google.md` | reference | complete | providers, google, vertex-ai, gemini, ai-studio, gcp, grounding | Google Vertex AI / Gemini API: AI Studio vs Vertex AI, 1M контекст, Search grounding, setup |
| `providers/groq.md` | reference | complete | providers, groq, lpu, low-latency, llama, mixtral, inference | Groq: LPU inference 500-2000 tok/s, OpenAI-совместимый API, low-latency streaming |

---

## Skills — Slash Commands

| Файл | Тип | Статус | Теги | Описание |
|------|-----|--------|------|---------|
| `skills/skills.md` | nav | complete | skills, slash-commands, claude-code, automation | Навигация: анатомия skill, встроенные команды, кастомные, примеры |
| `skills/anatomy.md` | concept | complete | skills, slash-commands, claude-code, anatomy, frontmatter, arguments, subagent, invocation | Анатомия skill: SKILL.md структура, frontmatter поля, аргументы, bash injection, subagent |
| `skills/examples.md` | reference | complete | skills, slash-commands, examples, commit, review, refactor, deploy, report, security | Готовые примеры skills: commit, review-pr, fix-issue, refactor, deploy, security-review |
| `skills/builtin.md` | reference | complete | skills, slash-commands, builtin, help, compact, init, batch, loop, debug, simplify | Встроенные команды Claude Code: CLI (/help, /compact, /init) и агентные (/batch, /loop, /debug) |
