---
type: guide
tags: [agents, testing, unit-tests, integration-tests, mocking, ci-cd]
status: complete
updated: 2026-03-25
related:
  - error-handling.md
  - ../infrastructure/deployment.md
---

# Testing Strategies

Стратегии тестирования AI-агентов: unit тесты, integration тесты, mocking LLM, CI/CD.

---

## Overview

Тестирование AI-агентов отличается от обычного ПО:
- LLM ответы недетерминированы
- Внешние API (дорогие, rate limits)
- Сложные multi-agent взаимодействия

Этот гайд покрывает практические подходы к тестированию.

---

## Unit Tests

### Testing Tool Functions

```python
# test_tools.py
import pytest
from tools import get_weather, calculator, web_search

def test_calculator_add():
    result = calculator("add", 5, 3)
    assert result == 8

def test_calculator_divide():
    result = calculator("divide", 10, 2)
    assert result == 5

def test_calculator_divide_by_zero():
    result = calculator("divide", 10, 0)
    assert "Error" in str(result)

def test_get_weather():
    result = get_weather("London")
    assert "temp" in result
    assert "conditions" in result
    assert isinstance(result["temp"], (int, float))

def test_web_search():
    results = web_search("test query")
    assert isinstance(results, list)
    assert len(results) <= 3
    assert all("title" in r and "url" in r for r in results)
```

### Testing Agent Logic

```python
# test_agent.py
import pytest
from agent import Agent

def test_agent_initialization():
    agent = Agent(name="test-agent")
    assert agent.name == "test-agent"
    assert agent.tools is not None

def test_agent_tool_selection():
    agent = Agent(name="test-agent")
    
    # Агент должен выбрать calculator для математики
    tool = agent.select_tool("Calculate 5 + 3")
    assert tool == "calculator"
    
    # Агент должен выбрать weather для погоды
    tool = agent.select_tool("What's the weather in London?")
    assert tool == "get_weather"
```

---

## Mocking LLM Responses

### Using pytest-mock

```python
# test_agent_with_mock.py
import pytest
from unittest.mock import Mock, patch
from agent import Agent

@pytest.fixture
def mock_anthropic_client(mocker):
    """Mock Anthropic client"""
    mock_client = mocker.patch('agent.client')
    
    # Mock response
    mock_response = Mock()
    mock_response.content = [Mock(text="Mocked response")]
    mock_response.stop_reason = "end_turn"
    mock_response.usage = Mock(input_tokens=100, output_tokens=50)
    
    mock_client.messages.create.return_value = mock_response
    
    return mock_client

def test_agent_run_with_mock(mock_anthropic_client):
    agent = Agent(name="test-agent")
    
    result = agent.run("Test input")
    
    # Проверяем что LLM был вызван
    mock_anthropic_client.messages.create.assert_called_once()
    
    # Проверяем результат
    assert result == "Mocked response"

def test_agent_tool_use_with_mock(mock_anthropic_client):
    """Тест tool use с mock"""
    # Mock tool_use response
    mock_response = Mock()
    mock_response.stop_reason = "tool_use"
    mock_response.content = [
        Mock(
            type="tool_use",
            name="calculator",
            input={"operation": "add", "a": 5, "b": 3},
            id="tool_1"
        )
    ]
    
    mock_anthropic_client.messages.create.return_value = mock_response
    
    agent = Agent(name="test-agent")
    result = agent.run("Calculate 5 + 3")
    
    # Проверяем что tool был вызван
    assert mock_anthropic_client.messages.create.call_count >= 1
```

### Mock Fixtures

```python
# conftest.py
import pytest
from unittest.mock import Mock

@pytest.fixture
def mock_llm_response():
    """Базовый mock LLM response"""
    response = Mock()
    response.content = [Mock(text="Test response")]
    response.stop_reason = "end_turn"
    response.usage = Mock(input_tokens=100, output_tokens=50)
    return response

@pytest.fixture
def mock_tool_use_response():
    """Mock tool use response"""
    response = Mock()
    response.stop_reason = "tool_use"
    response.content = [
        Mock(
            type="tool_use",
            name="test_tool",
            input={"param": "value"},
            id="tool_1"
        )
    ]
    return response

@pytest.fixture
def mock_qdrant_client(mocker):
    """Mock Qdrant client"""
    mock_client = mocker.patch('qdrant_client.QdrantClient')
    
    # Mock search results
    mock_result = Mock()
    mock_result.payload = {"text": "Test document", "doc_id": "doc_1"}
    mock_result.score = 0.95
    
    mock_client.return_value.search.return_value = [mock_result]
    
    return mock_client
```

---

## Integration Tests

### Testing Full Agent Flow

```python
# test_integration.py
import pytest
from agent import Agent

@pytest.mark.integration
def test_simple_agent_flow():
    """Интеграционный тест с реальным LLM"""
    agent = Agent(name="integration-test")
    
    result = agent.run("What is 2 + 2?")
    
    # Проверяем что ответ содержит правильный результат
    assert "4" in result.lower()

@pytest.mark.integration
@pytest.mark.slow
def test_rag_agent_flow():
    """Тест RAG агента с реальными компонентами"""
    from rag_agent import RAGAgent
    
    agent = RAGAgent()
    
    # Индексируем тестовые документы
    agent.index_documents([
        {"id": "doc_1", "text": "Python is a programming language"}
    ])
    
    # Запрос
    result = agent.run("What is Python?")
    
    # Проверяем что ответ релевантен
    assert "programming" in result.lower()
```

### Testing Multi-Agent System

