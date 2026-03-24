---
type: concept
tags: [agents, observability, tracing, logging, monitoring, opentelemetry, langsmith, phoenix]
status: complete
updated: 2026-03-24
related:
  - agents/evaluation.md
  - agents/multi-agent.md
  - agents/planning.md
---

# Observability — Наблюдаемость агентов

## Overview

Observability для агентов — это способность понять, что агент делал, почему принял те или иные решения и где сломался. Без неё отладка агентных систем превращается в угадайку: ошибка могла произойти на любом из десятков шагов.

Ключевое отличие от observability обычных сервисов: агент — недетерминированный, его "трасса" — это не HTTP-запросы, а последовательность решений (какой tool вызвать, как интерпретировать результат, продолжать или остановиться).

## How It Works

### Что наблюдаем

```
Запрос пользователя
    │
    ├─ [LLM call 1] — решение: вызвать tool_A           ← prompt tokens, latency, model
    │       │
    │   [tool_A execution] — результат                   ← input, output, duration, errors
    │       │
    ├─ [LLM call 2] — решение: вызвать tool_B + tool_C  ← tokens, latency
    │       │
    │   [tool_B] [tool_C] — параллельные вызовы         ← latency, errors
    │       │
    └─ [LLM call 3] — финальный ответ                   ← tokens, stop_reason

Итого: трасса (trace) = последовательность spans
```

### Три столпа

| Столп | Что даёт | Пример |
|-------|---------|--------|
| **Трассировка (Traces)** | Полный путь запроса через агента | span дерево: LLM calls + tool calls |
| **Метрики (Metrics)** | Агрегированные числа | avg latency, error rate, cost per task |
| **Логи (Logs)** | Детали конкретных событий | prompt text, tool input/output, exceptions |

## Patterns

### 1. Span-based трассировка

Каждое значимое действие — отдельный span:

```python
from opentelemetry import trace

tracer = trace.get_tracer("my-agent")

def run_agent(task: str):
    with tracer.start_as_current_span("agent.run") as span:
        span.set_attribute("task", task)

        with tracer.start_as_current_span("llm.call") as llm_span:
            response = llm.invoke(task)
            llm_span.set_attribute("tokens.input", response.usage.input_tokens)
            llm_span.set_attribute("tokens.output", response.usage.output_tokens)

        if response.tool_calls:
            with tracer.start_as_current_span("tool.execute") as tool_span:
                tool_span.set_attribute("tool.name", response.tool_calls[0].name)
                result = execute_tool(response.tool_calls[0])
```

---

### 2. LLM-специфичные атрибуты

Стандартные атрибуты для LLM spans (OpenTelemetry semantic conventions):

```python
span.set_attribute("gen_ai.system", "anthropic")
span.set_attribute("gen_ai.model", "claude-sonnet-4-6")
span.set_attribute("gen_ai.request.max_tokens", 1024)
span.set_attribute("gen_ai.usage.input_tokens", 500)
span.set_attribute("gen_ai.usage.output_tokens", 200)
span.set_attribute("gen_ai.response.finish_reason", "tool_use")
```

---

### 3. Cost tracking

Считать стоимость на уровне каждого запроса:

```python
PRICE_PER_1M = {
    "claude-sonnet-4-6": {"input": 3.0, "output": 15.0},
    "claude-haiku-4-5": {"input": 0.8, "output": 4.0},
}

def track_cost(model: str, input_tokens: int, output_tokens: int) -> float:
    prices = PRICE_PER_1M[model]
    cost = (input_tokens * prices["input"] + output_tokens * prices["output"]) / 1_000_000
    span.set_attribute("gen_ai.cost.usd", cost)
    return cost
```

---

### 4. Структурированное логирование трасс

Для post-hoc анализа — сохранять полную трассу агента:

```python
import json
from datetime import datetime

class AgentTracer:
    def __init__(self):
        self.steps = []

    def log_llm_call(self, prompt, response, model, tokens):
        self.steps.append({
            "type": "llm_call",
            "timestamp": datetime.utcnow().isoformat(),
            "model": model,
            "tokens": tokens,
            "stop_reason": response.stop_reason,
        })

    def log_tool_call(self, tool_name, input, output, duration_ms):
        self.steps.append({
            "type": "tool_call",
            "timestamp": datetime.utcnow().isoformat(),
            "tool": tool_name,
            "input": input,
            "output": output,
            "duration_ms": duration_ms,
        })

    def save(self, task_id: str):
        with open(f"traces/{task_id}.json", "w") as f:
            json.dump(self.steps, f, indent=2)
```

## Tools

| Инструмент | Тип | Сильные стороны |
|-----------|-----|----------------|
| **LangSmith** | SaaS | интеграция с LangChain/LangGraph, визуализация трасс, eval |
| **Arize Phoenix** | open-source | LLM observability, spans, evals, локальный деплой |
| **Langfuse** | open-source/SaaS | трассировка + eval + prompt management, self-hosted |
| **OpenTelemetry** | стандарт | вендор-нейтральный, интеграция с любым backend |
| **Helicone** | SaaS | proxy-based, нулевой код, cost tracking |
| **Datadog LLM Obs** | SaaS | если уже используется Datadog |

### Быстрый старт с Langfuse

```bash
pip install langfuse
```

```python
from langfuse.decorators import observe, langfuse_context

@observe()
def run_agent(task: str):
    langfuse_context.update_current_observation(
        input=task,
        metadata={"version": "1.0"}
    )

    response = call_llm(task)

    langfuse_context.update_current_observation(
        output=response,
        usage={"input": 500, "output": 200}
    )
    return response
```

### Быстрый старт с Phoenix

```bash
pip install arize-phoenix openinference-instrumentation-anthropic
```

```python
import phoenix as px
from openinference.instrumentation.anthropic import AnthropicInstrumentor

px.launch_app()  # открывает UI на localhost:6006
AnthropicInstrumentor().instrument()

# Дальше работаем с Anthropic SDK как обычно — трассировка автоматическая
```

## Что логировать

| Уровень | Что важно | Что не нужно |
|---------|-----------|-------------|
| LLM call | model, tokens, latency, stop_reason, cost | полный текст промпта (можно сэмплировать) |
| Tool call | tool name, input, output (truncated), duration, error | raw HTTP logs |
| Agent run | task, total_steps, total_cost, success/fail | промежуточные рассуждения (только для debug) |
| Production | error rate, p95 latency, cost per task | verbose debug logs |

## Tradeoffs

**Полная трассировка vs производительность:**
- Запись каждого шага добавляет ~1-5ms latency
- Хранение полных промптов — дорого и риск утечки PII
- Компромисс: сэмплировать 10% трасс в prod, 100% в staging

**Инструмент vs самописное решение:**
- LangSmith/Langfuse — быстрый старт, UI из коробки, деньги
- OpenTelemetry + Jaeger/Tempo — vendor-lock нет, сложнее настройка
- Для MVP: Langfuse self-hosted или Phoenix

## See Also

- [Evaluation — оценка качества агентов](evaluation.md)
- [Multi-Agent — трассировка оркестраций](multi-agent.md)
- [Planning — ReAct и reasoning стратегии](planning.md)
