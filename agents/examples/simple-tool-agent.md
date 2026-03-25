---
type: guide
tags: [agents, examples, tool-use, anthropic, python, beginner]
status: complete
updated: 2026-03-25
related:
  - ../tool-use.md
  - ../../ai/anthropic/anthropic.md
---

# Simple Tool Agent

Базовый агент с tool use — демонстрация механизма вызова функций через Anthropic Claude API.

---

## Overview

Этот пример показывает:
- Как определить инструменты (tools) для агента
- Как обработать tool use запрос от модели
- Как вернуть результат и получить финальный ответ

**Сложность:** ⭐ Начальный  
**Строк кода:** ~100  
**Инструменты:** weather, calculator, web_search

---

## Code

```python
#!/usr/bin/env python3
"""
Simple Tool Agent — базовый агент с 3 инструментами.
Демонстрирует механизм tool use в Anthropic Claude API.
"""

import os
import json
from anthropic import Anthropic
from dotenv import load_dotenv

load_dotenv()

client = Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))

# Определение инструментов
tools = [
    {
        "name": "get_weather",
        "description": "Get current weather for a location. Returns temperature in Celsius and conditions.",
        "input_schema": {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "City name, e.g. 'London' or 'New York'"
                }
            },
            "required": ["location"]
        }
    },
    {
        "name": "calculator",
        "description": "Perform basic math operations: add, subtract, multiply, divide.",
        "input_schema": {
            "type": "object",
            "properties": {
                "operation": {
                    "type": "string",
                    "enum": ["add", "subtract", "multiply", "divide"],
                    "description": "Math operation to perform"
                },
                "a": {
                    "type": "number",
                    "description": "First number"
                },
                "b": {
                    "type": "number",
                    "description": "Second number"
                }
            },
            "required": ["operation", "a", "b"]
        }
    },
    {
        "name": "web_search",
        "description": "Search the web for current information. Returns top 3 results.",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "Search query"
                }
            },
            "required": ["query"]
        }
    }
]

# Реализация инструментов
def get_weather(location: str) -> dict:
    """Mock weather API — в реальности вызов к wttr.in или OpenWeatherMap"""
    mock_data = {
        "London": {"temp": 12, "conditions": "Cloudy"},
        "New York": {"temp": 18, "conditions": "Sunny"},
        "Tokyo": {"temp": 22, "conditions": "Clear"}
    }
    return mock_data.get(location, {"temp": 15, "conditions": "Unknown"})

def calculator(operation: str, a: float, b: float) -> float:
    """Basic calculator"""
    ops = {
        "add": lambda x, y: x + y,
        "subtract": lambda x, y: x - y,
        "multiply": lambda x, y: x * y,
        "divide": lambda x, y: x / y if y != 0 else "Error: division by zero"
    }
    return ops[operation](a, b)

def web_search(query: str) -> list:
    """Mock web search — в реальности вызов к Brave/Google API"""
    return [
        {"title": f"Result 1 for {query}", "url": "https://example.com/1"},
        {"title": f"Result 2 for {query}", "url": "https://example.com/2"},
        {"title": f"Result 3 for {query}", "url": "https://example.com/3"}
    ]

# Маппинг имён инструментов на функции
tool_functions = {
    "get_weather": get_weather,
    "calculator": calculator,
    "web_search": web_search
}

def process_tool_call(tool_name: str, tool_input: dict) -> str:
    """Вызов инструмента и возврат результата"""
    if tool_name not in tool_functions:
        return json.dumps({"error": f"Unknown tool: {tool_name}"})
    
    try:
        result = tool_functions[tool_name](**tool_input)
        return json.dumps(result)
    except Exception as e:
        return json.dumps({"error": str(e)})

def run_agent(user_message: str):
    """Основной цикл агента"""
    print(f"\n🧑 User: {user_message}\n")
    
    messages = [{"role": "user", "content": user_message}]
    
    # Первый запрос к модели
    response = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        tools=tools,
        messages=messages
    )
    
    print(f"🤖 Agent thinking... (stop_reason: {response.stop_reason})\n")
    
    # Если модель хочет вызвать инструмент
    while response.stop_reason == "tool_use":
        # Добавляем ответ модели в историю
        messages.append({"role": "assistant", "content": response.content})
        
        # Обрабатываем все tool use блоки
        tool_results = []
        for block in response.content:
            if block.type == "tool_use":
                tool_name = block.name
                tool_input = block.input
                
                print(f"🔧 Calling tool: {tool_name}")
                print(f"   Input: {json.dumps(tool_input, indent=2)}")
                
                # Вызываем инструмент
                result = process_tool_call(tool_name, tool_input)
                
                print(f"   Result: {result}\n")
                
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": result
                })
        
        # Отправляем результаты обратно модели
        messages.append({"role": "user", "content": tool_results})
        
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=1024,
            tools=tools,
            messages=messages
        )
        
        print(f"🤖 Agent thinking... (stop_reason: {response.stop_reason})\n")
    
    # Финальный ответ
    final_response = next(
        (block.text for block in response.content if hasattr(block, "text")),
        None
    )
    
    print(f"💬 Agent: {final_response}\n")
    return final_response

# Примеры использования
if __name__ == "__main__":
    # Пример 1: Погода
    run_agent("What's the weather in London?")
    
    # Пример 2: Калькулятор
    run_agent("Calculate 15 * 7 + 23")
    
    # Пример 3: Веб-поиск
    run_agent("Search for latest AI news")
```

