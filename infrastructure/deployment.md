---
type: guide
tags: [infrastructure, deployment, docker, production, secrets, env-vars]
status: complete
updated: 2026-03-25
related:
  - monitoring.md
  - ../agents/error-handling.md
---

# Deployment Guide

Руководство по деплою AI-агентов в продакшн: контейнеризация, управление секретами, масштабирование.

---

## Overview

Этот гайд покрывает:
- Docker контейнеризация агента
- Docker Compose для multi-agent систем
- Environment variables и secrets management
- Production best practices

---

## Single Agent Deployment

### Dockerfile

```dockerfile
# Dockerfile для Simple Tool Agent
FROM python:3.11-slim

WORKDIR /app

# Установка зависимостей
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Копирование кода агента
COPY agent.py .
COPY .env .

# Запуск агента
CMD ["python", "agent.py"]
```

### requirements.txt

```txt
anthropic==0.18.1
python-dotenv==1.0.0
requests==2.31.0
```

### Build & Run

```bash
# Build образа
docker build -t my-agent:latest .

# Run контейнера
docker run --env-file .env my-agent:latest

# Run с volume для логов
docker run \
  --env-file .env \
  -v $(pwd)/logs:/app/logs \
  my-agent:latest
```

---

## Multi-Agent Deployment

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Orchestrator Agent
  orchestrator:
    build:
      context: ./orchestrator
      dockerfile: Dockerfile
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - AGENT_ROLE=orchestrator
    depends_on:
      - backend-specialist
      - frontend-specialist
    networks:
      - agent-network
    restart: unless-stopped

  # Backend Specialist
  backend-specialist:
    build:
      context: ./specialists
      dockerfile: Dockerfile.backend
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - AGENT_ROLE=backend
    networks:
      - agent-network
    restart: unless-stopped

  # Frontend Specialist
  frontend-specialist:
    build:
      context: ./specialists
      dockerfile: Dockerfile.frontend
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - AGENT_ROLE=frontend
    networks:
      - agent-network
    restart: unless-stopped

  # RAG Agent с Qdrant
  rag-agent:
    build:
      context: ./rag-agent
      dockerfile: Dockerfile
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - OPENROUTER_API_KEY=${OPENROUTER_API_KEY}
      - QDRANT_HOST=qdrant
      - QDRANT_PORT=6333
    depends_on:
      - qdrant
    networks:
      - agent-network
    restart: unless-stopped

  # Qdrant Vector Database
  qdrant:
    image: qdrant/qdrant:latest
    ports:
      - "6333:6333"
      - "6334:6334"
    volumes:
      - qdrant-storage:/qdrant/storage
    networks:
      - agent-network
    restart: unless-stopped

networks:
  agent-network:
    driver: bridge

volumes:
  qdrant-storage:
```

### Запуск системы

```bash
# Запуск всех сервисов
docker-compose up -d

# Проверка статуса
docker-compose ps

# Логи конкретного сервиса
docker-compose logs -f orchestrator

# Остановка
docker-compose down

# Остановка с удалением volumes
docker-compose down -v
```

---

## Environment Variables

### .env файл (для разработки)

```bash
# .env
ANTHROPIC_API_KEY=sk-ant-api03-...
OPENROUTER_API_KEY=sk-or-v1-...
GITHUB_TOKEN=ghp_...

# Database
QDRANT_HOST=localhost
QDRANT_PORT=6333

# Logging
LOG_LEVEL=INFO
LOG_FORMAT=json

# Agent Config
MAX_RETRIES=3
TIMEOUT_SECONDS=30
```

### Загрузка в коде

```python
import os
from dotenv import load_dotenv

load_dotenv()

ANTHROPIC_API_KEY = os.environ.get("ANTHROPIC_API_KEY")
OPENROUTER_API_KEY = os.environ.get("OPENROUTER_API_KEY")
MAX_RETRIES = int(os.environ.get("MAX_RETRIES", "3"))
TIMEOUT_SECONDS = int(os.environ.get("TIMEOUT_SECONDS", "30"))
```

---

## Secrets Management

### Option 1: Docker Secrets (Swarm)

```yaml
# docker-compose.yml (Swarm mode)
version: '3.8'

services:
  agent:
    image: my-agent:latest
    secrets:
      - anthropic_api_key
      - openrouter_api_key
    environment:
      - ANTHROPIC_API_KEY_FILE=/run/secrets/anthropic_api_key

secrets:
  anthropic_api_key:
    external: true
  openrouter_api_key:
    external: true
```

```bash
# Создание секретов
echo "sk-ant-..." | docker secret create anthropic_api_key -
echo "sk-or-v1-..." | docker secret create openrouter_api_key -
```

```python
# Чтение секретов в коде
def get_secret(secret_name: str) -> str:
    secret_path = f"/run/secrets/{secret_name}"
    if os.path.exists(secret_path):
        with open(secret_path) as f:
            return f.read().strip()
    return os.environ.get(secret_name.upper())

