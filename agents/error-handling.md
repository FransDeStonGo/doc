---
type: concept
tags: [agents, error-handling, retry, circuit-breaker, fallback, resilience]
status: complete
updated: 2026-03-25
related:
  - tool-use.md
  - multi-agent.md
  - ../infrastructure/deployment.md
---

# Error Handling

Паттерны обработки ошибок для AI-агентов: retry логика, circuit breaker, fallback стратегии, timeout handling.

---

## Overview

AI-агенты работают с внешними API, которые могут:
- Возвращать ошибки (rate limits, timeouts)
- Быть временно недоступны
- Давать некорректные ответы

Этот документ описывает паттерны для надёжной работы агентов.

---

## Retry Logic

### Exponential Backoff

```python
import time
import random
from typing import Callable, Any

def retry_with_exponential_backoff(
    func: Callable,
    max_retries: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 60.0,
    exponential_base: float = 2.0,
    jitter: bool = True
) -> Any:
    """
    Retry функции с exponential backoff.
    
    Args:
        func: Функция для вызова
        max_retries: Максимальное количество попыток
        base_delay: Начальная задержка (секунды)
        max_delay: Максимальная задержка (секунды)
        exponential_base: База для экспоненты (обычно 2)
        jitter: Добавить случайный jitter
    """
    for attempt in range(max_retries):
        try:
            return func()
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            
            # Рассчитываем задержку
            delay = min(base_delay * (exponential_base ** attempt), max_delay)
            
            # Добавляем jitter (случайность)
            if jitter:
                delay = delay * (0.5 + random.random())
            
            print(f"⚠️  Attempt {attempt + 1} failed: {e}")
            print(f"   Retrying in {delay:.2f}s...")
            
            time.sleep(delay)

# Использование
def call_api():
    response = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        messages=[{"role": "user", "content": "Hello"}]
    )
    return response

result = retry_with_exponential_backoff(call_api, max_retries=3)
```

### Retry на конкретные ошибки

```python
from anthropic import RateLimitError, APIConnectionError, APITimeoutError

RETRYABLE_ERRORS = (
    RateLimitError,
    APIConnectionError,
    APITimeoutError
)

def retry_on_errors(func: Callable, max_retries: int = 3) -> Any:
    """Retry только на конкретные ошибки"""
    for attempt in range(max_retries):
        try:
            return func()
        except RETRYABLE_ERRORS as e:
            if attempt == max_retries - 1:
                raise
            
            delay = 2 ** attempt  # 1s, 2s, 4s
            print(f"⚠️  Retryable error: {type(e).__name__}")
            print(f"   Retrying in {delay}s...")
            time.sleep(delay)
        except Exception as e:
            # Не retry на другие ошибки
            print(f"❌ Non-retryable error: {type(e).__name__}")
            raise
```

---

## Circuit Breaker

Паттерн для предотвращения каскадных сбоев.

```python
from enum import Enum
from datetime import datetime, timedelta

class CircuitState(Enum):
    CLOSED = "closed"      # Нормальная работа
    OPEN = "open"          # Сервис недоступен, не пытаемся вызывать
    HALF_OPEN = "half_open"  # Пробуем восстановить

class CircuitBreaker:
    """Circuit Breaker для защиты от каскадных сбоев"""
    
    def __init__(
        self,
        failure_threshold: int = 5,
        timeout: int = 60,
        expected_exception: type = Exception
    ):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.expected_exception = expected_exception
        
        self.failure_count = 0
        self.last_failure_time = None
        self.state = CircuitState.CLOSED
    
    def call(self, func: Callable) -> Any:
        """Вызвать функцию через circuit breaker"""
        if self.state == CircuitState.OPEN:
            # Проверяем можно ли перейти в HALF_OPEN
            if self._should_attempt_reset():
                self.state = CircuitState.HALF_OPEN
            else:
                raise Exception("Circuit breaker is OPEN")
        
        try:
            result = func()
            self._on_success()
            return result
        except self.expected_exception as e:
            self._on_failure()
            raise
    
    def _on_success(self):
        """Успешный вызов"""
        self.failure_count = 0
        self.state = CircuitState.CLOSED
    
    def _on_failure(self):
        """Неудачный вызов"""
        self.failure_count += 1
        self.last_failure_time = datetime.now()
        
        if self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN
            print(f"🔴 Circuit breaker OPEN (failures: {self.failure_count})")
    
    def _should_attempt_reset(self) -> bool:
        """Проверить можно ли попробовать восстановить"""
        return (
            self.last_failure_time and
            datetime.now() - self.last_failure_time > timedelta(seconds=self.timeout)
        )

# Использование
breaker = CircuitBreaker(failure_threshold=3, timeout=60)

def call_api():
    return client.messages.create(...)

try:
    result = breaker.call(call_api)
except Exception as e:
    print(f"Circuit breaker prevented call or API failed: {e}")
```