---

## How It Works

### 1. Tool Definition
```python
tools = [
    {
        "name": "get_weather",
        "description": "Get current weather...",
        "input_schema": {
            "type": "object",
            "properties": {...},
            "required": [...]
        }
    }
]
```

Каждый инструмент описывается JSON Schema — модель понимает когда и как его вызывать.

### 2. Tool Use Loop
```python
while response.stop_reason == "tool_use":
    # 1. Модель вернула tool_use блоки
    # 2. Вызываем функции
    # 3. Отправляем результаты обратно
    # 4. Модель продолжает или возвращает финальный ответ
```

### 3. Tool Result Format
```python
{
    "type": "tool_result",
    "tool_use_id": block.id,  # ID из tool_use блока
    "content": json.dumps(result)  # Результат в JSON
}
```

---

## Running

```bash
# 1. Установите зависимости
pip install anthropic python-dotenv

# 2. Создайте .env
echo "ANTHROPIC_API_KEY=sk-ant-..." > .env

# 3. Запустите
python simple-tool-agent.py
```

**Ожидаемый вывод:**
```
🧑 User: What's the weather in London?

🤖 Agent thinking... (stop_reason: tool_use)

🔧 Calling tool: get_weather
   Input: {
     "location": "London"
   }
   Result: {"temp": 12, "conditions": "Cloudy"}

🤖 Agent thinking... (stop_reason: end_turn)

💬 Agent: The current weather in London is 12°C and cloudy.
```

---

## Extending

### Добавление нового инструмента

```python
# 1. Добавьте в tools список
{
    "name": "get_stock_price",
    "description": "Get current stock price",
    "input_schema": {
        "type": "object",
        "properties": {
            "symbol": {"type": "string", "description": "Stock ticker, e.g. AAPL"}
        },
        "required": ["symbol"]
    }
}

# 2. Реализуйте функцию
def get_stock_price(symbol: str) -> dict:
    # Вызов к Alpha Vantage / Yahoo Finance API
    return {"symbol": symbol, "price": 150.25, "change": "+2.3%"}

# 3. Добавьте в маппинг
tool_functions["get_stock_price"] = get_stock_price
```

---

## Best Practices

1. **Tool descriptions** — чёткие, конкретные, с примерами
2. **Error handling** — всегда возвращайте JSON, даже при ошибке
3. **Input validation** — проверяйте параметры перед вызовом API
4. **Idempotency** — инструменты должны быть безопасны для повторного вызова
5. **Timeouts** — добавьте таймауты для внешних API

---

## Troubleshooting

**Модель не вызывает инструмент:**
- Проверьте description — достаточно ли понятно когда использовать?
- Добавьте примеры в description: "e.g. 'London', 'New York'"

**Ошибка "tool_use_id not found":**
- Убедитесь что `tool_use_id` в `tool_result` совпадает с `block.id` из `tool_use`

**Бесконечный цикл tool use:**
- Добавьте счётчик итераций: `max_iterations = 5`
- Проверьте что инструмент возвращает валидный JSON

---

## See Also

- [Tool Use](../tool-use.md) — теория механизма tool use
- [Anthropic API](../../ai/anthropic/anthropic.md) — документация по Claude API
- [MCP Agent](mcp-agent.md) — агент с MCP-серверами (стандартизированный tool use)
