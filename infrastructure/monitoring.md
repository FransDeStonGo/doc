---
type: guide
tags: [infrastructure, monitoring, observability, prometheus, grafana, metrics, logging]
status: complete
updated: 2026-03-25
related:
  - deployment.md
  - ../agents/observability.md
---

# Monitoring & Observability

Полное руководство по мониторингу AI-агентов: метрики, логи, трассировка, алерты.

---

## Overview

Этот гайд покрывает:
- Prometheus metrics для агентов
- Grafana dashboards
- Structured logging (JSON)
- Alerting rules
- Cost tracking

---

## Prometheus Metrics

### Agent Metrics

```python
# metrics.py
from prometheus_client import Counter, Histogram, Gauge, start_http_server
import time

# Request metrics
agent_requests_total = Counter(
    'agent_requests_total',
    'Total number of agent requests',
    ['agent_name', 'status']
)

agent_request_duration_seconds = Histogram(
    'agent_request_duration_seconds',
    'Agent request duration in seconds',
    ['agent_name'],
    buckets=[0.1, 0.5, 1.0, 2.0, 5.0, 10.0, 30.0, 60.0]
)

# Token metrics
agent_tokens_used = Counter(
    'agent_tokens_used_total',
    'Total tokens used',
    ['agent_name', 'token_type']  # token_type: input, output
)

# Cost metrics
agent_cost_usd = Counter(
    'agent_cost_usd_total',
    'Total cost in USD',
    ['agent_name', 'model']
)

# Active requests
agent_active_requests = Gauge(
    'agent_active_requests',
    'Number of active requests',
    ['agent_name']
)

# Error metrics
agent_errors_total = Counter(
    'agent_errors_total',
    'Total number of errors',
    ['agent_name', 'error_type']
)

# Tool use metrics
agent_tool_calls_total = Counter(
    'agent_tool_calls_total',
    'Total tool calls',
    ['agent_name', 'tool_name', 'status']
)

agent_tool_duration_seconds = Histogram(
    'agent_tool_duration_seconds',
    'Tool call duration',
    ['agent_name', 'tool_name']
)

# Запуск metrics server
def start_metrics_server(port=9090):
    start_http_server(port)
    print(f"✅ Metrics server started on port {port}")
```

### Instrumented Agent

```python
# agent_with_metrics.py
from metrics import *
import time

class MonitoredAgent:
    def __init__(self, name: str):
        self.name = name
    
    def run(self, user_input: str):
        # Increment active requests
        agent_active_requests.labels(agent_name=self.name).inc()
        
        start_time = time.time()
        
        try:
            # Process request
            result = self._process(user_input)
            
            # Record success
            agent_requests_total.labels(
                agent_name=self.name,
                status='success'
            ).inc()
            
            return result
            
        except Exception as e:
            # Record error
            agent_requests_total.labels(
                agent_name=self.name,
                status='error'
            ).inc()
            
            agent_errors_total.labels(
                agent_name=self.name,
                error_type=type(e).__name__
            ).inc()
            
            raise
        
        finally:
            # Record duration
            duration = time.time() - start_time
            agent_request_duration_seconds.labels(
                agent_name=self.name
            ).observe(duration)
            
            # Decrement active requests
            agent_active_requests.labels(agent_name=self.name).dec()
    
    def _process(self, user_input: str):
        # Call LLM
        response = self._call_llm(user_input)
        
        # Record tokens
        agent_tokens_used.labels(
            agent_name=self.name,
            token_type='input'
        ).inc(response.usage.input_tokens)
        
        agent_tokens_used.labels(
            agent_name=self.name,
            token_type='output'
        ).inc(response.usage.output_tokens)
        
        # Record cost
        cost = self._calculate_cost(response.usage)
        agent_cost_usd.labels(
            agent_name=self.name,
            model=response.model
        ).inc(cost)
        
        return response.content[0].text
    
    def call_tool(self, tool_name: str, tool_input: dict):
        start_time = time.time()
        
        try:
            result = self._execute_tool(tool_name, tool_input)
            
            agent_tool_calls_total.labels(
                agent_name=self.name,
                tool_name=tool_name,
                status='success'
            ).inc()
            
            return result
            
        except Exception as e:
            agent_tool_calls_total.labels(
                agent_name=self.name,
                tool_name=tool_name,
                status='error'
            ).inc()
            raise
        
        finally:
            duration = time.time() - start_time
            agent_tool_duration_seconds.labels(
                agent_name=self.name,
                tool_name=tool_name
            ).observe(duration)

# Запуск
if __name__ == "__main__":
    start_metrics_server(port=9090)
    
    agent = MonitoredAgent(name="orchestrator")
    agent.run("Build a todo app")
```