---

## Fallback Strategies

### Model Fallback

```python
from typing import List, Tuple

MODELS = [
    ("claude-sonnet-4-20250514", 3.0, 15.0),  # Primary
    ("claude-haiku-4-20250514", 0.25, 1.25),  # Fallback 1
    ("gpt-4o", 2.5, 10.0)                      # Fallback 2
]

def call_with_fallback(
    messages: List[dict],
    models: List[Tuple[str, float, float]] = MODELS
) -> Any:
    """Попробовать модели по порядку"""
    last_error = None
    
    for model, input_price, output_price in models:
        try:
            print(f"🔄 Trying model: {model}")
            
            response = client.messages.create(
                model=model,
                max_tokens=1024,
                messages=messages
            )
            
            print(f"✅ Success with {model}")
            return response
            
        except Exception as e:
            print(f"⚠️  {model} failed: {e}")
            last_error = e
            continue
    
    # Все модели провалились
    raise Exception(f"All models failed. Last error: {last_error}")

# Использование
response = call_with_fallback([
    {"role": "user", "content": "Hello"}
])
```

### Provider Fallback

```python
from anthropic import Anthropic
from openai import OpenAI

class MultiProviderClient:
    """Клиент с fallback между провайдерами"""
    
    def __init__(self):
        self.anthropic = Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])
        self.openai = OpenAI(api_key=os.environ["OPENAI_API_KEY"])
    
    def create_message(self, messages: List[dict]) -> str:
        """Попробовать Anthropic, затем OpenAI"""
        try:
            response = self.anthropic.messages.create(
                model="claude-sonnet-4-20250514",
                max_tokens=1024,
                messages=messages
            )
            return response.content[0].text
        except Exception as e:
            print(f"⚠️  Anthropic failed: {e}")
            print(f"🔄 Falling back to OpenAI...")
            
            # Конвертируем формат сообщений
            openai_messages = self._convert_to_openai_format(messages)
            
            response = self.openai.chat.completions.create(
                model="gpt-4o",
                messages=openai_messages
            )
            return response.choices[0].message.content
    
    def _convert_to_openai_format(self, messages: List[dict]) -> List[dict]:
        """Конвертировать Anthropic формат в OpenAI"""
        return messages  # Упрощённо, в реальности нужна конвертация
```

---

## Timeout Handling

```python
import signal
from contextlib import contextmanager

class TimeoutError(Exception):
    pass

@contextmanager
def timeout(seconds: int):
    """Context manager для timeout"""
    def timeout_handler(signum, frame):
        raise TimeoutError(f"Operation timed out after {seconds}s")
    
    # Установить handler
    old_handler = signal.signal(signal.SIGALRM, timeout_handler)
    signal.alarm(seconds)
    
    try:
        yield
    finally:
        # Восстановить handler
        signal.alarm(0)
        signal.signal(signal.SIGALRM, old_handler)

# Использование
try:
    with timeout(30):
        response = client.messages.create(...)
except TimeoutError as e:
    print(f"⏱️  Request timed out: {e}")
    # Fallback или retry
```

### Async Timeout

```python
import asyncio

async def call_with_timeout(func, timeout_seconds: int = 30):
    """Async timeout"""
    try:
        return await asyncio.wait_for(func(), timeout=timeout_seconds)
    except asyncio.TimeoutError:
        print(f"⏱️  Async operation timed out after {timeout_seconds}s")
        raise
```

---

## Rate Limiting

### Token Bucket