```python
# test_multi_agent.py
import pytest
from orchestrator import Orchestrator

@pytest.mark.integration
@pytest.mark.slow
def test_orchestrator_delegation():
    """Тест делегирования задач"""
    orchestrator = Orchestrator()
    
    result = orchestrator.run("Build a simple todo app")
    
    # Проверяем что оркестратор делегировал задачи
    assert orchestrator.backend_specialist.called
    assert orchestrator.frontend_specialist.called
    
    # Проверяем что результат содержит решения
    assert "backend" in result.lower()
    assert "frontend" in result.lower()
```

---

## Snapshot Testing

```python
# test_snapshots.py
import pytest
from syrupy import snapshot

def test_agent_response_snapshot(snapshot):
    """Snapshot тест для проверки регрессий"""
    agent = Agent(name="test-agent")
    
    # Используем детерминированный seed (если модель поддерживает)
    result = agent.run("Explain Python in one sentence", seed=42)
    
    # Сравниваем с сохранённым snapshot
    assert result == snapshot

# При первом запуске создаётся snapshot
# При последующих запусках проверяется соответствие
# Если ответ изменился — тест падает (можно обновить snapshot)
```

---

## Property-Based Testing

```python
# test_properties.py
from hypothesis import given, strategies as st
from agent import Agent

@given(st.text(min_size=1, max_size=100))
def test_agent_handles_any_input(user_input):
    """Агент должен обрабатывать любой input без краша"""
    agent = Agent(name="test-agent")
    
    try:
        result = agent.run(user_input)
        assert isinstance(result, str)
        assert len(result) > 0
    except Exception as e:
        # Логируем неожиданные ошибки
        pytest.fail(f"Agent crashed on input: {user_input}, error: {e}")

@given(st.integers(), st.integers())
def test_calculator_properties(a, b):
    """Property-based тест для calculator"""
    from tools import calculator
    
    # Свойство: add коммутативна
    assert calculator("add", a, b) == calculator("add", b, a)
    
    # Свойство: add и subtract обратны
    result = calculator("add", a, b)
    assert calculator("subtract", result, b) == a
```

---

## CI/CD Pipeline

### GitHub Actions

```yaml
# .github/workflows/test.yml
name: Test AI Agents

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-cov pytest-mock
      
      - name: Run unit tests
        run: |
          pytest tests/unit/ -v --cov=agents --cov-report=xml
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml
  
  integration-tests:
    runs-on: ubuntu-latest
    needs: unit-tests
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest
      
      - name: Start Qdrant
        run: |
          docker run -d -p 6333:6333 qdrant/qdrant
      
      - name: Run integration tests
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          OPENROUTER_API_KEY: ${{ secrets.OPENROUTER_API_KEY }}
        run: |
          pytest tests/integration/ -v -m integration
```

### pytest Configuration

```ini
# pytest.ini
[pytest]
markers =
    unit: Unit tests (fast, no external dependencies)
    integration: Integration tests (slow, requires API keys)
    slow: Slow tests (skip by default)

testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*

# Ignore warnings
filterwarnings =
    ignore::DeprecationWarning

# Coverage
addopts = 
    --strict-markers
    --tb=short
    --disable-warnings
```

### Test Structure

```
tests/
├── unit/
│   ├── test_tools.py
│   ├── test_agent.py
│   └── test_utils.py
├── integration/
│   ├── test_simple_agent.py
│   ├── test_rag_agent.py
│   └── test_multi_agent.py
├── fixtures/
│   ├── mock_responses.json
│   └── test_documents.json
└── conftest.py
```

---

## Test Coverage Goals

| Component | Target Coverage | Priority |
|-----------|----------------|----------|
| Tool functions | 90%+ | High |
| Agent core logic | 80%+ | High |
| Error handling | 90%+ | High |
| Integration flows | 60%+ | Medium |
| Edge cases | 70%+ | Medium |

---

## Best Practices

1. **Mock external APIs** — не тратьте деньги на тесты
2. **Separate unit and integration** — unit быстрые, integration медленные
3. **Use fixtures** — переиспользуйте mock объекты
4. **Test error paths** — не только happy path
5. **Property-based testing** — для поиска edge cases
6. **CI/CD integration** — автоматические тесты на каждый PR
7. **Coverage tracking** — стремитесь к 80%+

---

## Example: Complete Test Suite

```python
# tests/unit/test_simple_agent.py
import pytest
from unittest.mock import Mock
from agent import SimpleAgent

class TestSimpleAgent:
    """Test suite для Simple Agent"""
    
    @pytest.fixture
    def agent(self):
        return SimpleAgent(name="test-agent")
    
    @pytest.fixture
    def mock_client(self, mocker):
        mock = mocker.patch('agent.client')
        mock_response = Mock()
        mock_response.content = [Mock(text="Test response")]
        mock_response.stop_reason = "end_turn"
        mock.messages.create.return_value = mock_response
        return mock
    
    def test_initialization(self, agent):
        assert agent.name == "test-agent"
        assert len(agent.tools) == 3
    
    def test_run_with_mock(self, agent, mock_client):
        result = agent.run("Test input")
        assert result == "Test response"
        mock_client.messages.create.assert_called_once()
    
    def test_tool_call(self, agent):
        result = agent.call_tool("calculator", {"operation": "add", "a": 5, "b": 3})
        assert result == "8"
    
    def test_error_handling(self, agent, mock_client):
        mock_client.messages.create.side_effect = Exception("API Error")
        
        with pytest.raises(Exception):
            agent.run("Test input")
```

---

## See Also

- [Error Handling](error-handling.md) — тестирование retry логики
- [Deployment](../infrastructure/deployment.md) — CI/CD pipeline
- [pytest Documentation](https://docs.pytest.org/) — официальная документация