---

## Prometheus Configuration

### prometheus.yml

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  # Agent metrics
  - job_name: 'agents'
    static_configs:
      - targets:
          - 'orchestrator:9090'
          - 'backend-specialist:9090'
          - 'frontend-specialist:9090'
          - 'rag-agent:9090'
    
  # Qdrant metrics
  - job_name: 'qdrant'
    static_configs:
      - targets: ['qdrant:6333']

# Alerting rules
rule_files:
  - 'alerts.yml'

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']
```

### alerts.yml

```yaml
groups:
  - name: agent_alerts
    interval: 30s
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          rate(agent_errors_total[5m]) > 0.1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High error rate for {{ $labels.agent_name }}"
          description: "Error rate is {{ $value }} errors/sec"
      
      # High latency
      - alert: HighLatency
        expr: |
          histogram_quantile(0.95, 
            rate(agent_request_duration_seconds_bucket[5m])
          ) > 30
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency for {{ $labels.agent_name }}"
          description: "P95 latency is {{ $value }}s"
      
      # High cost
      - alert: HighCost
        expr: |
          rate(agent_cost_usd_total[1h]) > 10
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "High cost for {{ $labels.agent_name }}"
          description: "Cost rate is ${{ $value }}/hour"
      
      # Agent down
      - alert: AgentDown
        expr: up{job="agents"} == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Agent {{ $labels.instance }} is down"