```python
import time
from threading import Lock

class TokenBucket:
    """Token bucket для rate limiting"""
    
    def __init__(self, rate: float, capacity: int):
        """
        Args:
            rate: Токенов в секунду
            capacity: Максимальное количество токенов
        """
        self.rate = rate
        self.capacity = capacity
        self.tokens = capacity
        self.last_update = time.time()
        self.lock = Lock()
    
    def consume(self, tokens: int = 1) -> bool:
        """Попытаться потребить токены"""
        with self.lock:
            now = time.time()
            elapsed = now - self.last_update
            
            # Добавляем токены за прошедшее время
            self.tokens = min(
                self.capacity,
                self.tokens + elapsed * self.rate
            )
            self.last_update = now
            
            # Проверяем достаточно ли токенов
            if self.tokens >= tokens:
                self.tokens -= tokens
                return True
            else:
                return False
    
    def wait_for_token(self, tokens: int = 1):
        """Ждать пока токены станут доступны"""
        while not self.consume(tokens):
            time.sleep(0.1)

# Использование
bucket = TokenBucket(rate=10, capacity=100)  # 10 req/sec, burst 100

def rate_limited_call():
    bucket.wait_for_token()
    return client.messages.create(...)
```

---

## Graceful Degradation

```python
class Agent:
    """Агент с graceful degradation"""
    
    def __init__(self):
        self.qdrant_available = True
        self.embeddings_available = True
    
    def run(self, user_input: str) -> str:
        """Запустить агента с fallback"""
        try:
            # Попробовать полный RAG pipeline
            if self.qdrant_available and self.embeddings_available:
                return self._run_with_rag(user_input)
        except Exception as e:
            print(f"⚠️  RAG failed: {e}")
            self.qdrant_available = False
        
        try:
            # Fallback: без RAG, только LLM
            return self._run_without_rag(user_input)
        except Exception as e:
            print(f"⚠️  LLM failed: {e}")
            # Последний fallback: статический ответ
            return "Sorry, I'm experiencing technical difficulties. Please try again later."
    
    def _run_with_rag(self, user_input: str) -> str:
        """Полный pipeline с RAG"""
        docs = self.search_documents(user_input)
        context = self.build_context(docs)
        return self.generate_with_context(user_input, context)
    
    def _run_without_rag(self, user_input: str) -> str:
        """Без RAG, только LLM"""
        return self.generate_without_context(user_input)
```

---

## Best Practices

1. **Retry только на временные ошибки** — не retry на validation errors
2. **Exponential backoff с jitter** — избегайте thundering herd
3. **Circuit breaker для внешних сервисов** — защита от каскадных сбоев
4. **Timeout на все внешние вызовы** — не ждите бесконечно
5. **Graceful degradation** — лучше частичный ответ чем ничего
6. **Логируйте все ошибки** — для анализа и алертов
7. **Мониторинг error rate** — настройте алерты на высокий error rate

---

## Complete Example

```python
# resilient_agent.py
import time
from typing import Optional

class ResilientAgent:
    """Агент с полным набором error handling"""
    
    def __init__(self):
        self.circuit_breaker = CircuitBreaker(failure_threshold=3, timeout=60)
        self.rate_limiter = TokenBucket(rate=10, capacity=100)
        self.retry_count = 0
        self.max_retries = 3
    
    def run(self, user_input: str) -> str:
        """Запустить агента с error handling"""
        self.rate_limiter.wait_for_token()
        
        try:
            return self.circuit_breaker.call(
                lambda: self._run_with_retry(user_input)
            )
        except Exception as e:
            logger.error(f"Agent failed after all retries: {e}")
            return self._fallback_response(user_input)
    
    def _run_with_retry(self, user_input: str) -> str:
        """Запустить с retry логикой"""
        return retry_with_exponential_backoff(
            lambda: self._call_llm(user_input),
            max_retries=self.max_retries
        )
    
    def _call_llm(self, user_input: str) -> str:
        """Вызов LLM с timeout"""
        try:
            with timeout(30):
                response = client.messages.create(
                    model="claude-sonnet-4-20250514",
                    max_tokens=1024,
                    messages=[{"role": "user", "content": user_input}]
                )
                return response.content[0].text
        except TimeoutError:
            raise Exception("LLM call timed out")
    
    def _fallback_response(self, user_input: str) -> str:
        """Fallback ответ при полном провале"""
        return (
            "I'm experiencing technical difficulties right now. "
            "Please try again in a few moments."
        )
```

---

## See Also

- [Tool Use](tool-use.md) — error handling для tool calls
- [Multi-Agent](multi-agent.md) — error handling в multi-agent системах
- [Deployment](../infrastructure/deployment.md) — production deployment
- [Monitoring](../infrastructure/monitoring.md) — мониторинг ошибок