ANTHROPIC_API_KEY = get_secret("anthropic_api_key")
```

### Option 2: HashiCorp Vault

```python
import hvac

# Подключение к Vault
client = hvac.Client(url='http://vault:8200', token=os.environ['VAULT_TOKEN'])

# Чтение секретов
secrets = client.secrets.kv.v2.read_secret_version(path='agent/prod')
ANTHROPIC_API_KEY = secrets['data']['data']['anthropic_api_key']
```

### Option 3: AWS Secrets Manager

```python
import boto3
import json

client = boto3.client('secretsmanager', region_name='us-east-1')

response = client.get_secret_value(SecretId='prod/agent/api-keys')
secrets = json.loads(response['SecretString'])

ANTHROPIC_API_KEY = secrets['anthropic_api_key']
```

---

## Production Best Practices

### 1. Health Checks

```python
# health.py
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/health')
def health():
    return jsonify({
        "status": "healthy",
        "timestamp": datetime.utcnow().isoformat()
    })

@app.route('/ready')
def ready():
    # Проверка зависимостей
    qdrant_ok = check_qdrant_connection()
    
    if qdrant_ok:
        return jsonify({"status": "ready"}), 200
    else:
        return jsonify({"status": "not ready"}), 503

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

```dockerfile
# Dockerfile с health check
FROM python:3.11-slim

WORKDIR /app
COPY . .
RUN pip install -r requirements.txt

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

CMD ["python", "agent.py"]
```

### 2. Graceful Shutdown

```python
import signal
import sys

def signal_handler(sig, frame):
    print('Graceful shutdown initiated...')
    
    # Завершить текущие задачи
    agent.finish_current_tasks()
    
    # Закрыть соединения
    qdrant_client.close()
    
    print('Shutdown complete')
    sys.exit(0)

signal.signal(signal.SIGINT, signal_handler)
signal.signal(signal.SIGTERM, signal_handler)
```

### 3. Resource Limits

```yaml
# docker-compose.yml
services:
  agent:
    image: my-agent:latest
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 4G
        reservations:
          cpus: '1.0'
          memory: 2G
```

### 4. Logging

```python
import logging
import json

# Structured JSON logging
class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            "timestamp": self.formatTime(record),
            "level": record.levelname,
            "message": record.getMessage(),
            "module": record.module,
            "function": record.funcName
        }
        return json.dumps(log_data)

handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())

logger = logging.getLogger(__name__)
logger.addHandler(handler)
logger.setLevel(logging.INFO)

# Использование
logger.info("Agent started", extra={"agent_id": "orchestrator-1"})
```

---

## Scaling

### Horizontal Scaling

```yaml
# docker-compose.yml
services:
  agent:
    image: my-agent:latest
    deploy:
      replicas: 3  # 3 инстанса агента
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
```

### Load Balancing

```yaml
# nginx.conf
upstream agent_backend {
    least_conn;
    server agent-1:8080;
    server agent-2:8080;
    server agent-3:8080;
}

server {
    listen 80;
    
    location / {
        proxy_pass http://agent_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## Monitoring Integration

```python
# Prometheus metrics
from prometheus_client import Counter, Histogram, start_http_server

# Метрики
agent_requests = Counter('agent_requests_total', 'Total agent requests')
agent_latency = Histogram('agent_latency_seconds', 'Agent response latency')
agent_errors = Counter('agent_errors_total', 'Total agent errors')

# Использование
@agent_latency.time()
def process_request(user_input):
    agent_requests.inc()
    try:
        result = agent.run(user_input)
        return result
    except Exception as e:
        agent_errors.inc()
        raise

# Запуск metrics endpoint
start_http_server(9090)
```

---

## CI/CD Pipeline

### GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy Agent

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Build Docker image
        run: docker build -t my-agent:${{ github.sha }} .
      
      - name: Push to registry
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker push my-agent:${{ github.sha }}
      
      - name: Deploy to production
        run: |
          ssh deploy@prod-server "docker pull my-agent:${{ github.sha }} && docker-compose up -d"
```

---

## Troubleshooting

**Container keeps restarting:**
```bash
# Проверьте логи
docker logs <container_id>

# Проверьте health check
docker inspect <container_id> | grep Health
```

**Out of memory:**
```bash
# Увеличьте memory limit
docker run -m 4g my-agent:latest
```

**Secrets not loading:**
```bash
# Проверьте что секреты существуют
docker secret ls

# Проверьте permissions
ls -la /run/secrets/
```

---

## See Also

- [Monitoring & Observability](monitoring.md) — метрики и алерты
- [Error Handling](../agents/error-handling.md) — retry логика и fallbacks
- [Docker Documentation](https://docs.docker.com/) — официальная документация