```

---

## Grafana Dashboards

### Dashboard JSON

```json
{
  "dashboard": {
    "title": "AI Agent Monitoring",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(agent_requests_total[5m])",
            "legendFormat": "{{agent_name}} - {{status}}"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Latency (P50, P95, P99)",
        "targets": [
          {
            "expr": "histogram_quantile(0.50, rate(agent_request_duration_seconds_bucket[5m]))",
            "legendFormat": "P50 - {{agent_name}}"
          },
          {
            "expr": "histogram_quantile(0.95, rate(agent_request_duration_seconds_bucket[5m]))",
            "legendFormat": "P95 - {{agent_name}}"
          },
          {
            "expr": "histogram_quantile(0.99, rate(agent_request_duration_seconds_bucket[5m]))",
            "legendFormat": "P99 - {{agent_name}}"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Token Usage",
        "targets": [
          {
            "expr": "rate(agent_tokens_used_total[5m])",
            "legendFormat": "{{agent_name}} - {{token_type}}"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Cost per Hour",
        "targets": [
          {
            "expr": "rate(agent_cost_usd_total[1h]) * 3600",
            "legendFormat": "{{agent_name}} - {{model}}"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(agent_errors_total[5m])",
            "legendFormat": "{{agent_name}} - {{error_type}}"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Tool Calls",
        "targets": [
          {
            "expr": "rate(agent_tool_calls_total[5m])",
            "legendFormat": "{{tool_name}} - {{status}}"
          }
        ],
        "type": "graph"
      }
    ]
  }
}
```

---

## Structured Logging

### JSON Logger

```python
# logger.py
import logging
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            "timestamp": datetime.utcnow().isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
            "module": record.module,
            "function": record.funcName,
            "line": record.lineno
        }
        
        # Add extra fields
        if hasattr(record, 'agent_name'):
            log_data['agent_name'] = record.agent_name
        if hasattr(record, 'request_id'):
            log_data['request_id'] = record.request_id
        if hasattr(record, 'user_id'):
            log_data['user_id'] = record.user_id
        
        # Add exception info
        if record.exc_info:
            log_data['exception'] = self.formatException(record.exc_info)
        
        return json.dumps(log_data)

def setup_logger(name: str, level=logging.INFO):
    logger = logging.getLogger(name)
    logger.setLevel(level)
    
    handler = logging.StreamHandler()
    handler.setFormatter(JSONFormatter())
    
    logger.addHandler(handler)
    
    return logger

# Использование
logger = setup_logger(__name__)

logger.info(
    "Agent request started",
    extra={
        "agent_name": "orchestrator",
        "request_id": "req-123",
        "user_id": "user-456"
    }
)
```

### Log Aggregation (ELK Stack)

```yaml
# docker-compose.yml
services:
  # Elasticsearch
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - es-data:/usr/share/elasticsearch/data

  # Logstash
  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch

  # Kibana
  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

volumes:
  es-data:
```

---

## Docker Compose with Monitoring

```yaml
# docker-compose.monitoring.yml
version: '3.8'

services:
  # Prometheus
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9091:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alerts.yml:/etc/prometheus/alerts.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    networks:
      - monitoring

  # Grafana
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana-dashboards:/etc/grafana/provisioning/dashboards
    depends_on:
      - prometheus
    networks:
      - monitoring

  # Alertmanager
  alertmanager:
    image: prom/alertmanager:latest
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml
    networks:
      - monitoring

networks:
  monitoring:
    driver: bridge

volumes:
  prometheus-data:
  grafana-data:
```

```bash
# Запуск мониторинга
docker-compose -f docker-compose.monitoring.yml up -d

# Доступ к Grafana
open http://localhost:3000
# Login: admin / admin
```

---

## Cost Tracking

```python
# cost_tracker.py
from dataclasses import dataclass
from typing import Dict

@dataclass
class ModelPricing:
    input_per_1m: float  # USD per 1M input tokens
    output_per_1m: float  # USD per 1M output tokens

PRICING = {
    "claude-sonnet-4-20250514": ModelPricing(3.0, 15.0),
    "claude-haiku-4-20250514": ModelPricing(0.25, 1.25),
    "gpt-4o": ModelPricing(2.5, 10.0),
    "text-embedding-3-small": ModelPricing(0.02, 0.0)
}

def calculate_cost(model: str, input_tokens: int, output_tokens: int) -> float:
    """Рассчитать стоимость запроса"""
    pricing = PRICING.get(model)
    if not pricing:
        return 0.0
    
    input_cost = (input_tokens / 1_000_000) * pricing.input_per_1m
    output_cost = (output_tokens / 1_000_000) * pricing.output_per_1m
    
    return input_cost + output_cost

# Использование
cost = calculate_cost(
    model="claude-sonnet-4-20250514",
    input_tokens=1500,
    output_tokens=500
)
print(f"Cost: ${cost:.4f}")  # Cost: $0.0120
```

---

## Alerting

### Alertmanager Configuration

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname', 'agent_name']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'telegram'

receivers:
  - name: 'telegram'
    telegram_configs:
      - bot_token: 'YOUR_BOT_TOKEN'
        chat_id: YOUR_CHAT_ID
        message: |
          🚨 Alert: {{ .GroupLabels.alertname }}
          Agent: {{ .GroupLabels.agent_name }}
          {{ range .Alerts }}
          Summary: {{ .Annotations.summary }}
          Description: {{ .Annotations.description }}
          {{ end }}
```

---

## Best Practices

1. **Metric naming** — используйте `_total` для counters, `_seconds` для duration
2. **Label cardinality** — избегайте high-cardinality labels (user_id, request_id)
3. **Sampling** — для high-volume метрик используйте sampling
4. **Retention** — настройте retention policy (default 15 days)
5. **Dashboards** — создавайте отдельные dashboards для разных ролей (dev, ops, business)

---

## See Also

- [Deployment](deployment.md) — деплой агентов
- [Observability](../agents/observability.md) — теория observability
- [Prometheus Documentation](https://prometheus.io/docs/) — официальная документация
